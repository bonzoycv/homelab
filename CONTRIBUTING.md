# Contributing

> Este documento define las convenciones y buenas prácticas utilizadas en el repositorio **Homelab**. Su objetivo es mantener una estructura consistente, facilitar el mantenimiento y asegurar que toda la documentación y configuraciones sigan el mismo estándar.

---

# Filosofía del proyecto

El homelab no es únicamente un conjunto de máquinas virtuales y servicios.

Este repositorio documenta la evolución completa de la infraestructura, permitiendo reconstruir el laboratorio desde cero y comprender las decisiones técnicas tomadas en cada etapa.

Toda modificación importante debe quedar documentada y versionada.

---

# Estructura del repositorio

```text
homelab/
├── CONTRIBUTING.md
├── README.md
├── CHANGELOG.md
├── configs/
├── docs/
├── inventory/
├── proxmox/
├── scripts/
└── ubuntu/
```

Propósito de cada directorio.

| Directorio | Descripción |
|------------|-------------|
| configs | Archivos de configuración reutilizables (sin información sensible). |
| docs | Documentación técnica y bitácora del proyecto. |
| inventory | Inventario de hardware, máquinas virtuales y red. |
| proxmox | Documentación permanente relacionada con Proxmox. |
| ubuntu | Documentación permanente relacionada con Ubuntu Server. |
| scripts | Scripts desarrollados para la administración del homelab. |

---

# Convención para nombres de archivos

Todas las palabras se separan mediante guiones bajos (`_`).

La fecha siempre utiliza el formato:

```
YYYY-MM-DD
```

Formato general:

```
nombre_del_documento_YYYY-MM-DD.md
```

Ejemplos:

```
diagnostico_discos_2026-07-03.md
instalacion_proxmox_2026-07-02.md
configuracion_ssh_2026-07-02.md
instalacion_jellyfin_2026-07-15.md
```

---

# Documentación permanente

Los documentos que describen el estado actual del laboratorio **no incluyen fecha**.

Ejemplos:

```
README.md

inventory.md

proxmox.md

ubuntu_server.md
```

Estos documentos evolucionan con el proyecto y representan la configuración vigente.

---

# Bitácora técnica

Las actividades importantes generan un nuevo documento dentro de `docs/`.

Ejemplos:

```
diagnostico_discos_2026-07-03.md

instalacion_qemu_guest_agent_2026-07-03.md

configuracion_backups_2026-07-03.md
```

La bitácora representa el historial técnico del laboratorio.

Nunca debe sobrescribirse una actividad anterior.

---

# Plantilla para documentos

Todos los documentos técnicos deben seguir, cuando sea posible, la siguiente estructura:

```markdown
# Título

**Fecha:** YYYY-MM-DD

## Objetivo

...

---

## Requisitos

...

---

## Procedimiento

...

---

## Verificación

...

---

## Problemas encontrados

...

---

## Lecciones aprendidas

...

---

## Próximos pasos

...
```

Dependiendo del tipo de documento, algunas secciones podrán omitirse.

---

# Scripts

Los scripts utilizan nombres descriptivos.

Ejemplos:

```
backup_proxmox.sh

restore_vm.sh

update_proxmox.sh

check_smart.sh
```

Reglas:

- minúsculas
- palabras separadas por `_`
- nombres descriptivos

---

# Máquinas virtuales

Los nombres de las máquinas virtuales deben ser simples y descriptivos.

Ejemplos:

```
ubuntu_server

pihole

jellyfin

docker_host

monitoring

nextcloud

forgejo
```

Evitar:

- espacios
- caracteres especiales
- mayúsculas

---

# Commits

Cada cambio importante debe acompañarse de un commit descriptivo.

Formato recomendado:

```
categoria: descripción
```

Ejemplos:

```
docs: agregar diagnostico SMART de los discos

proxmox: configurar repositorio no_subscription

ubuntu: instalar qemu_guest_agent

backup: documentar estrategia de copias

inventory: agregar segundo SSD
```

Los mensajes deben ser claros y representar un único cambio lógico.

---

# Configuraciones

Las configuraciones reutilizables pueden almacenarse dentro de `configs/`.

Ejemplo:

```
configs/

└── ssh/
    ├── config.example
    └── README.md
```

Siempre deben utilizarse archivos de ejemplo cuando exista información sensible.

---

# Información sensible

Este repositorio **nunca** debe contener:

- claves privadas SSH
- certificados privados
- contraseñas
- tokens
- archivos `.env`
- API Keys
- secretos de cualquier tipo

Se utilizarán archivos `.example` para documentar configuraciones.

Ejemplos:

```
config.example

docker-compose.example.yml

.env.example
```

---

# Flujo de trabajo

Cada sesión del homelab debería seguir el siguiente ciclo:

```
Realizar cambios

↓

Verificar funcionamiento

↓

Documentar

↓

Actualizar CHANGELOG

↓

git add .

↓

git commit

↓

git push
```

---

# Objetivo final

El objetivo de este repositorio es que cualquier miembro del proyecto (incluido el autor dentro de varios años) pueda reconstruir completamente el homelab únicamente siguiendo la documentación y las configuraciones aquí almacenadas.

Toda decisión técnica importante debe quedar registrada.
