# 🖥️ Lab: Instalación Proxmox VE en Dell PowerEdge R720 con RAID PERC H710P

## Descripción

Laboratorio completo de configuración de almacenamiento RAID y instalación de Proxmox VE en un servidor Dell PowerEdge R720, aprovechando 6 discos físicos organizados en dos grupos RAID independientes.

---

## Hardware utilizado

| Componente | Detalle |
|---|---|
| Servidor | Dell PowerEdge R720 |
| Controlador RAID | PERC H710P |
| Discos sistema | 3x 900GB SAS |
| Discos almacenamiento | 3x 3TB SAS |
| NICs | 8x interfaces (nic0 a nic7) |
| IP del servidor | 10.0.0.3 |

---

## Arquitectura final

```
Discos físicos:
├── Slot 0 — 900GB — RAID 1 (sistema)
├── Slot 1 — 900GB — RAID 1 (sistema)
├── Slot 2 — 900GB — Hot Spare (repuesto automático)
├── Slot 3 — 3TB  — Sin RAID (ext4)
├── Slot 4 — 3TB  — Sin RAID (ext4)
└── Slot 5 — 3TB  — Sin RAID (ext4)

Discos virtuales exportados por el PERC:
├── VD0 — RAID 1 — 837GB  → /dev/sda → Sistema Proxmox
└── /dev/sdb — Sin RAID — 5.5TB → particionado ext4 → /mnt/storage-vms

Storage en Proxmox:
├── local        —  96GB  — Sistema base
├── local-lvm    — 701GB  — Discos de VMs (LVM)
└── storage-vms  —  5.5TB — ISOs, backups, contenedores
```

---

## Paso 1 — Configurar RAID en el PERC H710P

### Acceder al controlador

Al encender el servidor, presionar `Ctrl + R` cuando aparezca:
```
Press Ctrl+R to run the PERC Configuration Utility
```

### Crear RAID 1 (sistema — 2x 900GB)

```
1. Seleccionar Controller 0: PERC H710P → F2
2. Create New VD
   - RAID Level:   RAID-1
   - Physical Disks: Slot 0 + Slot 1
   - VD Name:      OS-Proxmox
   - Strip Size:   64KB
   - Write Policy: Write Back
   - Read Policy:  Read Ahead
3. OK → Fast Initialize → Yes
```

### Asignar Hot Spare (tercer disco de 900GB)

```
1. Seleccionar Slot 2 (900GB) → F2
2. Make Global Hot Spare → Yes
```

### Dejar discos de 3TB sin RAID en el PERC

Los 3 discos de 3TB **no se configuran en el PERC**. Se dejan como `Unconfigured Good` o `Ready`. El particionado y formateo se realiza directamente en Proxmox.

```
Los slots 3, 4 y 5 (3TB cada uno) deben aparecer como:
→ Unconfigured Good ✅ (no crear VD con ellos)
```

### Guardar y salir

```
Esc → Save Configuration → Yes → el servidor reinicia
```

> **Nota:** Si los discos aparecen como `Unconfigured Good` o `Ready`, están perfectamente disponibles. Si aparecen como `Foreign`, hacer primero: F2 → Clear Foreign Config.

---

## Paso 2 — Instalar Proxmox VE

### Preparar USB booteable

