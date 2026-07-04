# Migración de Pi-hole al nuevo homelab (Proxmox + VM Ubuntu Server)

Fecha: 2026-07-04

## Contexto

Migración del servidor anterior (bonzoserver, HP EliteBook, Ubuntu 24.04) hacia
el nuevo hardware (i5-6500, 32GB RAM) con Proxmox VE 9.2.3 como hypervisor.
Dentro de Proxmox corre una VM Ubuntu Server 26.04 (`ubuntu-server`,
192.168.1.108) donde viven todos los servicios Docker: Caddy, Jellyfin,
Cloudflared, Portainer y ahora Pi-hole.

Los discos del servidor anterior (HDD Toshiba de 750GB) se instalaron
físicamente en el nuevo hardware y se mantienen intactos, sin borrar nada,
hasta confirmar que todo funciona bien en la nueva plataforma.

## Arquitectura final

```
Proxmox (192.168.1.105)
├── Disco Toshiba 750GB (ubuntu-vg/ubuntu-lv) -> datos del servidor viejo, intacto
├── Discos nuevos (shared_data, media_ssd1, vm_storage)
└── VM ubuntu-server (192.168.1.108)
    └── Docker, red "homelab" (172.20.0.0/16)
        ├── caddy       (puerto 80/443, reverse proxy)
        ├── jellyfin    (puerto 8096)
        ├── cloudflared (tunnel, sin puertos expuestos)
        └── pihole      (network_mode: host, DNS puerto 53, WebUI puerto 8080)
```

Estructura de carpetas en la VM (`/home/bonzoycv/home-server/`):

```
home-server/
├── compose/
│   ├── caddy/
│   ├── cloudflared/
│   ├── jellyfin/
│   ├── portainer/
│   └── pihole/
│       ├── docker-compose.yml
│       └── .env
├── data/
│   ├── caddy/
│   ├── jellyfin/
│   ├── portainer/
│   └── pihole/
│       ├── etc-pihole/
│       └── etc-dnsmasq.d/
├── backups/
└── scripts/
```

## Paso 1: Localizar los datos de Pi-hole en el disco viejo

El disco del servidor anterior no estaba montado en Proxmox por defecto (el
punto `/mnt/legacy_server` existía como carpeta vacía, sin nada montado
encima). Se identificó el volumen LVM correspondiente:

```bash
sudo vgs
sudo lvs
```

Se encontró `ubuntu-vg/ubuntu-lv` (696.63G), correspondiente al disco viejo.
Se montó temporalmente en un punto de prueba para no interferir con el
export NFS existente:

```bash
sudo mkdir -p /mnt/legacy_server_test
sudo mount /dev/mapper/ubuntu--vg-ubuntu--lv /mnt/legacy_server_test
```

Dentro se localizó Pi-hole como bind mount de Docker (no como volumen
nombrado) en:

```
/mnt/legacy_server_test/opt/pihole/etc-pihole/
/mnt/legacy_server_test/opt/pihole/etc-dnsmasq.d/
/mnt/legacy_server_test/opt/pihole/docker-compose.yml
```

El `docker-compose.yml` original:

```yaml
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:2026.02.0
    network_mode: host
    environment:
      TZ: 'Europe/Belgrade'
      WEBPASSWORD: 'tucontraseña'
      DNSMASQ_LISTENING: 'all'
      WEB_PORT: '8080'
    volumes:
      - '/opt/pihole/etc-pihole:/etc/pihole'
      - '/opt/pihole/etc-dnsmasq.d:/etc/dnsmasq.d'
    restart: unless-stopped
```

Esto confirmó que se trataba de Pi-hole v6 (config unificada en
`pihole.toml`, ya no `setupVars.conf` separado).

## Paso 2: Configurar acceso SSH entre Proxmox y la VM

