# Configuración RAID — Dell PowerEdge R720

## Contexto

Este documento registra el proceso real de configuración de las unidades RAID del servidor Dell PowerEdge R720, realizado durante la preparación del hardware para el proyecto SOC. Este servidor es el cerebro del SOC y ejecuta Proxmox VE como hipervisor, con Wazuh, Zabbix y T-Pot corriendo como máquinas virtuales.

---

## Especificaciones relevantes

| Característica | Detalle |
|---|---|
| Modelo | Dell PowerEdge R720 |
| RAM | 64 GB ECC DDR3 1333 MHz |
| Controladora RAID | Dell PERC H710P Mini |
| Boot Mode | UEFI |
| Virtualización CPU | Intel VT-x / VT-d habilitado ✅ |

---

## ¿Qué es un Virtual Disk (VD)?

**Disco Físico (Physical Disk):** es el disco duro real instalado en el servidor.

**Virtual Disk (VD):** es un disco lógico que crea la controladora RAID combinando uno o más discos físicos. El sistema operativo (Proxmox) no ve los discos físicos directamente, solo ve el Virtual Disk.

```
Discos Físicos Reales        Controladora PERC H710P       Lo que ve Proxmox
┌──────────────┐
│  SAS 837GB   │ ──┐
│  SAS 837GB   │ ──┴──► RAID 1 ──► Virtual Disk 0 (~837GB)  → Proxmox OS
└──────────────┘
┌──────────────┐
│  SAS 2794GB  │ ──┐
│  SAS 2794GB  │ ──┤──► ZFS RAIDZ1 ──► Pool ZFS (~5.5TB)   → VMs + Datos
│  SAS 2794GB  │ ──┘
└──────────────┘
```

---

## Tipos de RAID disponibles en PERC H710P

| RAID | Discos mínimos | Tolerancia a fallas | Capacidad útil | Uso recomendado |
|---|---|---|---|---|
| RAID 0 | 2 | Ninguna | 100% | No usar en producción |
| RAID 1 | 2 | 1 disco | 50% | Disco de boot / OS |
| RAID 5 | 3 | 1 disco | (n-1)/n | Almacenamiento de datos |
| RAID 6 | 4 | 2 discos | (n-2)/n | Datos críticos |
| RAID 10 | 4 | 1 por par | 50% | Alta performance + redundancia |

> ⚠️ **Proxmox no puede instalarse sobre un Virtual Disk en RAID 5.** El instalador requiere RAID 1 o RAID 10 para el disco del sistema operativo. Los discos de datos se gestionan con ZFS directamente desde Proxmox.

---

## Inventario de discos del servidor

### Estado inicial encontrado

Al encender el servidor se encontraron 3 Virtual Disks preconfigurados de una instalación anterior, todos en estado Degraded o con discos físicos fallados:

| VD | RAID | Capacidad | Estado |
|---|---|---|---|
| VD0 | RAID 1 | 837 GB | Degraded ⚠️ |
| VD1 | RAID 5 | 1.675 GB | Degraded ⚠️ |
| VD2 | RAID 5 | 5.588 GB | Ready |

Todos los VDs fueron eliminados para comenzar desde cero.

### Diagnóstico de discos físicos

| Slot PERC | Tamaño | Estado | LED | Acción |
|---|---|---|---|---|
| 00:01:00 | 837 GB (~1TB) | Online ✅ | Verde | Usar en VD0 RAID 1 |
| 00:01:01 | 837 GB (~1TB) | Degraded 🔴 | Naranja | Excluido — disco malo |
| 00:01:02 | 837 GB (~1TB) | Online ✅ | Verde | Usar en VD0 RAID 1 |
| 00:01:03 | 837 GB (~1TB) | Degraded 🔴 | Naranja | Excluido — disco malo |
| 00:01:04 | 837 GB (~1TB) | Online ✅ | Verde | Hot Spare global |
| 00:01:05 | 2794 GB (~3TB) | Online ✅ | Verde | ZFS RAIDZ1 |
| 00:01:06 | 2794 GB (~3TB) | Online ✅ | Verde | ZFS RAIDZ1 |
| 00:01:07 | 2794 GB (~3TB) | Online ✅ | Verde | ZFS RAIDZ1 |

> ⚠️ Los discos son de fabricación 2012 (modelo Seagate ST9900805SS, SAS 10K RPM). Se recomienda monitorear su estado con frecuencia desde iDRAC y Zabbix.

### Significado de los LEDs

