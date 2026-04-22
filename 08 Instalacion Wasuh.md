# Fase 5 — Guía Wazuh (Servidor)

## Objetivo de esta fase

Instalar Wazuh all-in-one en el servidor Dell R720 usando la imagen OVA oficial, importándola como VM en Proxmox VE.

---

## ¿Qué es Wazuh?

Wazuh es una plataforma open source de seguridad (XDR/SIEM) que incluye tres componentes principales:

| Componente | Función |
|---|---|
| **Wazuh Manager** | Recibe y procesa alertas de los agentes |
| **Wazuh Indexer** | Almacena y indexa los eventos (basado en OpenSearch) |
| **Wazuh Dashboard** | Interfaz web de visualización y análisis |

En esta guía se instalan los tres componentes en una sola VM (all-in-one) usando la OVA oficial.

---

## Requisitos previos

- Proxmox VE instalado y funcionando
- Acceso a la shell de Proxmox
- Mínimo **8 GB de RAM** disponibles en el servidor
- Mínimo **50 GB de disco** disponibles
- Conexión a internet en el servidor

---

## Especificaciones de la VM

| Parámetro | Valor |
|---|---|
| **VM ID** | 200 |
| **Hostname** | `wazuh` |
| **CPU** | 4 cores |
| **RAM** | 8 GB |
| **Disco** | 50 GB en `vm-storage` (ZFS) |
| **OS** | Amazon Linux 2023 (incluido en la OVA) |
| **IP** | `10.0.0.4/24` |
| **Gateway** | `10.0.0.1` |
| **DNS** | `8.8.8.8` |
| **Versión Wazuh** | 4.14.4 |

---

## Paso 1 — Descargar y extraer la OVA

Desde la shell de Proxmox:

```bash
mkdir /var/lib/vz/wz
cd /var/lib/vz/wz
wget https://packages.wazuh.com/4.x/vm/wazuh-4.14.4.ova
tar -xvf wazuh-4.14.4.ova
```

Esto extrae tres archivos:
- `wazuh-4.14.4.ovf`
- `wazuh-4.14.4-disk-1.vmdk`
- `wazuh-4.14.4.mf`

---

## Paso 2 — Crear la VM en Proxmox

```bash
qm create 200 --name wazuh --memory 8192 --cores 4 --net0 virtio,bridge=vmbr0
```

---

## Paso 3 — Importar el disco

```bash
qm importdisk 200 wazuh-4.14.4-disk-1.vmdk vm-storage
```

> Si el almacenamiento ZFS tiene otro nombre, reemplaza `vm-storage` por el nombre correcto. Verificar con `pvesm status`.

---

## Paso 4 — Configurar y arrancar la VM

```bash
# Asignar el disco importado a la VM
qm set 200 --scsihw virtio-scsi-pci --scsi0 vm-storage:vm-200-disk-0

# Configurar disco de arranque
qm set 200 --boot c --bootdisk scsi0

# Iniciar la VM
qm start 200
```

---

## Paso 5 — Configurar IP estática

En Proxmox → selecciona la VM 200 → **Console**, luego:

1. Inicia sesión con:

```
usuario: wazuh-user
contraseña: wazuh
```

2. Cambia a root:

```bash
sudo -i
```

3. Configura la IP estática:

```bash
nmcli con mod "Wired connection 1" ipv4.addresses 10.0.0.4/24
nmcli con mod "Wired connection 1" ipv4.gateway 10.0.0.1
nmcli con mod "Wired connection 1" ipv4.dns 8.8.8.8
nmcli con mod "Wired connection 1" ipv4.method manual
nmcli con up "Wired connection 1"
```

4. Verifica la IP:

```bash
ip a
```

---

## Paso 6 — Acceder al dashboard de Wazuh

Desde cualquier equipo en la misma red, abre el navegador:

```
https://10.0.0.4
```

Credenciales por defecto:

```
usuario: admin
contraseña: admin
```

> ⚠️ El navegador mostrará advertencia de certificado autofirmado. Haz clic en "Avanzado" → "Continuar de todos modos".

> ⚠️ El dashboard puede tardar 1-2 minutos en inicializar la primera vez.

---

## Paso 7 — Cambiar contraseñas por defecto

Por seguridad, cambia las credenciales inmediatamente después de acceder:

```bash
/usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh --change-all
```

Guarda las nuevas contraseñas en un lugar seguro.

---

## Archivos de configuración importantes

| Componente | Ruta |
|---|---|
| **Wazuh Manager** | `/var/ossec/etc/ossec.conf` |
| **Wazuh Indexer** | `/etc/wazuh-indexer/opensearch.yml` |
| **Filebeat** | `/etc/filebeat/filebeat.yml` |
| **Wazuh Dashboard** | `/etc/wazuh-dashboard/opensearch_dashboards.yml` |

---

## Solución de problemas comunes

| Problema | Solución |
|---|---|
| Dashboard no carga después de 5 minutos | Verificar servicio: `systemctl status wazuh-dashboard` |
| No puedo hacer ping a la VM | Verificar IP con `ip a` y que el bridge `vmbr0` esté activo en Proxmox |
| Error de certificado en el navegador | Normal, es autofirmado. Aceptar la excepción de seguridad |
| El disco importado no aparece | Verificar nombre del storage con `pvesm status` y ajustar el comando del Paso 3 |
| La VM no arranca | Verificar que el bootdisk está correctamente asignado con `qm config 200` |

---

## Registro de instalación

> Completar a medida que se instale.

| Campo | Valor |
|---|---|
| Fecha instalación | |
| Versión Wazuh | 4.14.4 |
| IP asignada | 10.0.0.4 |
| VM ID en Proxmox | 200 |
| Observaciones | |

---

*← [Guía Proxmox](../04-guia-proxmox/README.md) | Siguiente: [Agentes Wazuh →](../06-guia-agentes/README.md)*
