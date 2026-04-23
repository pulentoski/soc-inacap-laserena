# Fase 5 — Guía Wazuh (Servidor)

## Objetivo de esta fase

Instalar Wazuh all-in-one en el servidor Dell R720 usando un contenedor LXC Ubuntu 22.04 en Proxmox VE.

---

## ¿Qué es Wazuh?

Wazuh es una plataforma open source de seguridad (XDR/SIEM) que incluye tres componentes principales:

| Componente | Función |
|---|---|
| **Wazuh Indexer** | Almacena y indexa los eventos (basado en OpenSearch) |
| **Wazuh Manager** | Recibe y procesa alertas de los agentes |
| **Wazuh Dashboard** | Interfaz web de visualización y análisis |

En esta guía se instalan los tres componentes en un solo contenedor LXC (all-in-one).

---

## Requisitos del sistema

| Agentes | CPU | RAM | Disco |
|---|---|---|---|
| 1-25 | 4 vCPU | 8 GB | 50 GB |
| 25-50 | 8 vCPU | 8 GB | 100 GB |
| 50-100 | 8 vCPU | 8 GB | 200 GB |

---

## Especificaciones del contenedor

| Parámetro | Valor |
|---|---|
| **CT ID** | 200 |
| **Tipo** | LXC sin privilegios |
| **Hostname** | `wazuh` |
| **OS** | Ubuntu 22.04 LTS |
| **CPU** | 4 cores |
| **RAM** | 8192 MB |
| **Swap** | 2048 MB |
| **Disco** | 50 GB en `sotorage-vms` |
| **IP** | `10.0.0.4/24` |
| **Gateway** | `10.0.0.1` |
| **DNS** | `8.8.8.8` |
| **Versión Wazuh** | 4.14.5 |

---

## Paso 1 — Descargar template Ubuntu 22.04

Desde la interfaz web de Proxmox:

1. Selecciona **local** → pestaña **CT Templates**
2. Haz clic en **Templates**
3. Busca `ubuntu-22.04-standard`
4. Selecciónalo y haz clic en **Download**

O desde la shell de Proxmox:

```bash
pveam update
pveam download local ubuntu-22.04-standard_22.04-1_amd64.tar.zst
```

---

## Paso 2 — Crear el contenedor LXC

Desde la interfaz web: **Create CT** y configura:

| Pantalla | Campo | Valor |
|---|---|---|
| General | CT ID | 200 |
| General | Hostname | wazuh |
| General | Password | (elige una contraseña) |
| Template | Template | ubuntu-22.04-standard |
| Disks | Storage | sotorage-vms |
| Disks | Disk size | 50 GB |
| CPU | Cores | 4 |
| Memory | Memory | 8192 MB |
| Memory | Swap | 2048 MB |
| Network | IPv4 | 10.0.0.4/24 |
| Network | Gateway | 10.0.0.1 |
| DNS | DNS | 8.8.8.8 |

Marca **"Start after created"** y haz clic en **Finish**.

O desde la shell de Proxmox:

```bash
pct create 200 local:vztmpl/ubuntu-22.04-standard_22.04-1_amd64.tar.zst \
  --hostname wazuh \
  --cores 4 \
  --memory 8192 \
  --swap 2048 \
  --rootfs sotorage-vms:50 \
  --net0 name=eth0,bridge=vmbr0,ip=10.0.0.4/24,gw=10.0.0.1 \
  --nameserver 8.8.8.8 \
  --unprivileged 1 \
  --start 1
```

---

## Paso 3 — Entrar al contenedor

```bash
pct enter 200
```

---

## Paso 4 — Actualizar el sistema

```bash
apt update && apt upgrade -y
```

---

## Paso 5 — Instalar Wazuh

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && bash ./wazuh-install.sh -a
```

> Este proceso instala automáticamente el Indexer, Manager, Filebeat y Dashboard. Tarda entre 15-20 minutos.

> ⚠️ Si aparece advertencia de recursos insuficientes, agrega el parámetro `-i`:
> ```bash
> curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && bash ./wazuh-install.sh -a -i
> ```

> ℹ️ Los errores `tr: write error: Broken pipe` durante la generación de certificados son normales y no afectan la instalación.

Al finalizar verás el resumen con las credenciales:

```
INFO: --- Summary ---
INFO: You can access the web interface https://<wazuh-dashboard-ip>:443
    User: admin
    Password: <CONTRASEÑA_GENERADA>
INFO: Installation finished.
```

> ⚠️ Guarda la contraseña generada en un lugar seguro.

---

## Paso 6 — Acceder al dashboard

Desde cualquier equipo en la misma red, abre el navegador:

```
https://10.0.0.4
```

Credenciales:

```
Usuario: admin
Contraseña: (la generada en el paso anterior)
```

> ⚠️ El navegador mostrará advertencia de certificado autofirmado. Haz clic en "Avanzado" → "Continuar de todos modos".

---

## Paso 7 — Verificar servicios

Desde dentro del contenedor:

```bash
systemctl status wazuh-manager
systemctl status wazuh-indexer
systemctl status wazuh-dashboard
systemctl status filebeat
```

Todos deben mostrar **active (running)**.

---

## Archivos de configuración importantes

| Componente | Ruta |
|---|---|
| **Wazuh Manager** | `/var/ossec/etc/ossec.conf` |
| **Wazuh Indexer** | `/etc/wazuh-indexer/opensearch.yml` |
| **Filebeat** | `/etc/filebeat/filebeat.yml` |
| **Wazuh Dashboard** | `/etc/wazuh-dashboard/opensearch_dashboards.yml` |
| **Contraseñas** | `/var/log/wazuh-install.log` |

---

## Solución de problemas comunes

| Problema | Solución |
|---|---|
| Dashboard no carga | Verificar: `systemctl status wazuh-dashboard` |
| Error `tr: write error: Broken pipe` | Normal, no afecta la instalación |
| Advertencia de recursos insuficientes | Agregar `-i` al comando de instalación |
| No puedo acceder a `https://10.0.0.4` | Verificar IP con `ip a` dentro del contenedor |
| Error de certificado en el navegador | Normal, es autofirmado. Aceptar la excepción |
| Olvidé la contraseña | Recuperar con: `tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt` |

---

## Registro de instalación

> Completar a medida que se instale.

| Campo | Valor |
|---|---|
| Fecha instalación | |
| Versión Wazuh | 4.14.5 |
| IP asignada | 10.0.0.4 |
| CT ID en Proxmox | 200 |
| Observaciones | |

---

*← [Guía Proxmox](../04-guia-proxmox/README.md) | Siguiente: [Agentes Wazuh →](../06-guia-agentes/README.md)*
