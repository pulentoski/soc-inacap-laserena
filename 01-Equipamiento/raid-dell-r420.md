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

## ⚠️ Nota importante — Proxmox y RAID 5

**Proxmox VE no puede instalarse sobre un Virtual Disk en RAID 5.** El instalador requiere RAID 1 o RAID 10 para el disco de sistema operativo. Por esta razón se usa RAID 1 en este servidor.

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
| 02 | 558 GB (~600GB) | Online ✅ | Sin asignar — disponible |
| 03 | — Vacío — | N/A | Disponible para expansión futura |

> ℹ️ Los valores en GB que muestra la controladora (558GB) corresponden a discos de 600GB nominales. La diferencia se debe al overhead de formateo y la conversión entre GB y GiB.

---

## Diseño RAID aplicado

### Configuración final

```
┌─────────────────────────────────────────────────────────┐
│  VD0 — RAID 1 — ~558GB  →  Proxmox OS + VMs            │
│  ├── Slot 00: SAS 558GB                                 │
│  └── Slot 01: SAS 558GB                                 │
├─────────────────────────────────────────────────────────┤
│  Slot 02: SAS 558GB — Sin asignar                       │
│  Slot 03: Vacío — disponible para expansión futura      │
└─────────────────────────────────────────────────────────┘
```

Con RAID 1 en 2 discos se obtienen **~558GB útiles** con tolerancia a 1 falla de disco. Proxmox y todas las VMs conviven en este VD.

---

## Procedimiento de configuración

### Acceder al BIOS de la controladora PERC H310

Al encender el servidor, presionar `Ctrl + R` cuando aparezca el banner de la controladora RAID.

En modo UEFI, el acceso es por:

```
F2 → Device Settings → Integrated RAID Controller 1: Dell PERC H310 Mini
```

---

### Paso 1 — Verificar VD existente

En este servidor ya existía un VD0 en RAID 1 con los parámetros correctos. Ir a:

```
Virtual Disk Management → Manage Virtual Disk Properties
```

Confirmar que el VD0 existente tiene:

```
Virtual Disk 0 → RAID 1 → ~558GB → Ready ✅
```

Si el VD existente está correcto, no es necesario eliminarlo ni recrearlo. Continuar al Paso 2.

Si el VD no existe o está mal configurado, ir a:
```
Virtual Disk Management → Create Virtual Disk
```

Y configurar:

| Campo | Valor |
|---|---|
| Select RAID Level | RAID 1 |
| Virtual Disk Name | Boot |
| Virtual Disk Size | Dejar vacío (máximo automático) |
| Strip Element Size | 64 KB |
| Read Policy | Read Ahead |
| Write Policy | Write Through |
| Initialize | Fast Initialize |

En **Select Physical Disks**, marcar únicamente:
- ✅ Slot 00 — SAS 558GB
- ✅ Slot 01 — SAS 558GB

→ `Apply Changes` → Confirmar

---

### Paso 2 — Configurar disco de arranque

Ir a:
```
Controller Management → Change Controller Properties → Set Bootable Device
```

Seleccionar: **Virtual Disk 0: RAID1, ~558GB**

→ `Apply Changes`

---

### Paso 3 — Verificación final RAID

Ir a:
```
Virtual Disk Management → Manage Virtual Disk Properties
```

Confirmar:

```
Virtual Disk 0 → RAID 1 → ~558GB → Ready ✅
```

Ir a:
```
Physical Disk Management → View Physical Disk Properties
```

Confirmar:

```
Slot 00 → 558GB → Online ✅  (miembro VD0)
Slot 01 → 558GB → Online ✅  (miembro VD0)
Slot 02 → 558GB → Online ✅  (sin asignar)
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

### Paso 4 — Instalación de Proxmox VE

`Finish` → el servidor reinicia.

Conectar el USB con Proxmox VE. Al encender presionar `F11` (Dell) para el menú de boot y seleccionar el USB.

Durante la instalación de Proxmox, el instalador verá:

```
Disco 1: ~558GB → Seleccionar este para instalar Proxmox
```

---

### ⚠️ Problema conocido — Proxmox no detecta discos RAID

**En este servidor se presentó el siguiente problema:** al iniciar el instalador de Proxmox, los discos RAID no eran detectados. La causa es que la función **IOMMU** del kernel entra en conflicto con la controladora H310 en servidores Dell de esta generación.

**Solución — Deshabilitar IOMMU en el instalador:**

1. Arrancar desde el USB de Proxmox.
2. En la pantalla de bienvenida seleccionar **"Install Proxmox VE"** pero **no presionar Enter**.
3. Presionar la tecla `e` para editar los comandos de arranque.
4. Usar las flechas del teclado para ubicarse en la línea que comienza con `linux`.
5. Al final de esa línea agregar:

```
intel_iommu=off
```

6. Presionar `Ctrl + X` o `F10` para iniciar la instalación con este parámetro.

A partir de este punto el instalador detecta correctamente el disco RAID y la instalación procede con normalidad.

> ℹ️ Este problema es conocido en servidores Dell PowerEdge de generación 12 (R420, R620, R720) con controladoras PERC H310/H710 al instalar distribuciones Linux modernas. El parámetro `intel_iommu=off` deshabilita la virtualización de E/S que causa el conflicto.

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
│  Storage Pool: VD0 (~558GB) — RAID 1 — 2xSAS    │
└──────────────────────────────────────────────────┘
```

El tráfico generado por estas VMs es capturado por los agentes Wazuh y enviado al servidor R720, permitiendo práctica de detección y respuesta a incidentes en tiempo real.

---

*← [RAID Dell R720](./raid-dell-r720.md) | [Volver al índice →](../README.md)*
