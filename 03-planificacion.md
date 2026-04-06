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
│  [LXC] Wazuh All-in-one  IP: por asignar  │
│         ├─ Wazuh Manager                    │
│         ├─ Wazuh Indexer                    │
│         └─ Wazuh Dashboard                  │
│                                              │
│  [LXC] Zabbix Server     IP: por asignar  │
│                                              │
│  IP de gestión Proxmox:  IP_PROXMOX_R720       │
└──────────────────────────────────────────────┘
```

> ⚠️ Las IPs son referenciales. Confirmar con encargado de sala la subred disponible en la sala de servidores.

**¿Por qué All-in-one?**

Wazuh puede instalarse de dos formas:

| Modo | Descripción | Cuándo usarlo |
|---|---|---|
| **All-in-one** | Manager + Indexer + Dashboard en un solo contenedor | Entornos académicos, laboratorios, hasta ~100 agentes |
| **Distribuido** | Cada componente en su propio servidor con su propia IP | Entornos empresariales con miles de agentes |

Para este proyecto se usa **All-in-one** porque simplifica la instalación y es suficiente para el volumen de agentes del laboratorio. Si en el futuro el SOC crece, se puede migrar a la instalación distribuida.

---

### Servidor 2 — Dell PowerEdge R420 (Servicios de red)

```
┌──────────────────────────────────────────────┐
│         Proxmox VE — Dell R420               │
│                                              │
│  [VM]  pfSense (firewall)  IP: por asignar │
│  [LXC] Grafana             IP: por asignar│
│                                              │
│  IP de gestión Proxmox:  IP_PROXMOX_R420       │
└──────────────────────────────────────────────┘
```

---

### Servidor 3 — HP ProLiant DL180 G6 (Laboratorio)

```
┌──────────────────────────────────────────────┐
│    Proxmox VE o Ubuntu Server — HP DL180     │
│                                              │
│  [VM] Ubuntu + Docker      IP: por asignar    │  ← Red aislada (VLAN Lab)
│       ├─ DVWA              Puerto: 8080      │
│       ├─ Juice Shop        Puerto: 3000      │
│       └─ (otros targets)                    │
│                                              │
│  IP de gestión:  IP_PROXMOX_DL180               │
└──────────────────────────────────────────────┘
```

---

## Segmentación de red

El punto más importante de seguridad en este proyecto: **el tráfico de ataque no debe salir del rack**.

### VLANs planeadas

| VLAN | Nombre | Rango IP | Descripción |
|---|---|---|---|
| VLAN 10 | Gestión | SUBRED_GESTION/24 | Red de los servidores Proxmox y el SOC |
| VLAN 20 | Laboratorio | SUBRED_LAB/24 | Red aislada para las máquinas vulnerables |
| VLAN 1 | Institucional | (red de INACAP) | Red normal de la institución |

### Reglas de firewall

```
VLAN Lab (SUBRED_LAB/24) → VLAN Gestión (SUBRED_GESTION/24): PERMITIDO solo puerto Wazuh (1514/1515)
VLAN Lab (SUBRED_LAB/24) → Red INACAP: BLOQUEADO totalmente
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

## Requisitos de red para la sala de servidores

Antes de instalar, confirmar con **encargado de sala**:

- [ ] ¿Cuál es la subred disponible para los servidores?
- [ ] ¿Hay un switch gestionable para configurar VLANs?
- [ ] ¿Cuál es el gateway (puerta de enlace) de la red institucional?
- [ ] ¿Cuál es el servidor DNS de INACAP?
- [ ] ¿Hay acceso a internet desde la sala para descargar paquetes?

---

## Tabla de IPs (para completar)

| Dispositivo | Rol | IP Asignada | Puerto gestión |
|---|---|---|---|
| Dell R720 — Proxmox | Hipervisor SOC | IP_PROXMOX_R720 | 8006 |
| Dell R720 — Wazuh (all-in-one) | Manager + Indexer + Dashboard | IP_WAZUH | 443 |
| Dell R720 — Zabbix | Monitoreo infra | IP_ZABBIX | 80/443 |
| Dell R420 — Proxmox | Hipervisor red | IP_PROXMOX_R420 | 8006 |
| HP DL180 — Proxmox | Hipervisor lab | IP_PROXMOX_DL180 | 8006 |
| HP DL180 — VM Lab | Máquinas vulnerables | IP_LAB | varios |

> Actualizar esta tabla con las IPs reales una vez confirmadas con encargado de sala.

---

*← [Software](../02-software/README.md) | Siguiente: [Guía Proxmox →](../04-guia-proxmox/README.md)*
