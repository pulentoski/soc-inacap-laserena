# Gestión y Uso de Servidores — Laboratorio SOC

## Descripción

Este documento establece las normas de uso, gestión y estándares técnicos para los servidores del laboratorio SOC. Su objetivo es mantener la infraestructura ordenada, estable y segura para el desarrollo de las actividades académicas.

---

## Infraestructura disponible

| Servidor | Hostname | IP | Panel Web | Rol |
|---|---|---|---|---|
| Dell PowerEdge R420 | proxmox-1 | 10.0.0.2 | https://10.0.0.2:8006 | Zabbix + Contenedores alumnos |
| Dell PowerEdge R720 | proxmox-2 | 10.0.0.3 | https://10.0.0.3:8006 | Wazuh (SIEM/XDR) |
| Wazuh (LXC en R720) | wazuh | 10.0.0.4 | https://10.0.0.4 | SIEM/XDR |
| Zabbix (LXC en R420) | zabbix | 10.0.0.5 | https://10.0.0.5 | Monitoreo |

---

## Roles y permisos

| Rol | Acceso | Permisos |
|---|---|---|
| **Docente** | Proxmox completo (root) | Crear, modificar y eliminar cualquier recurso |
| **Alumno** | Solo a sus contenedores asignados | Usar y configurar dentro de su contenedor |

> ⚠️ Los alumnos **no tienen acceso root** a los servidores Proxmox directamente. Solo el docente administra la infraestructura base.

---

## IPs asignadas y disponibles

| IP | Asignado a | Servidor | Estado |
|---|---|---|---|
| 10.0.0.1 | Gateway (Router) | — | Ocupada |
| 10.0.0.2 | Proxmox 1 (R420) | R420 | Ocupada |
| 10.0.0.3 | Proxmox 2 (R720) | R720 | Ocupada |
| 10.0.0.4 | Wazuh | R720 | Ocupada |
| 10.0.0.5 | Zabbix | R420 | Ocupada |
| 10.0.0.6 - 10.0.0.49 | Servicios SOC | R720 | Disponibles |
| 10.0.0.50 - 10.0.0.99 | Contenedores alumnos | R420 | Disponibles |
| 10.0.0.100 - 10.0.0.199 | Máquinas vulnerables | R420 | Reservadas |
| 10.0.0.200 - 10.0.0.254 | Reservado | — | No usar |

---

## Estándares para creación de contenedores

Todo contenedor creado en el laboratorio **debe cumplir** con los siguientes parámetros:

### Recursos base por contenedor

| Recurso | Valor base |
|---|---|
| **CPU** | 1 core |
| **RAM** | 1024 MB |
| **Swap** | 512 MB |
| **Disco** | 15 GB |

> Si una actividad requiere más recursos, debe ser solicitado y autorizado por el docente.

### Red

- Usar siempre el bridge **vmbr0**
- Asignar IP estática del rango **10.0.0.50 — 10.0.0.99**
- Gateway: **10.0.0.1**
- DNS: **8.8.8.8**
- **No crear bridges adicionales** sin autorización del docente
- **No usar subredes distintas** a 10.0.0.0/24 sin autorización

### Nomenclatura

| Campo | Formato | Ejemplo |
|---|---|---|
| **Hostname** | apellido-servicio | `castro-ubuntu` |
| **CT ID** | Rango 300-399 | `301` |

### Sistema operativo

- Usar preferentemente **Ubuntu 22.04 LTS**
- Si se requiere otro OS, debe ser aprobado por el docente

### Servidor donde crear contenedores

- Todos los contenedores de alumnos se crean en **Proxmox 1 (R420)** — `10.0.0.2`
- El R720 está reservado exclusivamente para servicios SOC (Wazuh)

---

## Actividades permitidas

- Instalación y configuración de servicios en contenedores propios
- Práctica de administración de sistemas Linux
- Instalación de agentes Wazuh en contenedores
- Escaneo y análisis de máquinas vulnerables del R420
- Captura y análisis de tráfico dentro de la red del laboratorio

---

## Actividades prohibidas

- ❌ Ataques a sistemas **fuera** de la red del laboratorio (10.0.0.0/24)
- ❌ Minería de criptomonedas
- ❌ Eliminar o modificar contenedores de otros alumnos
- ❌ Intentar acceder a los servidores Proxmox con credenciales de root
- ❌ Crear contenedores fuera de los estándares definidos en este documento
- ❌ Usar IPs fuera del rango asignado a alumnos (10.0.0.50 — 10.0.0.99)
- ❌ Instalar herramientas de ataque fuera de los contenedores del R420
- ❌ Compartir credenciales de acceso

> ⚠️ El incumplimiento de estas normas puede resultar en la suspensión del acceso al laboratorio.

---

## Responsabilidades

| Responsable | Tarea |
|---|---|
| **Docente** | Administrar servidores, crear contenedores base, autorizar recursos extra |
| **Alumno** | Documentar todo lo que realice en su contenedor en el repositorio git |

---

## Documentación obligatoria

Cada actividad realizada en el laboratorio debe ser documentada por el alumno en el repositorio git del curso, siguiendo el formato README.md establecido. Debe incluir:

- Descripción de la actividad
- Comandos ejecutados
- Problemas encontrados y soluciones
- Estado final

---

*← [Guía Wazuh](../05-guia-wazuh/README.md) | [Volver al índice →](../README.md)*