El SSH de Proxmox solo acepta autenticación por llave (no por password), así
que hubo que autorizar la llave pública de la VM manualmente desde la consola
web de Proxmox (Shell), ya que `ssh-copy-id` no funciona sin autenticación
previa:

```bash
# Desde la consola web de Proxmox
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
# pegar el contenido de ~/.ssh/id_ed25519.pub de la VM
chmod 600 ~/.ssh/authorized_keys
```

Verificación desde la VM:

```bash
ssh bonzoycv@192.168.1.105 "echo ok"
```

Nota: los comandos de copia deben ejecutarse con el usuario normal, sin
`sudo` en el lado local, ya que `sudo rsync` intenta usar las llaves SSH de
`root`, no las del usuario `bonzoycv`.

## Paso 3: Copiar los datos hacia la VM

```bash
rsync -avz --progress bonzoycv@192.168.1.105:/mnt/legacy_server_test/opt/pihole/etc-pihole/ /home/bonzoycv/home-server/data/pihole/etc-pihole/
rsync -avz --progress bonzoycv@192.168.1.105:/mnt/legacy_server_test/opt/pihole/etc-dnsmasq.d/ /home/bonzoycv/home-server/data/pihole/etc-dnsmasq.d/
```

Un archivo (`logrotate`) no se pudo copiar por permisos, pero no es crítico
(se regenera automáticamente).

Tras la copia, se limpiaron archivos que no debían migrarse:

```bash
cd /home/bonzoycv/home-server/data/pihole/etc-pihole/
rm -f pihole-FTL.db-wal pihole-FTL.db-shm
rm -f gravity_old.db
rm -rf migration_backup
```

Razón: los archivos `.db-wal`/`.db-shm` son de una sesión SQLite que puede
haber quedado abierta; es más seguro dejar que Pi-hole nuevo regenere su
propio WAL al arrancar. `gravity_old.db` y `migration_backup` eran restos de
una migración anterior de Pi-hole v5 a v6.

## Paso 4: Resolver el conflicto de puerto 53 (systemd-resolved)

Ubuntu Server usa `systemd-resolved` con un stub listener en 127.0.0.53,
lo cual no choca directamente con Pi-hole en modo host, pero es más seguro
desactivarlo para evitar comportamientos inesperados:

```bash
sudo mkdir -p /etc/systemd/resolved.conf.d/
sudo tee /etc/systemd/resolved.conf.d/no-stub.conf << 'EOF'
[Resolve]
DNSStubListener=no
EOF

sudo rm /etc/resolv.conf
sudo tee /etc/resolv.conf << 'EOF'
nameserver 1.1.1.1
nameserver 9.9.9.9
EOF

sudo systemctl restart systemd-resolved
```

Verificación de que el puerto quedó libre:

```bash
sudo ss -tulpn | grep ':53'
# sin salida = puerto libre
```

## Paso 5: Crear el docker-compose.yml nuevo

Carpeta: `home-server/compose/pihole/`

`.env`:
```
TZ=Europe/Belgrade
WEBPASSWORD=tucontraseña_real
```

`docker-compose.yml`:
```yaml
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:2026.02.0
    network_mode: host
    environment:
      TZ: '${TZ}'
      WEBPASSWORD: '${WEBPASSWORD}'
      DNSMASQ_LISTENING: 'all'
      WEB_PORT: '8080'
    volumes:
      - '../../data/pihole/etc-pihole:/etc/pihole'
      - '../../data/pihole/etc-dnsmasq.d:/etc/dnsmasq.d'
    restart: unless-stopped
```

Se mantuvo `network_mode: host` igual que en la configuración original, y el
puerto de la WebUI en 8080 (en vez del 80/443 por defecto) porque Caddy ya
ocupa esos puertos en la misma VM.

Levantar el contenedor:

```bash
docker compose up -d
docker compose logs -f
```

## Paso 6: Corregir el puerto de la WebUI

