# Fase 5 — Guía SOC

## Objetivo

Instalar y configurar Wazuh, Zabbix y el laboratorio de máquinas vulnerables dentro de Proxmox VE.

> ⚠️ Esta fase asume que Proxmox ya está instalado y funcionando en los servidores. Ver [Fase 4](../04-guia-proxmox/README.md).

---

## Parte A — Instalar Wazuh (SIEM)

### A1 — Crear el contenedor LXC para Wazuh

En la interfaz web de Proxmox del **Dell R720**:

1. Descarga la plantilla de Ubuntu 22.04:
   - Panel izquierdo → `local` → **CT Templates** → **Templates**
   - Busca `ubuntu-22.04-standard` → **Download**

2. Haz clic en **Create CT** y configura:

| Campo | Valor |
|---|---|
| **CT ID** | 100 |
| **Hostname** | `wazuh-soc` |
| **Template** | ubuntu-22.04-standard |
| **Disk** | 100 GB (los logs ocupan espacio) |
| **CPU** | 4 núcleos |
| **RAM** | 8192 MB (8 GB) |
| **Swap** | 2048 MB |
| **Network** | IP estática: 192.168.10.11/24, GW: 192.168.10.1 |
| **DNS** | 8.8.8.8 |

3. **Importante:** Antes de iniciar el contenedor, ve a **Opciones** y activa:
   - `Nesting: Yes` (necesario para que Wazuh funcione dentro de LXC)

4. Inicia el contenedor → **Console**

---

### A2 — Instalar Wazuh dentro del contenedor

En la consola del contenedor `wazuh-soc`:

```bash
# Actualizar el sistema
apt update && apt upgrade -y

# Instalar dependencias básicas
apt install curl wget -y

# Ejecutar el instalador automático de Wazuh (All-in-one)
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && bash wazuh-install.sh -a
```

> El proceso tarda entre 10–20 minutos. Al terminar, mostrará las credenciales de acceso. **¡Anótalas!**

```
# Ejemplo de lo que mostrará al terminar:
INFO: --- Summary ---
INFO: You can access the web interface https://192.168.10.11
    User: admin
    Password: XXXXXXXXXXXXXX
```

---

### A3 — Acceder al Dashboard de Wazuh

1. Desde cualquier laptop en la red, abre el navegador
2. Ingresa a: `https://192.168.10.11`
3. Acepta el certificado de seguridad
4. Inicia sesión con el usuario `admin` y la contraseña generada

---

### A4 — Instalar el agente Wazuh en una máquina de prueba

Para monitorizar una máquina, debes instalar el agente en ella. Ejemplo para Ubuntu:

```bash
# En la máquina que quieres monitorizar
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring \
  --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && \
  chmod 644 /usr/share/keyrings/wazuh.gpg

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ \
  stable main" | tee /etc/apt/sources.list.d/wazuh.list

apt update && apt install wazuh-agent -y

# Configurar a qué Manager conectarse
WAZUH_MANAGER='192.168.10.11' dpkg-reconfigure wazuh-agent

# Iniciar el agente
systemctl enable wazuh-agent && systemctl start wazuh-agent
```

---

## Parte B — Instalar Zabbix (Monitoreo de infraestructura)

### B1 — Crear el contenedor LXC para Zabbix

En la interfaz de Proxmox del **Dell R720**, crea otro contenedor:

| Campo | Valor |
|---|---|
| **CT ID** | 101 |
| **Hostname** | `zabbix-monitor` |
| **Template** | ubuntu-22.04-standard |
| **Disk** | 30 GB |
| **CPU** | 2 núcleos |
| **RAM** | 2048 MB (2 GB) |
| **Network** | IP estática: 192.168.10.13/24 |

---

### B2 — Instalar Zabbix dentro del contenedor

En la consola del contenedor `zabbix-monitor`:

