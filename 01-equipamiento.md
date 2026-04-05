# Fase 1 — Equipamiento

## Inventario de Servidores

Este proyecto utiliza 3 servidores físicos de rack ubicados en la sala **A315** de INACAP La Serena.

---

## Servidor 1 — Dell PowerEdge R720

**Rol en el proyecto:** Cerebro del SOC (Wazuh Indexer + Dashboard + Zabbix)

| Característica | Detalle |
|---|---|
| **Modelo** | Dell PowerEdge R720 |
| **Factor de forma** | 2U Rack |
| **Procesador** | Intel Xeon E5-2600 (2 sockets) |
| **Virtualización CPU** | Intel VT-x / VT-d ✅ |
| **RAM** | Por confirmar (estimado 64–128 GB) |
| **Almacenamiento** | Múltiples discos SAS 3TB + SAS 600GB |
| **Controladora RAID** | Dell PERC H710 / H710P |
| **Tarjetas de red** | 4 puertos Gigabit Ethernet integrados |
| **Gestión remota** | iDRAC 7 |
| **Sistema a instalar** | Proxmox VE |

### Configuración RAID planeada

| Arreglo | Discos | RAID | Uso |
|---|---|---|---|
| Virtual Disk 1 | 2 × 600GB SAS | RAID 1 (espejo) | Sistema operativo (Proxmox) |
| Virtual Disk 2 | Discos 3TB restantes | RAID 5 | Almacenamiento de VMs y logs |

### VMs / Contenedores planeados

- LXC: Wazuh Indexer + Dashboard
- LXC: Zabbix Server
- VM: Ubuntu Server (agente de prueba)

---

## Servidor 2 — Dell PowerEdge R420

**Rol en el proyecto:** Servicios de red y apoyo

| Característica | Detalle |
|---|---|
| **Modelo** | Dell PowerEdge R420 |
| **Factor de forma** | 1U Rack |
| **Procesador** | Intel Xeon E5-2400 (1–2 sockets) |
| **Virtualización CPU** | Intel VT-x ✅ |
| **RAM** | Por confirmar |
| **Almacenamiento** | Por confirmar |
| **Controladora RAID** | Dell PERC H310 / H710 |
| **Tarjetas de red** | 4 puertos Gigabit Ethernet |
| **Gestión remota** | iDRAC 7 |
| **Sistema a instalar** | Proxmox VE (unido al clúster del R720) |

### VMs / Contenedores planeados

- VM: Wazuh Manager
- LXC: Grafana (visualización adicional)
- VM: pfSense (firewall virtual — opcional)

---

## Servidor 3 — HP ProLiant DL180 G6

**Rol en el proyecto:** Laboratorio de máquinas vulnerables

| Característica | Detalle |
|---|---|
| **Modelo** | HP ProLiant DL180 G6 |
| **Factor de forma** | 2U Rack |
| **Procesador** | Intel Xeon X5500 / L5500 |
| **Virtualización CPU** | Intel VT-x ✅ (verificar en BIOS) |
| **RAM** | Por confirmar |
| **Almacenamiento** | Múltiples discos 1TB |
| **Controladora RAID** | HP Smart Array P410 |
| **Tarjetas de red** | 2–4 puertos Gigabit Ethernet |
| **Gestión remota** | HP iLO 2 |
| **Sistema a instalar** | Proxmox VE o Ubuntu Server |

### VMs / Contenedores planeados

- VM: Ubuntu Server con Docker
  - Contenedor DVWA
  - Contenedor Juice Shop
  - Contenedor Metasploitable (o VM separada)
- Todos con agente Wazuh instalado

---

## Equipamiento de red y gestión

| Equipo | Modelo | Función |
|---|---|---|
| **Switch KVM** | Tripp-Lite 8 puertos | Gestión de los 3 servidores con un solo teclado/monitor |
| **Switch de red** | Por confirmar | Conectividad entre servidores y red institucional |

---

## Diagrama físico del rack

```
┌─────────────────────────────┐
│  Switch KVM Tripp-Lite 8P   │  ← Gestión de consola
├─────────────────────────────┤
│   Dell PowerEdge R720       │  ← Servidor 1 (SOC principal)
│   [Wazuh Indexer + Zabbix]  │
├─────────────────────────────┤
│   Dell PowerEdge R420       │  ← Servidor 2 (Red y servicios)
│   [Wazuh Manager]           │
├─────────────────────────────┤
│   HP ProLiant DL180 G6      │  ← Servidor 3 (Lab. vulnerable)
│   [Docker + DVWA + Juice]   │
└─────────────────────────────┘
```

---

## Checklist pre-instalación

Antes de instalar cualquier software, verificar lo siguiente en cada servidor:

- [ ] Encender servidor y entrar a BIOS (F2 en Dell, F9 en HP)
- [ ] Verificar que Intel VT-x está **habilitado**
- [ ] Ingresar a controladora RAID (Ctrl+R en Dell, F8 en HP) y confirmar estado de discos
- [ ] Limpiar filtros de polvo (especialmente el HP DL180)
- [ ] Verificar conectividad KVM con los 3 servidores
- [ ] Registrar la IP que asignará Waldo para cada servidor
- [ ] Confirmar especificaciones reales (RAM, discos) al encender

---

*← [Investigación](../00-investigacion/README.md) | Siguiente: [Software →](../02-software/README.md)*
