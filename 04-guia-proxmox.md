# Fase 4 — Guía Proxmox VE

## Objetivo de esta fase

Aprender a instalar y usar Proxmox VE **primero en VirtualBox** (en tu laptop) y luego replicar el proceso en los servidores reales del rack.

---

## Parte A — Instalación en VirtualBox (Práctica)

> Esta parte se realiza en la laptop del docente o del alumno antes de tocar los servidores físicos.

### Requisitos

- VirtualBox 7.x instalado ([descargar aquí](https://www.virtualbox.org/wiki/Downloads))
- ISO de Proxmox VE 8.x ([descargar aquí](https://www.proxmox.com/en/downloads))
- Laptop con mínimo **16 GB de RAM** y CPU con VT-x habilitado
- **8 GB de RAM disponibles** para asignar a la VM

---

### Paso 1 — Verificar VT-x en la BIOS de tu laptop

Antes de crear la VM, asegúrate de que tu laptop tiene la virtualización activada:

1. Reinicia la laptop
2. Presiona `F2` (en ASUS) al arrancar para entrar a la BIOS
3. Busca la opción **"Intel Virtualization Technology"** o **"VT-x"**
4. Verifica que esté en **Enabled**
5. Guarda y reinicia

> ✅ Si ya usas VirtualBox normalmente, probablemente ya está habilitado.

---

### Paso 2 — Crear la VM en VirtualBox

1. Abre VirtualBox y haz clic en **"Nueva"**
2. Configura lo siguiente:

| Campo | Valor |
|---|---|
| **Nombre** | `Proxmox-Practica` |
| **Tipo** | Linux |
| **Versión** | Debian (64-bit) |
| **RAM** | 8192 MB (8 GB) |
| **CPU** | 4 núcleos |
| **Disco duro** | Crear nuevo → VDI → Dinámico → **60 GB** |

3. Haz clic en **Crear**

---

### Paso 3 — Habilitar virtualización anidada (CRÍTICO)

> Sin este paso, Proxmox se instalará pero no te dejará crear VMs ni contenedores adentro.

1. Con la VM creada (sin encender), ve a **Configuración → Sistema → Procesador**
2. Marca la casilla **"Nested VT-x/AMD-V"**

Si la casilla aparece gris (bloqueada), debes habilitarla desde la terminal. El procedimiento varía según tu sistema operativo anfitrión.

#### En Linux (Ubuntu) — sistema del docente

Las VMs en VirtualBox se registran por usuario, no de forma global. Si ejecutas como root y no ves tus VMs, es porque están registradas bajo tu usuario normal.

Primero identifica el nombre exacto de tu VM:

```bash
# Si estás con tu usuario normal
VBoxManage list vms

# Si estás como root, consulta las VMs del usuario normal (ejemplo: usuario "miusuario")
su - TU_USUARIO -c "VBoxManage list vms"
```

La salida mostrará algo como:
```
"proxmox" {e3a7bed9-83fd-49e5-8bc7-75eed7d0bddf}
```

Ahora habilita la virtualización anidada con el nombre exacto que apareció:

```bash
# Con tu usuario normal
VBoxManage modifyvm "proxmox" --nested-hw-virt on

# O si estás como root y la VM pertenece al usuario normal
su - TU_USUARIO -c "VBoxManage modifyvm 'proxmox' --nested-hw-virt on"
```

Verifica que quedó habilitado correctamente:

```bash
su - TU_USUARIO -c "VBoxManage showvminfo proxmox | grep -i nested"

# Debe mostrar:
# Nested VT-x/AMD-V:  enabled
# Nested Paging:      enabled
```

#### En Windows

```cmd
# Abrir CMD o PowerShell como Administrador
VBoxManage modifyvm "Proxmox-Practica" --nested-hw-virt on
```

#### En macOS

```bash
VBoxManage modifyvm "Proxmox-Practica" --nested-hw-virt on
```

Después de ejecutar el comando, vuelve a abrir VirtualBox y verifica que la casilla **Nested VT-x/AMD-V** ya aparece marcada.

---

### Paso 4 — Cargar la ISO de Proxmox

1. Ve a **Configuración → Almacenamiento**
2. Haz clic en el ícono de CD vacío
3. Haz clic en el ícono de disco a la derecha → "Elegir un archivo de disco"
4. Selecciona el archivo `.iso` de Proxmox que descargaste

---

### Paso 5 — Configurar la red en VirtualBox

1. Ve a **Configuración → Red**
2. **Adaptador 1:** Modo Puente (Bridge) — para que Proxmox obtenga una IP de tu red local

> Si modo puente no funciona, usa "NAT" como alternativa.

---

### Paso 6 — Instalar Proxmox

1. Enciende la VM
2. Verás el menú de arranque de Proxmox → selecciona **"Install Proxmox VE (Graphical)"**
3. Acepta la licencia
4. **Target Disk:** Selecciona el disco virtual que creaste (el de 60 GB)
5. **Country:** Chile | **Timezone:** America/Santiago | **Keyboard:** Latam
6. **Contraseña:** Elige una robusta (la necesitarás para acceder)
7. **Email:** Tu correo institucional
8. **Configuración de red:**

| Campo | Valor |
|---|---|
| **Hostname** | `proxmox-practica.local` |
| **IP Address** | `192.168.X.X/24` (la que te asigne tu red) |
| **Gateway** | IP de tu router (ej: 192.168.1.1) |
| **DNS** | 8.8.8.8 |

9. Haz clic en **Install** y espera unos 5–10 minutos
10. Al terminar, la VM se reiniciará y mostrará una pantalla negra con el texto:

```
Welcome to the Proxmox Virtual Environment. Please use your web browser to configure 
this server - connect to:

  https://192.168.X.X:8006/
```

---

### Paso 7 — Post-instalación (script recomendado)

Una vez que Proxmox arranca, antes de hacer cualquier otra cosa, ejecuta el script de post-instalación de la comunidad. Ve a `pve` (panel izquierdo) → **Shell** y ejecuta:

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/tools/pve/post-pve-install.sh)"
```

Responde **"y"** a todo lo que pregunte. El script realiza lo siguiente:

| Acción | Por qué |
|---|---|
| Deshabilita repo Enterprise | Evita errores 401 (requiere suscripción de pago) |
| Activa repo No-Subscription | Repositorio gratuito para actualizaciones |
| Deshabilita el aviso de suscripción | Elimina el pop-up molesto al entrar |
| Deshabilita HA y Corosync | No necesarios en un solo nodo |
| Actualiza los paquetes | Deja el sistema al día |

> ⚠️ Si el script muestra `apt update failed` al final, ver el paso 7b.

---

### Paso 7b — Solución: error 401 en apt update (Proxmox 9.x)

En Proxmox 9.x existen archivos `.sources` que el script no deshabilita. Síntoma:

```
Err: https://enterprise.proxmox.com/debian/pve trixie InRelease
  401  Unauthorized
```

Solución:

```bash
# Verificar qué archivos tienen repos enterprise activos
grep -r "enterprise.proxmox" /etc/apt/

# Agregar "Enabled: no" a los archivos .sources
echo "Enabled: no" >> /etc/apt/sources.list.d/pve-enterprise.sources
echo "Enabled: no" >> /etc/apt/sources.list.d/ceph.sources

# Verificar
cat /etc/apt/sources.list.d/pve-enterprise.sources
cat /etc/apt/sources.list.d/ceph.sources

# Actualizar sin errores
apt update
```

---

### Paso 8 — Acceder a la interfaz web de Proxmox

1. **No cierres VirtualBox**, déjalo minimizado
2. Abre Chrome o Firefox en tu laptop
3. Escribe la dirección que apareció: `https://192.168.X.X:8006`
4. El navegador mostrará una advertencia de seguridad (certificado autofirmado) → haz clic en **"Avanzado" → "Continuar de todos modos"**
5. Inicia sesión:
   - **Usuario:** `root`
   - **Contraseña:** la que configuraste en el paso anterior
   - **Realm:** Linux PAM

¡Listo! Estás dentro de Proxmox VE.

---

### Paso 8 — Primeras acciones en la interfaz

Una vez dentro, familiarízate con estos elementos:

```
Panel izquierdo:
├── Datacenter          ← Vista general del clúster
│   └── proxmox-practica (nodo)
│       ├── local       ← Almacenamiento local (aquí subirás ISOs)
│       └── local-lvm   ← Almacenamiento para VMs y contenedores

Botones superiores:
├── Create VM           ← Crear una máquina virtual (KVM)
└── Create CT           ← Crear un contenedor (LXC)
```

#### Subir una ISO (para practicar)

1. Haz clic en **local** (en el panel izquierdo)
2. Ve a la pestaña **ISO Images**
3. Haz clic en **Upload** y sube la ISO de Ubuntu Server 22.04

---

### Paso 9 — Crear tu primer contenedor LXC

Los contenedores son más ligeros que las VMs. Prueba crear uno:

1. Haz clic en **Create CT** (botón superior derecho)
2. Completa los campos:

| Campo | Valor |
|---|---|
| **CT ID** | 100 |
| **Hostname** | `ubuntu-test` |
| **Password** | Una contraseña para el root del contenedor |
| **Template** | ubuntu-22.04-standard (debes descargarlo primero) |
| **Disk** | 8 GB |
| **CPU** | 1 núcleo |
| **RAM** | 512 MB |
| **Network** | DHCP |

3. Haz clic en **Finish**
4. El contenedor aparecerá en el panel izquierdo
5. Selecciónalo → haz clic en **Start** → luego **Console**
6. Verás una terminal de Ubuntu corriendo dentro de Proxmox 🎉

---

## Parte B — Instalación en servidores reales (Dell/HP)

> Esta parte se realiza una vez que el rack esté instalado en la sala de servidores.

### Diferencias respecto a VirtualBox

| | VirtualBox (práctica) | Servidor real |
|---|---|---|
| **Boot** | Desde ISO en VirtualBox | Desde USB con Proxmox |
| **RAID** | No aplica | Configurar antes de instalar |
| **VT-x** | Habilitar en VirtualBox | Habilitar en BIOS del servidor |
| **Red** | IP de tu LAN hogareña | IP de la red de INACAP |

---

### Paso B1 — Grabar ISO en USB

Descarga **Balena Etcher** (https://etcher.balena.io/) y graba la ISO de Proxmox en un pendrive USB de al menos 4 GB.

---

### Paso B2 — Configurar RAID en Dell R720

1. Conecta teclado y monitor via KVM
2. Enciende el servidor
3. Cuando aparezca el POST (pantalla con logo Dell), presiona `Ctrl + R` para entrar a la controladora PERC
4. En la utilidad de RAID:

```
Acción: Create New VD (Virtual Disk)

VD 1 — Para el sistema operativo:
  RAID Level: RAID 1 (Mirror)
  Discos: Selecciona 2 discos de 600 GB
  Nombre: OS_Drive

VD 2 — Para almacenamiento de VMs:
  RAID Level: RAID 5
  Discos: Los discos de 3TB restantes
  Nombre: VM_Storage
```

5. Guarda y reinicia

---

### Paso B3 — Instalar Proxmox desde USB

1. Inserta el USB con Proxmox
2. Al encender el servidor, presiona `F11` (Dell) para el menú de boot
3. Selecciona el USB
4. Sigue los mismos pasos de instalación que en VirtualBox (Paso 6), pero con las IPs reales de INACAP

---

### Paso B4 — Acceder desde la red de INACAP

Una vez instalado, desde cualquier laptop en la red de INACAP:

```
https://IP_DEL_SERVIDOR:8006
```

---

## Solución de problemas comunes

| Problema | Sistema | Solución |
|---|---|---|
| `VBoxManage list vms` no muestra nada ejecutando como root | Linux | Las VMs pertenecen al usuario normal. Usar `su - USUARIO -c "VBoxManage list vms"` |
| Error: `Could not find a registered machine named 'proxmox'` | Linux/Windows | El nombre no coincide. Listar primero con `VBoxManage list vms` y copiar el nombre exacto |
| La casilla Nested VT-x está gris en VirtualBox | Todos | Ejecutar `VBoxManage modifyvm "NOMBRE" --nested-hw-virt on` con el usuario dueño de la VM |
| Proxmox no arranca en el servidor físico | Servidor | Verificar que VT-x está habilitado en la BIOS del servidor |
| No puedo acceder a la interfaz web `:8006` | Todos | Verificar que la red es Adaptador Puente (no NAT). Probar `ping IP_PROXMOX` desde la laptop |
| El servidor físico no reconoce los discos | Servidor | Entrar al RAID controller (Ctrl+R en Dell) y crear el Virtual Disk primero |
| Error de certificado en el navegador al abrir Proxmox | Todos | Es normal (certificado autofirmado). Clic en "Avanzado" → "Continuar de todos modos" |

---

## Registro de instalación

> Completar a medida que se instale.

| Servidor | Fecha instalación | IP asignada | Versión Proxmox | Observaciones |
|---|---|---|---|---|
| Laptop (VirtualBox) | | | | |
| Dell R720 | | | | |
| Dell R420 | | | | |
| HP DL180 G6 | | | | |

---

*← [Planificación](../03-planificacion/README.md) | Siguiente: [Guía SOC →](../05-guia-soc/README.md)*