| Color LED | Significado |
|---|---|
| Verde fijo | Disco OK, activo |
| Verde parpadeando | Actividad de lectura/escritura |
| Naranja/Ámbar fijo | Disco en falla o predicción de falla |
| Naranja parpadeando | RAID rebuild en progreso o disco degradado |
| Apagado | Sin disco o no detectado |

---

## Diseño final aplicado

### Configuración final

```
┌─────────────────────────────────────────────────────────┐
│  VD0 — RAID 1 — ~837GB  →  Proxmox OS                  │
│  ├── Slot 00:01:00: SAS 837GB                           │
│  └── Slot 00:01:02: SAS 837GB                           │
├─────────────────────────────────────────────────────────┤
│  HOT SPARE GLOBAL                                       │
│  └── Slot 00:01:04: SAS 837GB                           │
├─────────────────────────────────────────────────────────┤
│  UNCONFIGURED GOOD — Para ZFS en Proxmox                │
│  ├── Slot 00:01:05: SAS 2794GB                          │
│  ├── Slot 00:01:06: SAS 2794GB                          │
│  └── Slot 00:01:07: SAS 2794GB                          │
├─────────────────────────────────────────────────────────┤
│  EXCLUIDOS — Discos malos                               │
│  ├── Slot 00:01:01: SAS 837GB — Degraded                │
│  └── Slot 00:01:03: SAS 837GB — Degraded                │
└─────────────────────────────────────────────────────────┘
```

### ¿Por qué los discos de 3TB quedan sin RAID en la controladora?

Los 3 discos de 2794GB se dejan como **Unconfigured Good** intencionalmente. Proxmox incluye soporte nativo para **ZFS RAIDZ1** (equivalente a RAID 5 por software), que ofrece:

- ~5.5TB útiles con tolerancia a 1 falla de disco
- Checksums e integridad de datos
- Snapshots nativos
- Gestión completa desde la interfaz web de Proxmox

---

## Procedimiento de configuración RAID

### Acceder al BIOS de la controladora PERC H710P

Al encender el servidor presionar `Ctrl + R` cuando aparezca el banner de la controladora RAID.

En modo UEFI:
```
F2 → Device Settings → Integrated RAID Controller 1: Dell PERC H710P Mini
```

---

### Paso 1 — Eliminar Virtual Disks existentes

Ir a:
```
Virtual Disk Management → Select Virtual Disk Operations
```

Eliminar **siempre de mayor a menor número**:

1. Seleccionar VD2 → `Delete Virtual Disk` → confirmar **YES**
2. Seleccionar VD1 → `Delete Virtual Disk` → confirmar **YES**
3. Seleccionar VD0 → `Delete Virtual Disk` → confirmar **YES**

Todos los discos sanos quedan como `Unconfigured Good`.

---

### Paso 2 — Crear VD0 (Boot — Proxmox OS)

Ir a:
```
Virtual Disk Management → Create Virtual Disk
```

| Campo | Valor |
|---|---|
| Select RAID Level | RAID 1 |
| Virtual Disk Name | Data |
| Virtual Disk Size | Dejar vacío (máximo automático) |
| Strip Element Size | 64 KB |
| Read Policy | Read Ahead |
| Write Policy | Write Back |
| Initialize | Fast Initialize |

En **Select Physical Disks**, marcar únicamente:
- ✅ 00:01:00 — SAS 837GB
- ✅ 00:01:02 — SAS 837GB

→ `Apply Changes` → Confirmar

---

### Paso 3 — Asignar Hot Spare

Ir a:
```
Physical Disk Management → Select Physical Disk Operations
```

- Seleccionar 00:01:04 (837GB) → `Assign Global Hot Spare` → Confirmar

---

### Paso 4 — Verificar discos 3TB

Confirmar que los 3 discos de 2794GB quedaron como `Unconfigured Good`:

```
00:01:05 → 2794GB → Unconfigured Good ✅
00:01:06 → 2794GB → Unconfigured Good ✅
00:01:07 → 2794GB → Unconfigured Good ✅
```

> ℹ️ No crear ningún VD con estos discos — Proxmox los gestionará directamente con ZFS.

---

### Paso 5 — Configurar disco de arranque

Ir a:
```
Controller Management → Change Controller Properties → Set Bootable Device
```

Seleccionar: **Virtual Disk 0: RAID1, ~837GB**

→ `Apply Changes`

---

### Paso 6 — Verificación final RAID

