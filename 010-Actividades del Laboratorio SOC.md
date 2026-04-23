# Actividades del Laboratorio SOC — INACAP La Serena

## Descripción

Este documento describe las actividades académicas que pueden realizarse en el laboratorio SOC de la sala A315, orientadas a las carreras de Ciberseguridad, Telecomunicaciones, Informática y Programación.

El laboratorio está diseñado para actividades de **gestión, monitoreo y ciberseguridad**, apoyando el desarrollo de competencias prácticas en entornos reales de infraestructura.

---

## Infraestructura disponible para actividades

| Recurso | Uso académico |
|---|---|
| Proxmox 1 (R420) | Contenedores de alumnos, máquinas vulnerables, CTF |
| Proxmox 2 (R720) | Wazuh SIEM — monitoreo y análisis de alertas |
| Wazuh | Gestión de agentes, análisis de logs, detección de amenazas |
| Zabbix | Monitoreo de red, switches y routers del laboratorio |

---

## Carreras y modalidad de trabajo

| Carrera | Modalidad recomendada | Observaciones |
|---|---|---|
| **Ciberseguridad** | Individual o grupal | Carrera principal del laboratorio |
| **Telecomunicaciones** | Individual | Pocos alumnos, pueden trabajar sin restricciones |
| **Informática** | Grupal | Muchas secciones, trabajo por grupos |
| **Programación** | Grupal | Muchas secciones, trabajo por grupos |

> ℹ️ Para carreras con muchas secciones se recomienda crear **un contenedor por grupo** en lugar de uno por alumno, para no saturar los recursos del servidor.

---

## Actividades disponibles por área

### 1. Monitoreo y gestión (Zabbix)

**Carreras:** Telecomunicaciones, Informática, Ciberseguridad

- Monitoreo de switches y routers del laboratorio A315
- Visualización de métricas de red en tiempo real
- Configuración de alertas por umbral (CPU, RAM, tráfico)
- Análisis de disponibilidad de servicios
- Creación de dashboards de monitoreo

---

### 2. Ciberseguridad ofensiva — Pentesting (R420)

**Carreras:** Ciberseguridad

- Escaneo de vulnerabilidades con Nmap, Nessus o similares
- Explotación de máquinas vulnerables creadas por el docente
- Análisis de vulnerabilidades conocidas (CVE)
- Práctica con frameworks de explotación (Metasploit)

> ⚠️ Las máquinas vulnerables son creadas y administradas exclusivamente por el docente. Los alumnos solo realizan actividades sobre los targets autorizados.

---

### 3. Ciberseguridad defensiva — SIEM (Wazuh)

**Carreras:** Ciberseguridad, Telecomunicaciones

- Instalación de agentes Wazuh en contenedores propios
- Análisis de logs y alertas generadas
- Correlación de eventos de seguridad
- Detección de actividad maliciosa en tiempo real
- Generación de reportes de seguridad
- Respuesta a incidentes simulados

---

### 4. CTF — Capture The Flag

**Carreras:** Ciberseguridad, Informática, Programación

- Plataforma CTF desarrollada y administrada por alumnos del proyecto SOC
- Desafíos de categorías: web, forense, criptografía, reversing, redes
- Actividades competitivas entre secciones o carreras
- Disponible para actividades extracurriculares y eventos institucionales

> ℹ️ El CTF es desarrollado por los alumnos a cargo del proyecto SOC y puede ser utilizado por cualquier carrera como actividad práctica.

---

### 5. Administración de sistemas Linux (Contenedores)

**Carreras:** Informática, Programación, Telecomunicaciones

- Creación y gestión de contenedores LXC
- Instalación y configuración de servicios (web, base de datos, DNS, etc.)
- Gestión de usuarios y permisos
- Configuración de red y firewall
- Scripting y automatización con Bash

> ℹ️ Para ramos de Sistemas Operativos con muchas secciones se recomienda trabajo grupal — un contenedor por grupo de 3-4 alumnos.

---

### 6. Redes y conectividad (Telecomunicaciones)

**Carreras:** Telecomunicaciones, Informática

- Análisis de tráfico de red con Wireshark o tcpdump
- Configuración de VLANs y bridges en Proxmox
- Monitoreo de dispositivos de red con Zabbix
- Análisis de protocolos de red en entorno controlado

---

## Actividades NO disponibles en este laboratorio

- ❌ Ramos de Sistemas Operativos con trabajo **individual** (capacidad insuficiente para todas las secciones)
- ❌ Actividades que requieran acceso a internet sin restricciones
- ❌ Ataques fuera de la red 10.0.0.0/24
- ❌ Actividades no relacionadas con las áreas de gestión, monitoreo o ciberseguridad

---

## Recomendaciones para el docente

- Crear las máquinas vulnerables con anticipación antes de la clase
- Asignar rangos de IP específicos por sección para evitar conflictos
- Para actividades masivas, crear los contenedores base antes de la clase y que los alumnos solo los configuren
- Coordinar con el encargado del proyecto SOC para actividades CTF
- Documentar cada actividad en el repositorio git del curso

---

## Sala A315 — Consideraciones

- El laboratorio SOC está ubicado en la **sala A315**, única sala con acceso físico a los servidores
- El acceso al panel web de Proxmox está disponible desde cualquier equipo conectado a la red del laboratorio
- Los servidores deben permanecer encendidos durante las actividades
- Ante cualquier problema con los servidores, contactar al docente encargado del proyecto SOC

---

*← [Gestión de Servidores](../gestion-servidores/README.md) | [Volver al índice →](../README.md)*
