## Instalación

Se instaló Proxmox VE sobre el SSD de 250 GB.

Configuración inicial:

- Dirección IP: 192.168.1.105
- Acceso Web:
```
https://192.168.1.105:8006
```

---

## Repositorio No Subscription

Se eliminó el repositorio Enterprise.

Se añadió el repositorio:

```
deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription
```

Posteriormente:

```bash
apt update
apt full-upgrade
```

También se eliminó el aviso de suscripción desde la interfaz web.