```bash
# Agregar repositorio de Zabbix 6.4
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
apt update

# Instalar Zabbix Server, Frontend y Agente
apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent -y

# Instalar base de datos MySQL
apt install mysql-server -y

# Configurar base de datos para Zabbix
mysql -uroot -e "
  CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
  CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'Zabbix2024!';
  GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
  FLUSH PRIVILEGES;
"

# Importar esquema de base de datos
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql -uzabbix -p zabbix

# Configurar la contraseña de BD en Zabbix
sed -i 's/# DBPassword=/DBPassword=Zabbix2024!/' /etc/zabbix/zabbix_server.conf

# Iniciar servicios
systemctl enable --now zabbix-server zabbix-agent apache2
```

---

### B3 — Acceder a Zabbix

1. Desde el navegador, ingresa a: `http://192.168.10.13/zabbix`
2. Sigue el asistente de configuración web
3. Credenciales por defecto: `Admin` / `zabbix`
4. **Cambiar la contraseña inmediatamente** desde el perfil

---

## Parte C — Laboratorio de máquinas vulnerables (Docker)

### C1 — Crear la VM Ubuntu en el HP DL180

En la interfaz de Proxmox del **HP DL180**, crea una VM (no LXC, porque Docker funciona mejor en VM):

1. Haz clic en **Create VM**:

| Campo | Valor |
|---|---|
| **VM ID** | 200 |
| **Name** | `lab-vulnerable` |
| **ISO** | Ubuntu Server 22.04 |
| **Disk** | 80 GB |
| **CPU** | 4 núcleos |
| **RAM** | 8192 MB |
| **Network** | VLAN 20 (red aislada) |

2. Instala Ubuntu Server normalmente (solo SSH server en los paquetes adicionales)

---

### C2 — Instalar Docker en la VM

En la consola de la VM `lab-vulnerable`:

```bash
# Instalar Docker
apt update
apt install ca-certificates curl -y
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  tee /etc/apt/sources.list.d/docker.list

apt update && apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y

# Verificar instalación
docker --version
```

---

### C3 — Levantar las aplicaciones vulnerables

Crea el archivo de configuración:

```bash
mkdir -p /opt/lab-vulnerable && cd /opt/lab-vulnerable
nano docker-compose.yml
```

Contenido del archivo:

```yaml
version: '3'

services:
  dvwa:
    image: vulnerables/web-dvwa
    container_name: dvwa
    ports:
      - "8080:80"
    restart: unless-stopped

  juiceshop:
    image: bkimminich/juice-shop
    container_name: juiceshop
    ports:
      - "3000:3000"
    restart: unless-stopped

  webgoat:
    image: webgoat/goat-and-wolf
    container_name: webgoat
    ports:
      - "8888:8080"
    restart: unless-stopped
```

```bash
# Levantar todos los contenedores
docker compose up -d

# Verificar que están corriendo
docker ps
```

---

### C4 — Instalar agente Wazuh en la VM del laboratorio

```bash
# Instalar el agente en la VM del laboratorio
# (mismo proceso que A4, apuntando al Manager en 192.168.10.11)
WAZUH_MANAGER='192.168.10.11' bash wazuh-agent-install.sh
```

---

## Resumen de accesos

| Servicio | URL | Usuario | Contraseña |
|---|---|---|---|
| Proxmox Dell R720 | https://192.168.10.10:8006 | root | (definida al instalar) |
| Wazuh Dashboard | https://192.168.10.11 | admin | (generada al instalar) |
| Zabbix Frontend | http://192.168.10.13/zabbix | Admin | zabbix (cambiar) |
| DVWA | http://10.0.0.10:8080 | admin | password |
| Juice Shop | http://10.0.0.10:3000 | - | - |
| WebGoat | http://10.0.0.10:8888 | - | - |

> ⚠️ Completar la columna de contraseñas al momento de la instalación real.

---

## Registro de instalación SOC

| Componente | Fecha | Versión instalada | Estado | Observaciones |
|---|---|---|---|---|
| Wazuh (All-in-one) | | | | |
| Zabbix Server | | | | |
| Docker | | | | |
| DVWA | | | | |
| Juice Shop | | | | |

---

*← [Guía Proxmox](../04-guia-proxmox/README.md) | Siguiente: [Validación →](../06-validacion/README.md)*
