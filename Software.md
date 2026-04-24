# Fase 2 — Software

## Stack tecnológico del proyecto

Este proyecto utiliza exclusivamente software **open source y gratuito**. A continuación se describe cada herramienta, su versión recomendada, dónde descargarla y por qué fue elegida.

---

## 1. Proxmox VE — Hipervisor

| | |
|---|---|
| **Versión recomendada** | Proxmox VE 8.x |
| **Descarga** | https://www.proxmox.com/en/downloads |
| **Formato** | ISO (para grabar en USB o montar en VirtualBox) |
| **Licencia** | AGPLv3 (gratuito) |

### ¿Por qué Proxmox?

- Se instala directo sobre el hardware (Hipervisor Tipo 1)
- Permite crear VMs (KVM) y Contenedores (LXC) desde una interfaz web
- Soporta clúster: los 3 servidores pueden trabajar juntos
- Incluye ZFS para snapshots de máquinas virtuales
- Es el estándar en muchas empresas y data centers reales

### Requisitos mínimos

| Recurso | Mínimo | Recomendado |
|---|---|---|
| CPU | 64-bit con VT-x | Intel Xeon (nuestros servidores ✅) |
| RAM | 2 GB | 8 GB o más |
| Disco | 20 GB | 60 GB o más |
| Red | 1 NIC | 2 NIC (gestión + datos) |

---

## 2. Wazuh — SIEM / XDR

| | |
|---|---|
| **Versión recomendada** | Wazuh 4.14.x |
| **Descarga** | https://documentation.wazuh.com/current/installation-guide/ |
| **Formato** | Script de instalación (bash) |
| **Licencia** | GPLv2 (gratuito) |

### Componentes a instalar

| Componente | Servidor | Método |
|---|---|---|
| Wazuh Indexer | Dell R720 | LXC Ubuntu 22.04 |
| Wazuh Manager | Dell R720 o R420 | LXC Ubuntu 22.04 |
| Wazuh Dashboard | Dell R720 | LXC Ubuntu 22.04 |
| Wazuh Agent | Todos los equipos monitorizados | Instalación por paquete |

### Comando de instalación rápida (All-in-one)

> ⚠️ Este comando instala Indexer + Manager + Dashboard en un solo paso.
> Ejecutar dentro del contenedor LXC Ubuntu 22.04.

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash wazuh-install.sh -a
```

### Requisitos mínimos para Wazuh

| Recurso | Mínimo | Recomendado |
|---|---|---|
| CPU | 2 núcleos | 4 núcleos |
| RAM | 4 GB | 8 GB |
| Disco | 50 GB | 100 GB (para logs) |

---

## 3. Zabbix — Monitoreo de infraestructura

| | |
|---|---|
| **Versión recomendada** | Zabbix 6.4 LTS o 7.0 LTS |
| **Descarga** | https://www.zabbix.com/download |
| **Formato** | Repositorio apt (instalación por paquetes) |
| **Licencia** | GPLv2 (gratuito) |

### Componentes a instalar

| Componente | Servidor | Método |
|---|---|---|
| Zabbix Server | Dell R720 | LXC Ubuntu 22.04 |
| Zabbix Frontend | Dell R720 | Mismo LXC (con Apache/Nginx) |
| Zabbix Agent | Todos los servidores | Instalación por paquete |

### Requisitos mínimos para Zabbix

| Recurso | Mínimo | Recomendado |
|---|---|---|
| CPU | 2 núcleos | 2 núcleos |
| RAM | 512 MB | 2 GB |
| Disco | 10 GB | 50 GB |

---

## 4. Docker — Laboratorio de aplicaciones vulnerables

| | |
|---|---|
| **Versión recomendada** | Docker CE 24.x o superior |
| **Descarga** | https://docs.docker.com/engine/install/ubuntu/ |
| **Formato** | Repositorio apt |
| **Licencia** | Apache 2.0 (gratuito) |

### Se instalará sobre

- **HP ProLiant DL180 G6** — dentro de una VM Ubuntu Server o directamente en Ubuntu Server

### Contenedores vulnerables a levantar

```yaml
# docker-compose.yml (referencial)
services:
  dvwa:
    image: vulnerables/web-dvwa
    ports:
      - "8080:80"

  juiceshop:
    image: bkimminich/juice-shop
    ports:
      - "3000:3000"
```

---

## 5. VirtualBox — Práctica previa (laptop del docente)

| | |
|---|---|
| **Versión recomendada** | VirtualBox 7.x |
| **Descarga** | https://www.virtualbox.org/wiki/Downloads |
| **Formato** | Instalador (.exe para Windows) |
| **Licencia** | GPLv2 (gratuito) |

### ¿Para qué se usa aquí?

Antes de tocar los servidores reales, el docente y los alumnos pueden **practicar la instalación de Proxmox** en VirtualBox sobre su propia laptop. Esto permite:

- Aprender la interfaz de Proxmox sin riesgo
- Probar la creación de VMs y contenedores
- Cometer errores sin consecuencias

> Ver guía completa en: [04-guia-proxmox](../04-guia-proxmox/README.md)

---

## Resumen del stack

```
┌─────────────────────────────────────────────────────────┐
│                    CAPA DE HARDWARE                     │
│         Dell R720 │ Dell R420 │ HP DL180 G6             │
├─────────────────────────────────────────────────────────┤
│                  CAPA DE HIPERVISOR                     │
│                    Proxmox VE 8.x                       │
├─────────────────────────────────────────────────────────┤
│                  CAPA DE SERVICIOS                      │
│   Wazuh (SIEM)  │  Zabbix (Monitoreo)  │  Docker (Lab) │
├─────────────────────────────────────────────────────────┤
│                 CAPA DE VISUALIZACIÓN                   │
│    Wazuh Dashboard  │  Zabbix Frontend  │  Grafana      │
└─────────────────────────────────────────────────────────┘
```

---

## ISOs y recursos a descargar

- [ ] Proxmox VE 8.x ISO → https://www.proxmox.com/en/downloads
- [ ] Ubuntu Server 22.04 LTS ISO → https://ubuntu.com/download/server
- [ ] VirtualBox 7.x → https://www.virtualbox.org/wiki/Downloads
- [ ] Kali Linux ISO (para pruebas de ataque) → https://www.kali.org/get-kali/

---

*← [Equipamiento](../01-equipamiento/README.md) | Siguiente: [Planificación →](../03-planificacion/README.md)*
