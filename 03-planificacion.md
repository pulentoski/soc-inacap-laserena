# Fase 3 — Planificación

## Arquitectura del SOC

Esta sección define **qué va en cada servidor**, cómo se comunican entre sí y cómo se segmenta la red para que el tráfico de ataque no afecte la red institucional de INACAP.

---

## Distribución de roles por servidor

### Servidor 1 — Dell PowerEdge R720 (SOC Principal)

```
┌──────────────────────────────────────────────┐
│         Proxmox VE — Dell R720               │
│                                              │
│  [LXC] Wazuh Indexer     IP: 192.168.10.11  │
│  [LXC] Wazuh Dashboard   IP: 192.168.10.12  │
│  [LXC] Zabbix Server     IP: 192.168.10.13  │
│  [VM]  Wazuh Manager     IP: 192.168.10.14  │
│                                              │
│  IP de gestión Proxmox:  192.168.10.10       │
└──────────────────────────────────────────────┘
```

> ⚠️ Las IPs son referenciales. Confirmar con Waldo la subred disponible en la sala A315.

---

### Servidor 2 — Dell PowerEdge R420 (Servicios de red)

```
┌──────────────────────────────────────────────┐
│         Proxmox VE — Dell R420               │
│                                              │
│  [VM]  pfSense (firewall)  IP: 192.168.10.1 │
│  [LXC] Grafana             IP: 192.168.10.20│
│                                              │
│  IP de gestión Proxmox:  192.168.10.15       │
└──────────────────────────────────────────────┘
```

---

### Servidor 3 — HP ProLiant DL180 G6 (Laboratorio)

```
┌──────────────────────────────────────────────┐
│    Proxmox VE o Ubuntu Server — HP DL180     │
│                                              │
│  [VM] Ubuntu + Docker      IP: 10.0.0.10    │  ← Red aislada (VLAN Lab)
│       ├─ DVWA              Puerto: 8080      │
│       ├─ Juice Shop        Puerto: 3000      │
│       └─ (otros targets)                    │
│                                              │
│  IP de gestión:  192.168.10.18               │
└──────────────────────────────────────────────┘
```

---

## Segmentación de red

El punto más importante de seguridad en este proyecto: **el tráfico de ataque no debe salir del rack**.

### VLANs planeadas

| VLAN | Nombre | Rango IP | Descripción |
|---|---|---|---|
| VLAN 10 | Gestión | 192.168.10.0/24 | Red de los servidores Proxmox y el SOC |
| VLAN 20 | Laboratorio | 10.0.0.0/24 | Red aislada para las máquinas vulnerables |
| VLAN 1 | Institucional | (red de INACAP) | Red normal de la institución |

### Reglas de firewall

```
VLAN Lab (10.0.0.0/24) → VLAN Gestión (192.168.10.0/24): PERMITIDO solo puerto Wazuh (1514/1515)
VLAN Lab (10.0.0.0/24) → Red INACAP: BLOQUEADO totalmente
VLAN Gestión → Red INACAP: PERMITIDO (para actualizaciones)
Red INACAP → VLAN Lab: BLOQUEADO
```

---

## Diagrama lógico de red

```
Internet / Red INACAP
        │
        │  (VLAN 1 — Institucional)
        │
   [pfSense / Firewall]  ← Servidor 2
        │
        ├──────────────────────────────────┐
        │                                  │
  (VLAN 10 — Gestión)              (VLAN 20 — Lab)
        │                                  │
   ┌────┴─────┐                    ┌───────┴──────┐
   │ Servidor 1│                   │  Servidor 3  │
   │ Dell R720 │                   │  HP DL180    │
   │           │                   │              │
   │ Wazuh ────┼───── Agentes ─────┤ DVWA         │
   │ Zabbix    │      (logs)       │ Juice Shop   │
   └───────────┘                   └──────────────┘
```

---

## Flujo de una alerta en el SOC

```
1. Alumno lanza ataque → Máquina vulnerable (Servidor 3)
           ↓
2. Agente Wazuh detecta el evento (log de intrusión)
           ↓
3. Agente envía el log al Wazuh Manager (Servidor 1) por puerto 1514
           ↓
4. Wazuh Manager procesa y categoriza la alerta
           ↓
5. Se guarda en el Wazuh Indexer
           ↓
6. El alumno ve la alerta en el Wazuh Dashboard desde su laptop
```

---

## Requisitos de red para la sala A315

Antes de instalar, confirmar con **Waldo**:

- [ ] ¿Cuál es la subred disponible para los servidores?
- [ ] ¿Hay un switch gestionable para configurar VLANs?
- [ ] ¿Cuál es el gateway (puerta de enlace) de la red institucional?
- [ ] ¿Cuál es el servidor DNS de INACAP?
- [ ] ¿Hay acceso a internet desde la sala para descargar paquetes?

---

## Tabla de IPs (para completar)

| Dispositivo | Rol | IP Asignada | Puerto gestión |
|---|---|---|---|
| Dell R720 — Proxmox | Hipervisor SOC | 192.168.10.10 | 8006 |
| Dell R720 — Wazuh Indexer | Base de datos logs | 192.168.10.11 | 9200 |
| Dell R720 — Wazuh Dashboard | Interfaz web SOC | 192.168.10.12 | 443 |
| Dell R720 — Zabbix | Monitoreo infra | 192.168.10.13 | 80/443 |
| Dell R420 — Proxmox | Hipervisor red | 192.168.10.15 | 8006 |
| HP DL180 — Proxmox | Hipervisor lab | 192.168.10.18 | 8006 |
| HP DL180 — VM Lab | Máquinas vulnerables | 10.0.0.10 | varios |

> Actualizar esta tabla con las IPs reales una vez confirmadas con Waldo.

---

*← [Software](../02-software/README.md) | Siguiente: [Guía Proxmox →](../04-guia-proxmox/README.md)*
