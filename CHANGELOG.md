# Changelog

## 2026-07-02

### Añadido
- Instalación de Proxmox VE.
- Activación de Intel VT-x.
- Primera VM Ubuntu Server.
- Configuración de OpenSSH.
- Autenticación mediante clave pública.
- Usuario administrador `bonzoycv`.
- Acceso desde PikaOS.
- Acceso desde Fedora.

### Pendiente
- QEMU Guest Agent.
- Snapshots.
- Backups.
- Jellyfin.
- Pi-hole.


## 2026-07-03

### Añadido

- Documentación del diagnóstico SMART de los discos.
- Nuevo documento `diagnostico_discos_2026-07-03.md`.
- Creación del archivo `CONTRIBUTING.md` con las convenciones del proyecto.
- Definición de la estructura y organización del repositorio.
- Convención oficial para nombres de archivos (`nombre_del_documento_YYYY-MM-DD.md`).
- Convención para documentación permanente y bitácora técnica.
- Convención para nombres de scripts.
- Convención para nombres de máquinas virtuales.
- Convención para mensajes de commit.
- Definición del flujo de trabajo del proyecto.
- Documentación de buenas prácticas para el manejo de información sensible.

### Modificado

- Actualización del `README.md`.
- Mejora de la organización general del repositorio.

### Verificado

- Estado SMART del Samsung SSD 850 EVO 250 GB.
- Estado SMART del Seagate BarraCuda 1 TB.
- Ambos discos superan la verificación SMART (`PASSED`).

### Pendiente

- Ejecutar Self-Test extendido del SSD.
- Ejecutar Self-Test extendido del HDD.
- Instalar el segundo SSD de 250 GB.
- Instalar y configurar `qemu-guest-agent`.
- Crear el snapshot **Fresh Install**.
- Configurar la estrategia de backups.