Descargar Proxmox VE desde [https://www.proxmox.com/downloads](https://www.proxmox.com/downloads) y grabarlo con Rufus o Balena Etcher.

### Bootear desde USB

```
Al encender el servidor → F11 → Boot Menu → seleccionar USB
```

### Instalación gráfica

```
1. Seleccionar: Install Proxmox VE (Graphical)

2. Target Harddisk:
   → /dev/sda (837GB — VD0 RAID1) ← el sistema va aquí
   
3. Filesystem: ext4
   ⚠️ NO usar ZFS — el RAID ya está hecho por hardware (PERC)

4. Configuración regional:
   - Country:   Chile
   - Timezone:  America/Santiago
   - Keyboard:  Spanish (Latin American)

5. Credenciales:
   - Password: (contraseña root)
   - Email:    (email administrador)

6. Red:
   - Hostname:  proxmox.local
   - IP:        10.0.0.3/24
   - Gateway:   10.0.0.1
   - DNS:       8.8.8.8

7. Install → reinicia automáticamente
```

---

## Paso 3 — Configuración post-instalación

### Acceder al panel web

```
https://10.0.0.3:8006
Usuario: root
```

### Corregir repositorios (sin licencia enterprise)

Proxmox instala por defecto repos enterprise que requieren suscripción paga. Hay que desactivarlos:

```bash
# Desactivar repos enterprise en formato .list
sed -i 's/^deb/# deb/' /etc/apt/sources.list.d/pve-enterprise.list
sed -i 's/^deb/# deb/' /etc/apt/sources.list.d/ceph.list

# Desactivar repos enterprise en formato .sources (formato nuevo apt)
sed -i '1s/^/Enabled: no\n/' /etc/apt/sources.list.d/pve-enterprise.sources
sed -i '1s/^/Enabled: no\n/' /etc/apt/sources.list.d/ceph.sources

# Agregar repo gratuito de Proxmox
echo "deb http://download.proxmox.com/debian/pve trixie pve-no-subscription" \
  > /etc/apt/sources.list.d/pve-no-sub.list
```

### Sincronizar el reloj del servidor

```bash
timedatectl set-ntp false
timedatectl set-time "YYYY-MM-DD HH:MM:SS"  # ajustar a hora actual
timedatectl set-ntp true
date
```

> **Importante:** Si el reloj está desincronizado, apt falla al verificar firmas de los repositorios con error `Not live until...`

### Actualizar Proxmox

```bash
apt update && apt upgrade -y
```

---

## Paso 4 — Configurar almacenamiento de 5.5TB

### Verificar que el disco aparece

```bash
lsblk
```

Resultado esperado:
```
sda  837.8G  disk   ← sistema (RAID1)
sdb    5.5T  disk   ← almacenamiento (sin RAID) ✅
```

### Particionar, formatear y montar

```bash
# Instalar parted si no está disponible
apt install -y parted

# Crear tabla de particiones GPT y partición única
parted /dev/sdb mklabel gpt
parted /dev/sdb mkpart primary ext4 0% 100%

# Formatear en ext4
mkfs.ext4 /dev/sdb1

# Crear punto de montaje y montar
mkdir /mnt/storage-vms
mount /dev/sdb1 /mnt/storage-vms

# Montaje permanente al reiniciar
echo '/dev/sdb1 /mnt/storage-vms ext4 defaults 0 2' >> /etc/fstab

# Verificar
df -h | grep storage
```

Resultado esperado:
```
/dev/sdb1   5.5T   2.1M   5.2T   1%   /mnt/storage-vms
```

### Agregar storage al panel web de Proxmox

```
Centro de datos → Almacenamiento → Agregar → Directorio

ID:          storage-vms
Directorio:  /mnt/storage-vms
Contenido:   ✓ Imagen de disco
             ✓ Imagen ISO
             ✓ Copia de seguridad VZDump
             ✓ Plantilla de contenedor

→ Agregar
```

---

## Problemas encontrados y soluciones

### ❌ Error 401 en repositorios enterprise
**Causa:** Proxmox instala repos enterprise por defecto, requieren suscripción paga.  
**Solución:** Desactivar archivos `.list` y `.sources` enterprise, agregar repo `pve-no-subscription`.

### ❌ Error "Not live until" al hacer apt update
**Causa:** El reloj del servidor estaba ~4 horas atrasado, las firmas GPG fallaban.  
**Solución:** Sincronizar hora con `timedatectl` antes de correr apt.

### ❌ El disco de 5.5TB no aparecía en lsblk
**Causa:** El RAID 5 estaba creado en el PERC pero sin inicializar (Fast Initialize pendiente).  
**Solución:** Entrar al PERC con Ctrl+R, verificar estado del VD1 y ejecutar Fast Initialize.

### ❌ perccli no disponible en repositorios
**Causa:** perccli es una herramienta propietaria de Dell/Broadcom, no está en repos de Debian.  
**Alternativa:** Verificar discos con `lsblk`, `dmesg | grep scsi` y `cat /proc/scsi/scsi`.

---

## Comandos útiles de diagnóstico

```bash
# Ver discos y particiones
lsblk -d -o NAME,SIZE,TYPE,VENDOR,MODEL

# Ver uso de almacenamiento
df -h

# Ver logs del controlador RAID
dmesg | grep -i "sd\|scsi\|raid\|perc" | tail -30

# Ver dispositivos SCSI detectados
cat /proc/scsi/scsi

# Ver interfaces de red
ip a

# Ver velocidad de NIC
ethtool nic0 | grep -E "Speed|Duplex"

# Ver repositorios activos
grep -r "enterprise" /etc/apt/sources.list.d/

# Ver estado del reloj
timedatectl status
```

---

## Estado final del servidor

```
Panel web:   https://10.0.0.3:8006
SO:          Proxmox VE (Debian trixie)
Kernel:      proxmox-kernel-6.17

Almacenamiento:
├── local        96GB   activo ✅
├── local-lvm   701GB   activo ✅
└── storage-vms  5.5TB  activo ✅

Red:
└── vmbr0  10.0.0.3/24  UP ✅ (bridge sobre nic0)
    nic1 ~ nic7 disponibles para configurar
```

---

## Próximos pasos sugeridos

- [ ] Subir ISOs para instalación de VMs
- [ ] Crear primera VM de prueba
- [ ] Configurar backup automático en storage-vms
- [ ] Configurar NICs adicionales (nic1~nic7) para red de VMs
- [ ] Configurar acceso iDRAC para gestión remota
- [ ] Agregar usuario administrador no-root en Proxmox

---

*Laboratorio realizado en Dell PowerEdge R720 con Proxmox VE sobre Debian trixie*
