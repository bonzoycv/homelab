# Diagnóstico de Discos - Homelab

**Fecha:** 2026-07-03

## Objetivo

Verificar el estado de salud de los dispositivos de almacenamiento adquiridos junto con el servidor del homelab antes de comenzar a almacenar máquinas virtuales, contenedores y datos importantes.

---

# Resumen

se uttilizaron los siguientes comandos para realizar el chequeo:

-  sudo apt install smartmontools

-  sudo smartctl

-  sudo smartctl --version

-  sudo /usr/sbin/smartctl --scan

| Disco | Modelo | Capacidad | Estado SMART | Horas de uso | Uso previsto |
|--------|---------|----------:|:------------:|-------------:|--------------|
| /dev/sdb | Samsung SSD 850 EVO | 250 GB | ✅ PASSED | 40.466 h | Sistema Proxmox |
| /dev/sda | Seagate BarraCuda ST1000DM010-2EP102 | 1 TB | ✅ PASSED | 34.980 h | Backups y almacenamiento multimedia |

---

# Disco 1

## Información general

| Campo | Valor |
|-------|-------|
| Dispositivo | `/dev/sdb` |
| Modelo | Samsung SSD 850 EVO 250GB |
| Número de serie | S2R6NX0H723493B |
| Firmware | EMT02B6Q |
| Capacidad | 250 GB |
| Interfaz | SATA III (6.0 Gb/s) |
| Tipo | SSD |

---

## Estado SMART

**Resultado**

```text
PASSED
```

No se detectan fallos críticos.

---

## Horas de uso

```text
40.466 horas
```

Aproximadamente:

- 1.686 días
- 4,6 años de funcionamiento continuo

---

## Atributos relevantes

| Atributo | Valor | Estado |
|----------|------:|--------|
| Reallocated Sector Count | 0 | ✅ |
| Wear Leveling Count | 94 | ✅ Excelente |
| Used Reserved Blocks | 0 | ✅ |
| Program Fail Count | 0 | ✅ |
| Erase Fail Count | 0 | ✅ |
| Runtime Bad Block | 0 | ✅ |
| Uncorrectable Errors | 0 | ✅ |
| CRC Error Count | 0 | ✅ |
| Temperatura | 30 °C | ✅ |

---

## Self-Test

### Short Test

```text
Completed without error
```

### Extended Test

Pendiente.

Comando:

```bash
smart -t long /dev/sdb
```

Duración estimada:

```
133 minutos
```

---

## Evaluación

El SSD se encuentra en excelente estado.

No presenta sectores defectuosos, bloques reservados utilizados ni errores de escritura.

Se utilizará como disco principal del sistema Proxmox.

---

# Disco 2

## Información general

| Campo | Valor |
|-------|-------|
| Dispositivo | `/dev/sda` |
| Modelo | Seagate BarraCuda ST1000DM010-2EP102 |
| Número de serie | Z9A9L8VF |
| Firmware | CC43 |
| Capacidad | 1 TB |
| Interfaz | SATA III |
| Velocidad | 7200 RPM |
| Tipo | HDD |

---

## Estado SMART

**Resultado**

```text
PASSED
```

No se detectan fallos críticos.

---

## Horas de uso

```text
34.980 horas
```

Aproximadamente:

- 1.457 días
- 4 años de funcionamiento continuo

---

## Atributos relevantes

| Atributo | Valor | Estado |
|----------|------:|--------|
| Reallocated Sector Count | 0 | ✅ |
| Current Pending Sector | 0 | ✅ |
| Offline Uncorrectable | 0 | ✅ |
| Reported Uncorrectable | 0 | ✅ |
| UDMA CRC Error Count | 0 | ✅ |
| Temperatura | 36 °C | ✅ |

---

## Self-Test

### Short Test

No registrado.

### Extended Test

Pendiente.

Comando:

```bash
smart -t long /dev/sda
```

Duración estimada:

```
105 minutos
```

---

## Evaluación

El disco duro presenta un excelente estado para su antigüedad.

No existen sectores reasignados ni pendientes y no se registran errores de lectura o escritura.

Se utilizará como almacenamiento para:

- Copias de seguridad de Proxmox.
- Biblioteca multimedia de Jellyfin.
- Almacenamiento general del homelab.

---

# Próximas acciones

- [x] Verificar estado SMART.
- [x] Revisar atributos SMART.
- [ ] Ejecutar Self-Test extendido del SSD.
- [ ] Ejecutar Self-Test extendido del HDD.
- [ ] Documentar el resultado de ambos Self-Test.
- [ ] Instalar el segundo SSD de 250 GB.
- [ ] Crear la política definitiva de almacenamiento del homelab.

---

# Conclusión

Los dos dispositivos analizados se encuentran en buen estado de funcionamiento y no presentan indicadores SMART que sugieran un fallo inminente.

El Samsung SSD 850 EVO será utilizado para alojar el sistema Proxmox, mientras que el Seagate BarraCuda de 1 TB se destinará al almacenamiento de copias de seguridad y contenido multimedia.

Se recomienda repetir esta revisión SMART cada seis meses o después de detectar cualquier comportamiento anómalo del sistema.
