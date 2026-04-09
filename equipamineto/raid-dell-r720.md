# Configuración RAID — Dell PowerEdge R720

## Contexto

Este documento registra el proceso real de configuración de las unidades RAID del servidor Dell PowerEdge R720, realizado durante la preparación del hardware para el proyecto SOC. Incluye el diagnóstico de discos, los problemas encontrados y la configuración final aplicada.

---

## ¿Qué es un Virtual Disk (VD)?

Antes de entrar al procedimiento, es importante entender la diferencia entre disco físico y disco virtual:

**Disco Físico (Physical Disk):** es el disco duro real, el hardware que puedes tocar e instalar en el servidor. El servidor tiene varios de estos.

**Virtual Disk (VD):** es un disco lógico que crea la controladora RAID combinando uno o más discos físicos. El sistema operativo (Proxmox) no ve los discos físicos directamente, solo ve el Virtual Disk. Por ejemplo, un VD de 5TB puede estar compuesto por 3 discos físicos de 3TB trabajando juntos en RAID 5, pero Proxmox lo ve como si fuera un solo disco.

```
Discos Físicos Reales        Controladora PERC H710P       Lo que ve Proxmox
┌──────────────┐
│  SAS 837GB   │ ──┐
│  SAS 837GB   │ ──┴──► RAID 1 ──► Virtual Disk 0 (~837GB)  → Proxmox OS
└──────────────┘
┌──────────────┐
│  SAS 2794GB  │ ──┐
│  SAS 2794GB  │ ──┤──► RAID 5 ──► Virtual Disk 1 (~5588GB) → VMs + Datos
│  SAS 2794GB  │ ──┘
└──────────────┘
```

---

## Tipos de RAID disponibles en PERC H710

| RAID | Discos mínimos | Tolerancia a fallas | Capacidad útil | Uso recomendado |
|---|---|---|---|---|
| RAID 0 | 2 | Ninguna | 100% | No usar en producción |
| RAID 1 | 2 | 1 disco | 50% | Disco de boot / OS |
| RAID 5 | 3 | 1 disco | (n-1)/n | Almacenamiento de datos |
| RAID 6 | 4 | 2 discos | (n-2)/n | Datos críticos |
| RAID 10 | 4 | 1 por par | 50% | Alta performance + redundancia |

---

## Inventario de discos del servidor

### Estado inicial encontrado

Al encender el servidor por primera vez se encontraron 3 Virtual Disks preconfigurados de una instalación anterior:

| VD | RAID | Capacidad | Estado |
|---|---|---|---|
| VD0 | RAID 1 | 837 GB | Degraded ⚠️ |
| VD1 | RAID 5 | 1.675 GB | Degraded ⚠️ |
| VD2 | RAID 5 | 5.588 GB | Ready |

**Degraded** significa que el RAID perdió uno de sus miembros (disco físico fallado o expulsado). El RAID seguía funcionando pero sin redundancia.

### Diagnóstico de discos físicos

| Slot | Tamaño | Estado | LED | Acción |
|---|---|---|---|---|
| 00 | 837 GB (1TB) | Online ✅ | Verde | Usar en VD0 |
| 01 | 837 GB (1TB) | Online ✅ | Verde | Usar en VD0 |
| 02 | 837 GB (1TB) | Online ✅ | Verde | Hot Spare |
| 03 | 837 GB (1TB) | Online ✅ | Verde | Hot Spare |
| 04 | 837 GB (1TB) | Online ✅ | Verde | Hot Spare |
| 05 | 2794 GB (3TB) | Online ✅ | Verde | Usar en VD1 |
| 06 | 2794 GB (3TB) | Online ✅ | Verde | Usar en VD1 |
| 07 | 2794 GB (3TB) | Online ✅ | Verde | Usar en VD1 |

> ⚠️ Nota: Los discos son de fabricación 2012 (modelo Seagate ST9900805SS, SAS 10K RPM). Todos los discos de 837GB tienen 13 años de uso. Se recomienda monitorear su estado con frecuencia desde iDRAC y Zabbix.

### Significado de los LEDs

| Color LED | Significado |
|---|---|
| Verde fijo | Disco OK, activo |
| Verde parpadeando | Actividad de lectura/escritura |
| Naranja/Ámbar fijo | Disco en falla o predicción de falla |
| Naranja parpadeando | RAID rebuild en progreso o disco degradado |
| Apagado | Sin disco o no detectado |

