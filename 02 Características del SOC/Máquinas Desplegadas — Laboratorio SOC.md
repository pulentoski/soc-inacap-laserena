# Máquinas Desplegadas — Laboratorio SOC

## Estado actual de la infraestructura

> Última actualización: Abril 2026

---

## Proxmox 2 — Dell R720 (SOC Principal)

**Host:** proxmox-2 | **IP:** 10.0.0.3 | **Panel:** https://10.0.0.3:8006

| CT ID | Nombre | IP | OS | Servicio | Estado |
|---|---|---|---|---|---|
| 200 | wazuh | 10.0.0.4 | Ubuntu 22.04 | Wazuh SIEM/XDR (all-in-one) | ✅ Running |
| 201 | Suricata | 10.0.0.5 | Ubuntu 22.04 | IDS Suricata | ✅ Running |
| 300 | zabbix-720 | 10.0.0.50 | Ubuntu 22.04 | Zabbix (monitoreo) | ✅ Running |

---

## Proxmox 1 — Dell R420 (Actividades y Lab)

**Host:** prox (pendiente cambio a proxmox-1) | **IP:** 10.0.0.2 | **Panel:** https://10.0.0.2:8006

| CT ID | Nombre | IP | Servicio | Estado | Observaciones |
|---|---|---|---|---|---|
| 101 | ct01-ubuntu-web | 172.16.0.10 | Ubuntu Web | ✅ Running | ⚠️ IP fuera del rango estándar |
| 102 | ct02-alma-interno | 192.168.50.20 | AlmaLinux interno | ✅ Running | ⚠️ IP fuera del rango estándar |
| 200 | zabbix-monitor | 10.0.0.20 | Zabbix (monitor) | ✅ Running | |
| 201 | ct201-ubuntu-web | 10.0.0.30 | Ubuntu Web | ✅ Running | |
| 202 | ct202-debian-dev | 172.16.1.20 | Debian Dev | ✅ Running | ⚠️ IP fuera del rango estándar |
| 203 | ct203-alma-final | 192.168.60.20 | AlmaLinux final | ✅ Running | ⚠️ IP fuera del rango estándar |

> ⚠️ Varios contenedores del R420 tienen IPs fuera del rango estándar definido (10.0.0.50 — 10.0.0.99). Deben ser corregidos según las normas del documento de gestión.

---

## Resumen general

| Servidor | Contenedores activos | Servicios SOC | Estado |
|---|---|---|---|
| R720 (proxmox-2) | 3 | Wazuh, Suricata, Zabbix | ✅ Operativo |
| R420 (proxmox-1) | 6 | Actividades alumnos | ✅ Operativo |
| HP DL180 | 0 | — | ❌ No operativo |

---

## Pendientes

- [ ] Cambiar hostname del R420 de `prox` a `proxmox-1`
- [ ] Corregir IPs de contenedores del R420 al rango estándar (10.0.0.50 — 10.0.0.99)
- [ ] Instalar Zabbix en el R720 con IP correcta (actualmente en 10.0.0.50)
- [ ] Instalar HP DL180

---

*← [Equipamiento](../01-equipamiento/README.md) | [Volver al índice →](../README.md)*