```
Virtual Disk Count  : 1
Physical Disk Count : 8

VD0  → RAID 1 → ~837GB → Ready ✅

00:01:00 → 837GB  → Online    ✅  (miembro VD0)
00:01:02 → 837GB  → Online    ✅  (miembro VD0)
00:01:04 → 837GB  → Hot Spare ✅
00:01:05 → 2794GB → Unconfigured Good ✅
00:01:06 → 2794GB → Unconfigured Good ✅
00:01:07 → 2794GB → Unconfigured Good ✅
```

---

## Instalación de Proxmox VE

### Preparar el USB de instalación

En otro equipo:

1. Descargar la ISO de Proxmox VE → [https://www.proxmox.com/downloads](https://www.proxmox.com/downloads)
2. Descargar Rufus → [https://rufus.ie](https://rufus.ie)
3. Usar un USB de mínimo 8GB
4. En Rufus: seleccionar la ISO, modo **DD**, grabar

### Instalar Proxmox

1. Conectar el USB al servidor
2. Encender y presionar `F11` para el menú de boot
3. Seleccionar el USB
4. En el instalador seleccionar el disco de **~837GB** (VD0) como destino
5. Los 3 discos de 2794GB aparecerán también — **no seleccionarlos**, se configuran después

### ⚠️ Problema conocido — Proxmox no detecta discos RAID

En servidores Dell PowerEdge de generación 12 con controladoras PERC H710P, el instalador de Proxmox puede no detectar los discos. La causa es un conflicto entre la función **IOMMU** del kernel y la controladora.

**Solución — Deshabilitar IOMMU en el instalador:**

1. Arrancar desde el USB de Proxmox
2. En la pantalla de bienvenida seleccionar **"Install Proxmox VE"** pero **no presionar Enter**
3. Presionar la tecla `e` para editar los comandos de arranque
4. Ubicarse en la línea que comienza con `linux`
5. Al final de esa línea agregar:

```
intel_iommu=off
```

6. Presionar `Ctrl + X` o `F10` para iniciar la instalación

---

### Paso post-instalación — Crear ZFS RAIDZ1

Una vez instalado Proxmox, desde la interfaz web:

```
Datacenter → Node → Disks → ZFS → Create ZFS
```

| Campo | Valor |
|---|---|
| Name | data-pool |
| RAID level | RAIDZ |
| Compression | lz4 |
| Discos | Seleccionar los 3x 2794GB |

Resultado: **~5.5TB disponibles** para almacenar VMs.

---

## Arquitectura final — Cómo se usa en Proxmox

```
┌──────────────────────────────────────────────────────┐
│           PROXMOX VE (instalado en VD0)              │
│                                                      │
│  ┌──────────────────────────────────────────────┐    │
│  │  VM 1 — Wazuh All-in-one                     │    │
│  │  Manager + Indexer + Dashboard               │    │
│  │  Disco virtual del ZFS pool                  │    │
│  └──────────────────────────────────────────────┘    │
│                                                      │
│  ┌─────────────────┐  ┌─────────────────────────┐    │
│  │  VM 2 — Zabbix  │  │  VM 3 — T-Pot           │    │
│  │  Monitoreo      │  │  Honeypot               │    │
│  │  ZFS pool       │  │  ZFS pool               │    │
│  └─────────────────┘  └─────────────────────────┘    │
│                                                      │
│  ZFS RAIDZ1 (~5.5TB) — 3x SAS 2794GB                │
└──────────────────────────────────────────────────────┘
```

---

## Resumen de conceptos clave

| Término | Definición |
|---|---|
| **Physical Disk** | Disco duro real instalado en el servidor |
| **Virtual Disk (VD)** | Disco lógico creado por la controladora RAID, es lo que ve el OS |
| **RAID 1** | Espejo entre 2 discos. Si falla uno, el otro sigue funcionando |
| **RAID 5** | Striping con paridad. 3+ discos, tolera 1 falla |
| **ZFS RAIDZ1** | Equivalente a RAID 5 gestionado por Proxmox. Soportado nativamente |
| **Hot Spare** | Disco en espera que reemplaza automáticamente un disco fallado |
| **Degraded** | El RAID perdió un miembro, funciona pero sin redundancia |
| **Unconfigured Good** | Disco sin RAID asignado, disponible para uso |
| **Rebuilding** | La controladora está reconstruyendo un RAID con un nuevo disco |
| **Fast Initialize** | Inicialización rápida del VD, corre en segundos |
| **Strip Element Size** | Tamaño de bloque de datos distribuido entre discos (64KB = uso general) |
| **Patrol Read** | Tarea automática que recorre los discos buscando errores |

---

*← [Equipamiento](./01-equipamiento.md) | [Volver al índice →](../README.md)*
