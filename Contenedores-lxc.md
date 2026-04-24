# Contenedores en Proxmox VE

## ¿Qué es un contenedor LXC?

Un contenedor LXC (Linux Containers) es un sistema Linux completo que corre de forma aislada dentro de Proxmox, compartiendo el kernel del servidor anfitrión. Es más liviano que una máquina virtual porque no necesita emular hardware.

La analogía simple: una VM es una habitación completa con sus propias paredes, techo y fundaciones. Un contenedor LXC es una habitación dentro de un edificio que comparte la estructura del edificio pero tiene su propio espacio privado.

---

## ¿Es lo mismo que Docker?

No. Aunque ambos usan el concepto de "contenedor", son tecnologías distintas:

| | LXC (Proxmox) | Docker |
|---|---|---|
| **¿Qué es?** | Un Linux completo aislado | Una sola aplicación empaquetada |
| **Ejemplo** | Mini-servidor Debian completo | Una caja que solo corre Nginx |
| **Gestión** | `apt install`, configuras todo manualmente | `docker run`, ya viene todo listo |
| **Uso típico** | Servicios que necesitan un SO completo | Microservicios, aplicaciones modernas |
| **En Proxmox** | Nativo, gestionado directamente por Proxmox | No nativo, se instala dentro de una VM o LXC |

---

## ¿Proxmox soporta Docker?

Proxmox no corre Docker de forma nativa. Tienes dos opciones para usarlo:

**Opción 1 — Docker dentro de un LXC** (más liviano)
```
Proxmox
└── Contenedor LXC (Debian/Ubuntu)
    └── Docker instalado adentro
        ├── Contenedor DVWA
        └── Contenedor Juice Shop
```
Requiere activar opciones especiales en el LXC (`nesting` y `keyctl`).

**Opción 2 — Docker dentro de una VM** (más simple)
```
Proxmox
└── VM Ubuntu
    └── Docker instalado adentro
        ├── Contenedor DVWA
        └── Contenedor Juice Shop
```
Consume más RAM pero funciona exactamente igual que Docker en cualquier Linux.

En este proyecto SOC se usa la **Opción 2** para el laboratorio vulnerable porque es más simple de enseñar y más estable.

---

## ¿Qué es una plantilla?

Para crear un contenedor necesitas una plantilla: un archivo comprimido que contiene un Linux mínimo ya preparado (Debian, Ubuntu, Alpine, etc.). Proxmox las descarga desde sus servidores y las guarda en el almacenamiento local.

Es la base sobre la cual se construye el contenedor.

### Tres formas de obtener plantillas e ISOs en Proxmox

| Método | Cuándo usarlo |
|---|---|
| **Templates** (botón en CT Templates) | Plantillas de contenedores LXC desde los servidores de Proxmox |
| **Upload** | Tienes el archivo en tu laptop y lo subes manualmente |
| **Download from URL** | Pegas una URL directa y Proxmox descarga el archivo solo |

---

## Crear un contenedor — Interfaz gráfica

### Paso 1 — Descargar la plantilla

1. Panel izquierdo → `local (pve)` → **CT Templates**
2. Haz clic en el botón **Templates**
3. Busca `debian-12` en el buscador
4. Selecciona `debian-12-standard` y haz clic en **Download**
5. Espera que aparezca **TASK OK** en el visor de tareas

> La plantilla pesa aproximadamente 118 MB. El tiempo de descarga depende de la conexión.

---

### Paso 2 — Crear el contenedor

Haz clic en **Create CT** (botón arriba a la derecha).

Se abre un asistente con varias pestañas:

**Pestaña General:**

| Campo | Valor | Descripción |
|---|---|---|
| CT ID | 100 | Número único que identifica el contenedor |
| Hostname | hola-mundo | Nombre del contenedor |
| Password | (elige una) | Contraseña del usuario root del contenedor |

**Pestaña Template:**
- Selecciona la plantilla `debian-12-standard` que descargaste

**Pestaña Disks:**
- Disk size: **4 GB**

**Pestaña CPU:**
- Cores: **1**

**Pestaña Memory:**
- Memory: **512 MB**
- Swap: **512 MB**

**Pestaña Network:**
- IPv4: **DHCP** (obtiene IP automáticamente)

**Pestaña Confirm:**
- Deja marcado **Start after created**
- Haz clic en **Finish**

---

### Paso 3 — Acceder al contenedor

Una vez creado, aparecerá en el panel izquierdo bajo el nodo `pve`:

```
Datacenter
└── pve
    └── 100 (hola-mundo)   ← tu contenedor
```

Haz clic en el contenedor → **Console** para abrir una terminal dentro de él.

---

## Crear un contenedor — Línea de comandos

El mismo contenedor anterior, pero por comandos desde la Shell de Proxmox:

```bash
# Paso 1: Actualizar lista de plantillas disponibles
pveam update

# Paso 2: Descargar la plantilla de Debian 12
pveam download local debian-12-standard_12.12-1_amd64.tar.zst

# Paso 3: Crear el contenedor
pct create 100 local:vztmpl/debian-12-standard_12.12-1_amd64.tar.zst \
  --hostname hola-mundo \
  --memory 512 \
  --cores 1 \
  --rootfs local-zfs:4 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --password 1234 \
  --unprivileged 1 \
  --start 1

# Paso 4: Entrar al contenedor
pct enter 100

# Paso 5: Salir del contenedor
exit
```

### Comandos útiles para gestionar contenedores

```bash
pct list              # Ver todos los contenedores y su estado
pct start 100         # Iniciar el contenedor 100
pct stop 100          # Detener el contenedor 100
pct enter 100         # Entrar a la consola del contenedor 100
pct destroy 100       # Eliminar el contenedor 100 (irreversible)
pct snapshot 100 snap1   # Crear snapshot del contenedor 100
pct rollback 100 snap1   # Restaurar snapshot
```

---

## Ejemplo práctico — Hola Mundo con PHP

Una vez dentro del contenedor (via consola o `pct enter 100`):

```bash
# Actualizar e instalar PHP
apt update && apt install php -y

# Crear el archivo PHP
echo "<?php echo '<h1>Hola Mundo desde Proxmox!</h1>'; ?>" > /var/www/html/index.php

# Levantar servidor PHP en el puerto 8080
php -S 0.0.0.0:8080 /var/www/html/index.php
```

Luego desde el navegador de tu laptop, entra a:
```
http://IP_DEL_CONTENEDOR:8080
```

> Para saber la IP del contenedor: dentro del contenedor ejecuta `ip a` y busca la IP de `eth0`.

---

## Diferencia entre contenedor y VM en Proxmox

| | Contenedor LXC | Máquina Virtual (KVM) |
|---|---|---|
| **Aislamiento** | Comparte kernel del host | Kernel propio, aislamiento total |
| **Velocidad de inicio** | Segundos | 1-2 minutos |
| **Consumo de RAM** | Muy bajo (usa solo lo necesario) | Alto (reserva lo asignado) |
| **Sistemas operativos** | Solo Linux | Linux, Windows, cualquier OS |
| **Snapshots** | Sí | Sí |
| **Uso en este proyecto** | Wazuh, Zabbix, servicios Linux | Lab vulnerable con Docker, Windows |

---

*← [Guía Proxmox](./README.md) | Siguiente: [Guía SOC →](../05-guia-soc/README.md)*