---

## Diseño RAID aplicado

### ¿Por qué dos Virtual Disks separados?

Se separa el OS de los datos por seguridad operativa:

- Si falla VD0 → se reinstala Proxmox, las VMs en VD1 quedan intactas
- Si falla VD1 → Proxmox sigue vivo, solo se restauran las VMs

Es equivalente a tener disco C: para el sistema y disco D: para los datos en un PC.

### Configuración final

```
┌─────────────────────────────────────────────────────────┐
│  VD0 — RAID 1 — ~837GB  →  Proxmox OS                  │
│  ├── Slot 00: SAS 837GB                                 │
│  └── Slot 01: SAS 837GB                                 │
├─────────────────────────────────────────────────────────┤
│  VD1 — RAID 5 — ~5588GB →  Storage VMs (Wazuh, Zabbix) │
│  ├── Slot 05: SAS 2794GB                                │
│  ├── Slot 06: SAS 2794GB                                │
│  └── Slot 07: SAS 2794GB                                │
├─────────────────────────────────────────────────────────┤
│  HOT SPARE GLOBAL (reemplazo automático ante falla)     │
│  ├── Slot 02: SAS 837GB                                 │
│  ├── Slot 03: SAS 837GB                                 │
│  └── Slot 04: SAS 837GB                                 │
└─────────────────────────────────────────────────────────┘
```

**¿Qué es un Hot Spare?** Es un disco en espera. Si falla cualquier disco del RAID, la controladora lo reemplaza automáticamente con el Hot Spare y reconstruye el array sin intervención humana.

---

## Procedimiento de configuración

### Acceder al BIOS de la controladora PERC H710

Al encender el servidor, presionar `Ctrl + R` cuando aparezca el banner de la controladora RAID (antes de que cargue el sistema operativo).

En los servidores Dell más nuevos (UEFI), el acceso es por:

```
F2 → Device Settings → Integrated RAID Controller 1: Dell PERC H710P Mini
```

---

### Paso 1 — Eliminar Virtual Disks existentes

> Solo necesario si el servidor tiene VDs previos. Si está limpio, saltar al Paso 2.

Ir a:
```
Virtual Disk Management → Select Virtual Disk Operations
```

Eliminar **siempre de mayor a menor número**:

1. Seleccionar VD2 → `Delete Virtual Disk` → confirmar **YES**
2. Seleccionar VD1 → `Delete Virtual Disk` → confirmar **YES**
3. Seleccionar VD0 → `Delete Virtual Disk` → confirmar **YES**

Después de esto todos los discos sanos quedan como `Unconfigured Good`.

---

### Paso 2 — Crear VD0 (Boot — Proxmox OS)

Ir a:
```
Virtual Disk Management → Create Virtual Disk
```

Configurar con estos valores:

| Campo | Valor |
|---|---|
| Select RAID Level | RAID 1 |
| Virtual Disk Name | Boot |
| Virtual Disk Size | Dejar vacío (toma el máximo automático) |
| Strip Element Size | 64 KB |
| Read Policy | Read Ahead |
| Write Policy | Write Back |
| Initialize | Fast Initialize |

En **Select Physical Disks**, marcar únicamente:
- ✅ Slot 00 — SAS 837GB
- ✅ Slot 01 — SAS 837GB

→ `Apply Changes` → Confirmar

---

### Paso 3 — Crear VD1 (Datos SOC — VMs)

Volver a:
```
Virtual Disk Management → Create Virtual Disk
```

Configurar:

| Campo | Valor |
|---|---|
| Select RAID Level | RAID 5 |
| Virtual Disk Name | Data |
| Virtual Disk Size | Dejar vacío (máximo) |
| Strip Element Size | 64 KB |
| Read Policy | Read Ahead |
| Write Policy | Write Back |
| Initialize | Fast Initialize |

En **Select Physical Disks**, marcar únicamente:
- ✅ Slot 05 — SAS 2794GB
- ✅ Slot 06 — SAS 2794GB
- ✅ Slot 07 — SAS 2794GB

→ `Apply Changes` → Confirmar

---

### Paso 4 — Asignar Hot Spares

Ir a:
```
Physical Disk Management → Select Physical Disk Operations
```

Repetir para cada disco sobrante:

- Seleccionar Slot 02 → `Assign Global Hot Spare` → Confirmar
- Seleccionar Slot 03 → `Assign Global Hot Spare` → Confirmar
- Seleccionar Slot 04 → `Assign Global Hot Spare` → Confirmar

---

### Paso 5 — Configurar disco de arranque

Ir a:
```
Controller Management → Change Controller Properties → Set Bootable Device
```

Seleccionar: **Virtual Disk 0: RAID1, ~837GB**

→ `Apply Changes`

---

### Paso 6 — Verificación final

Ir a:
```
Virtual Disk Management → Manage Virtual Disk Properties
```

Confirmar que aparece:

```
Virtual Disk 0 → RAID 1 → ~837GB  → Ready ✅
Virtual Disk 1 → RAID 5 → ~5588GB → Ready ✅
```

Ir a:
```
Physical Disk Management → View Physical Disk Properties
```

Confirmar:

```
Slot 00 → 837GB  → Online    ✅  (miembro VD0)
Slot 01 → 837GB  → Online    ✅  (miembro VD0)
Slot 02 → 837GB  → Hot Spare ✅
Slot 03 → 837GB  → Hot Spare ✅
Slot 04 → 837GB  → Hot Spare ✅
Slot 05 → 2794GB → Online    ✅  (miembro VD1)
Slot 06 → 2794GB → Online    ✅  (miembro VD1)
Slot 07 → 2794GB → Online    ✅  (miembro VD1)
```

Ir a:
```
View Controller Information
```

Confirmar:
```
Virtual Disk Count: 2
Physical Disk Count: 8
```

---

### Paso 7 — Salir e instalar Proxmox

`Finish` → el servidor reinicia.

Conectar el USB con Proxmox VE. Al encender presionar `F11` (Dell) para el menú de boot y seleccionar el USB.

Durante la instalación de Proxmox, el instalador verá:

```
Disco 1: ~837GB  → Seleccionar este para instalar Proxmox
Disco 2: ~5588GB → Se configura después desde Proxmox como storage de VMs
```

---

## Resumen de conceptos clave

| Término | Definición |
|---|---|
| **Physical Disk** | Disco duro real instalado en el servidor |
| **Virtual Disk (VD)** | Disco lógico creado por la controladora RAID, es lo que ve el OS |
| **RAID 1** | Espejo entre 2 discos. Si falla uno, el otro sigue funcionando |
| **RAID 5** | Striping con paridad. 3+ discos, tolera 1 falla, aprovecha mejor la capacidad |
| **Hot Spare** | Disco en espera que reemplaza automáticamente un disco fallado |
| **Degraded** | El RAID perdió un miembro, funciona pero sin redundancia |
| **Rebuilding** | La controladora está reconstruyendo un RAID con un nuevo disco |
| **Fast Initialize** | Inicialización rápida del VD, corre en segundos |
| **Full Initialize** | Inicialización completa, puede tardar horas, borra todo el disco |
| **Strip Element Size** | Tamaño de bloque de datos distribuido entre discos (64KB = uso general) |
| **Patrol Read** | Tarea automática que recorre los discos buscando errores |
| **BGI** | Background Initialization, proceso automático post-creación de VD |

---

## Arquitectura final — Cómo se usa en Proxmox

```
┌──────────────────────────────────────────────────┐
│              PROXMOX VE (instalado en VD0)       │
│                                                  │
│  ┌────────────────────────────────────────────┐  │
│  │  VM 1 — Wazuh All-in-one                   │  │
│  │  Manager + Indexer + Dashboard             │  │
│  │  RAM: ~8GB  |  Disco virtual del VD1       │  │
│  └────────────────────────────────────────────┘  │
│                                                  │
│  ┌──────────────────┐  ┌──────────────────────┐  │
│  │  VM 2 — Zabbix   │  │  VM 3 — Libre        │  │
│  │  RAM: ~2GB       │  │  Windows, otros...   │  │
│  │  Disco del VD1   │  │  Disco del VD1       │  │
│  └──────────────────┘  └──────────────────────┘  │
│                                                  │
│  Storage Pool: VD1 (~5588GB) — RAID 5 — 3xSAS   │
└──────────────────────────────────────────────────┘
```

Proxmox instala en VD0 (RAID 1, ~837GB) y desde su interfaz web asigna espacio del VD1 (RAID 5, ~5588GB) a cada VM como disco virtual.

---

*← [Equipamiento](./01-equipamiento.md) | [Volver al índice →](../README.md)*