Al primer arranque, el contenedor falló al iniciar el servidor web. Causa:
el `pihole.toml` migrado ya traía su propia configuración de puertos
(80/443), y en Pi-hole v6 el archivo de configuración existente tiene
prioridad sobre la variable de entorno `WEB_PORT`. Como el puerto 80 ya
estaba tomado por Caddy, el webserver no pudo arrancar.

Solución, editando la configuración directamente vía la CLI de FTL:

```bash
docker exec -it pihole pihole-FTL --config webserver.port "8080o,[::]:8080o"
docker restart pihole
```

Tras el reinicio, el log confirmó:

```
INFO: Web server ports:
INFO:   - 0.0.0.0:8080 (HTTP, IPv4, optional, OK)
INFO:   - [::]:8080 (HTTP, IPv6, optional, OK)
```

## Paso 7: Verificación

```bash
docker ps | grep pihole
curl -I http://localhost:8080/admin/
```

Acceso a la interfaz web: `http://192.168.1.108:8080/admin/`

El dashboard confirmó la migración exitosa: 866,510 dominios en listas,
grupos y dominios personalizados presentes, igual que en el servidor
anterior.

## Paso 8: Apuntar la VM a usar Pi-hole como DNS

Una vez confirmado que Pi-hole funcionaba, se actualizó el DNS de la propia
VM (antes apuntaba a Cloudflare/Quad9 de forma temporal):

```bash
sudo tee /etc/resolv.conf << 'EOF'
nameserver 127.0.0.1
EOF
```

## Paso 9: Configurar el router para que toda la red use Pi-hole

En el router TP-Link (192.168.1.1), sección Advanced > Network > LAN
Settings > DHCP Server:

- Primary DNS: `192.168.1.108` (Pi-hole)
- Secondary DNS: `1.1.1.1` (Cloudflare, como respaldo)

Esto permite que cualquier dispositivo que se conecte a la red reciba
automáticamente el DNS de Pi-hole vía DHCP, sin configuración manual por
dispositivo. Si Pi-hole llegara a caerse, los dispositivos usan el DNS
secundario tras un breve timeout, por lo que el internet sigue funcionando
(sin el bloqueo de anuncios) en vez de quedarse sin resolución DNS.

Tras guardar el cambio, se reinició el router para forzar a todos los
clientes a renovar su lease DHCP de inmediato.

## Decisión: Proxmox no usa Pi-hole como DNS

Se decidió mantener el host Proxmox usando el DNS del router directamente,
sin apuntarlo a Pi-hole. Razón: Pi-hole corre dentro de una VM alojada en
el mismo Proxmox; si Pi-hole se cae o la VM no arranca, el hypervisor podría
quedarse sin resolución DNS para sus propias tareas de administración. Es
más seguro mantener el host de virtualización independiente del servicio
que él mismo hospeda.

## Verificación final

Pruebas de resolución DNS desde distintos hosts de la red:

```bash
# Desde ubuntu-server (usando Pi-hole local)
nslookup doubleclick.net
# Address: 0.0.0.0  -> bloqueado correctamente

# Desde pikaos (usando DHCP del router -> Pi-hole)
nslookup doubleclick.net
# Server: 192.168.1.108
# Address: 0.0.0.0  -> bloqueado correctamente

# Desde proxmox (DNS directo al router, a propósito)
nslookup doubleclick.net
# Server: 192.168.1.1
# Address: 142.251.38.206  -> resuelve normal, sin pasar por Pi-hole
```

## Pendientes

- Configurar backup periódico de `gravity.db` y `pihole.toml` en
  `home-server/backups/`.
- Evaluar si activar DHCP en Pi-hole en el futuro (actualmente el DHCP lo
  sigue manejando el router; el archivo `dhcp.leases` migrado estaba vacío,
  por lo que no había reservas previas que preservar).
- Revisar la carpeta `ruta/` encontrada en la raíz del disco viejo durante
  la exploración (no estándar de Ubuntu, pendiente de investigar contenido).
