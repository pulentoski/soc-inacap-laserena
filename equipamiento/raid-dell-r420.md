# Configuración RAID — Dell PowerEdge R420

## Contexto

Este documento registra el proceso real de configuración de las unidades RAID del servidor Dell PowerEdge R420, realizado durante la preparación del hardware para el proyecto SOC. Este servidor actúa como laboratorio de máquinas vulnerables, ejecutando Proxmox VE con VMs para práctica ofensiva y defensiva.

---

## Controladora RAID — PERC H310 Mini

Este servidor utiliza la controladora **Dell PERC H310 Mini**, que es más básica que la H710P del R720. Diferencias importantes:

| Característica | PERC H710P (R720) | PERC H310 (R420) |
|---|---|---|
| Write Back Cache | ✅ Soportado | ❌ No soportado |
| Hot Spare Global | ✅ Soportado | ⚠️ Limitado |
| RAID soportados | 0, 1, 5, 6, 10 | 0, 1, 5 |
| Cache dedicada | 512MB o 1GB | Sin caché dedicada |

> ⚠️ En esta controladora usar siempre **Write Through** en lugar de Write Back.

---

## Inventario de discos del servidor

### Estado inicial encontrado

Al revisar el servidor se encontró 1 Virtual Disk preconfigurado de una instalación anterior:

| VD | RAID | Capacidad | Estado |
|---|---|---|---|
| VD0 | RAID 1 | 558 GB | Ready |

### Diagnóstico de discos físicos

| Slot | Tamaño | Estado | Acción |
|---|---|---|---|
| 00 | 558 GB (~600GB) | Online ✅ | Usar en VD0 |
| 01 | 558 GB (~600GB) | Online ✅ | Usar en VD0 |
| 02 | 558 GB (~600GB) | Online ✅ | Usar en VD0 |
| 03 | — Vacío — | N/A | Disponible para expansión futura |

> ℹ️ Los valores en GB que muestra la controladora (558GB) corresponden a discos de 600GB nominales. La diferencia se debe al overhead de formateo y la conversión entre GB y GiB.

---

## Diseño RAID aplicado

### ¿Por qué un solo Virtual Disk?

A diferencia del R720, este servidor tiene solo 3 discos disponibles y un rol más acotado (laboratorio de práctica). Se usa un único VD con RAID 5 para maximizar el espacio útil manteniendo redundancia:

- Con RAID 5 en 3 discos de 558GB se obtienen **~1116GB (~1.1TB) útiles**
- Tolera la falla de 1 disco sin perder datos
- Proxmox y todas las VMs conviven en este mismo VD

### Configuración final

```
┌─────────────────────────────────────────────────────────┐
│  VD0 — RAID 5 — ~1116GB  →  Proxmox OS + VMs           │
│  ├── Slot 00: SAS 558GB                                 │
│  ├── Slot 01: SAS 558GB                                 │
│  └── Slot 02: SAS 558GB                                 │
├─────────────────────────────────────────────────────────┤
│  Slot 03: Vacío — disponible para expansión futura      │
└─────────────────────────────────────────────────────────┘
```

---

## Procedimiento de configuración

### Acceder al BIOS de la controladora PERC H310

Al encender el servidor, presionar `Ctrl + R` cuando aparezca el banner de la controladora RAID.

En modo UEFI, el acceso es por:

```
F2 → Device Settings → Integrated RAID Controller 1: Dell PERC H310 Mini
```

---

### Paso 1 — Eliminar Virtual Disks existentes

> Solo necesario si el servidor tiene VDs previos. Si está limpio, saltar al Paso 2.

Ir a:
```
Virtual Disk Management → Select Virtual Disk Operations
```

1. Seleccionar VD0 → `Delete Virtual Disk` → confirmar **YES**

Después de esto todos los discos quedan como `Unconfigured Good`.

---

### Paso 2 — Crear VD0 (Proxmox + VMs)

Ir a:
```
Virtual Disk Management → Create Virtual Disk
```

Configurar con estos valores:

| Campo | Valor |
|---|---|
| Select RAID Level | RAID 5 |
| Virtual Disk Name | Data |
| Virtual Disk Size | Dejar vacío (toma el máximo automático) |
| Strip Element Size | 64 KB |
| Read Policy | Read Ahead |
| Write Policy | Write Through |
| Initialize | Fast Initialize |

En **Select Physical Disks**, marcar los 3 discos disponibles:
- ✅ Slot 00 — SAS 558GB
- ✅ Slot 01 — SAS 558GB
- ✅ Slot 02 — SAS 558GB

→ `Apply Changes` → Confirmar

---

### Paso 3 — Configurar disco de arranque

Ir a:
```
Controller Management → Change Controller Properties → Set Bootable Device
```

Seleccionar: **Virtual Disk 0: RAID5**

→ `Apply Changes`

---

### Paso 4 — Verificación final

Ir a:
```
Virtual Disk Management → Manage Virtual Disk Properties
```

Confirmar que aparece:

```
Virtual Disk 0 → RAID 5 → ~1116GB → Ready ✅
```

Ir a:
```
Physical Disk Management → View Physical Disk Properties
```

Confirmar:

```
Slot 00 → 558GB → Online ✅  (miembro VD0)
Slot 01 → 558GB → Online ✅  (miembro VD0)
Slot 02 → 558GB → Online ✅  (miembro VD0)
Slot 03 → Vacío
```

Ir a:
```
View Controller Information
```

Confirmar:
```
Virtual Disk Count: 1
Physical Disk Count: 3
```

---

### Paso 5 — Salir e instalar Proxmox

`Finish` → el servidor reinicia.

Conectar el USB con Proxmox VE. Al encender presionar `F11` (Dell) para el menú de boot y seleccionar el USB.

Durante la instalación de Proxmox, el instalador verá:

```
Disco 1: ~1116GB → Seleccionar este para instalar Proxmox
```

---

## Arquitectura final — Cómo se usa en Proxmox

```
┌──────────────────────────────────────────────────┐
│         PROXMOX VE (instalado en VD0)            │
│                                                  │
│  ┌──────────────────┐  ┌──────────────────────┐  │
│  │  VM 1            │  │  VM 2                │  │
│  │  Metasploitable  │  │  DVWA                │  │
│  │  Disco del VD0   │  │  Disco del VD0       │  │
│  └──────────────────┘  └──────────────────────┘  │
│                                                  │
│  ┌──────────────────┐  ┌──────────────────────┐  │
│  │  VM 3            │  │  VM 4+               │  │
│  │  Windows         │  │  Máquinas VulnHub    │  │
│  │  vulnerable      │  │  Disco del VD0       │  │
│  └──────────────────┘  └──────────────────────┘  │
│                                                  │
│  Storage Pool: VD0 (~1116GB) — RAID 5 — 3xSAS   │
└──────────────────────────────────────────────────┘
```

El tráfico generado por estas VMs es capturado por los agentes Wazuh y enviado al servidor R720, permitiendo práctica de detección y respuesta a incidentes en tiempo real.

---

*← [RAID Dell R720](./raid-dell-r720.md) | [Volver al índice →](../README.md)*
