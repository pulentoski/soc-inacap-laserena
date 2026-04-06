# 🛡️ Proyecto SOC Académico — INACAP La Serena

| | |
|---|---|
| 🏫 **Institución** | INACAP — Sede La Serena |
| 📚 **Asignatura** | Virtualización de Data Center |
| 🎓 **Carrera** | Técnico en Ciberseguridad / Infraestructura TI |
| 👨‍🏫 **Docente** | Por confirmar |
| 🏢 **Sala** | Por confirmar |
| 📅 **Año** | 2026 |

---

## 📌 Descripción del proyecto

Este repositorio documenta la construcción completa de un **SOC (Security Operations Center) académico** desplegado sobre hardware real de servidores de rack. El proyecto integra los contenidos del electivo de **Virtualización de Data Center** con competencias prácticas de ciberseguridad, permitiendo a los estudiantes operar una infraestructura de monitoreo y detección de amenazas equivalente a la que encontrarán en el mundo laboral.

El SOC está construido íntegramente con herramientas **open source y gratuitas**, lo que lo hace replicable en cualquier institución educativa. Toda la documentación está estructurada para que un alumno pueda:

- 🔍 Entender el propósito y la arquitectura del sistema desde cero.
- ⚙️ Reproducir la instalación completa siguiendo esta guía.
- 📊 Operar el SOC: monitorizar, detectar ataques y generar reportes.
- 🔧 Mantener o reparar el sistema ante una falla, partiendo desde este documento.

El método de planificación utilizado es **Top-Down**: se parte desde la visión general del sistema (¿qué queremos construir y para qué?) y se desciende progresivamente hacia los detalles técnicos de instalación y configuración.

---

## 🏛️ Contexto institucional

INACAP La Serena forma técnicos y profesionales en el área de tecnologías de la información. Una de las competencias más demandadas en el mercado laboral actual es la capacidad de operar infraestructura de seguridad: detectar intrusiones, monitorizar redes y responder ante incidentes.

Este proyecto nace de la necesidad de trasladar esas competencias del plano teórico al plano práctico. En lugar de simular un entorno de seguridad en software de escritorio, los alumnos trabajan sobre **servidores físicos de rack reales**, configuran **redes virtuales**, despliegan **plataformas SIEM profesionales** y ejecutan **ataques controlados** para observar cómo el sistema los detecta en tiempo real.

---

## 🎯 Objetivos del proyecto

### Objetivo general

Construir e implementar un SOC funcional sobre infraestructura virtualizada, utilizando herramientas open source de nivel empresarial, que permita a los estudiantes adquirir competencias reales en monitoreo de seguridad, detección de amenazas y administración de sistemas.

### Objetivos específicos

1. ⚙️ Instalar y configurar Proxmox VE como hipervisor de Tipo 1 en servidores físicos de rack.
2. 🛡️ Desplegar Wazuh como plataforma SIEM/XDR para la recolección y análisis de eventos de seguridad.
3. 📈 Implementar Zabbix para el monitoreo de salud de la infraestructura (CPU, RAM, discos, servicios).
4. 🐳 Crear un laboratorio de máquinas vulnerables usando Docker para practicar técnicas de ataque y defensa.
5. 🌐 Segmentar la red mediante VLANs para aislar el tráfico de laboratorio de la red institucional.
6. ✅ Validar el sistema mediante pruebas de ataque controladas y verificar la detección en tiempo real.

---

## 🎓 Actividades pedagógicas

Este proyecto permite desarrollar las siguientes actividades con los alumnos:

### 🧠 Actividades de comprensión

- Analizar la arquitectura de un SOC real y mapearla sobre la infraestructura del proyecto.
- Identificar los componentes de Wazuh (Indexer, Manager, Dashboard, Agent) y explicar el rol de cada uno.
- Comparar la virtualización mediante KVM versus contenedores LXC, argumentando cuándo usar cada tecnología.
- Interpretar alertas de seguridad en el Dashboard de Wazuh: nivel de severidad, regla activada, origen del evento.

### 🔧 Actividades de instalación y configuración

- Instalar Proxmox VE en VirtualBox (práctica en laptop antes de tocar los servidores reales).
- Crear máquinas virtuales y contenedores LXC dentro de Proxmox, asignando recursos de CPU, RAM y almacenamiento.
- Desplegar Wazuh mediante el script de instalación automatizada y verificar que todos los componentes funcionen.
- Instalar el agente Wazuh en diferentes sistemas operativos y verificar la conexión al Manager.
- Levantar aplicaciones vulnerables con Docker Compose y verificar su accesibilidad.

### ⚔️ Actividades de ataque y defensa (Red Team / Blue Team)

- Ejecutar un escaneo de puertos con Nmap contra las máquinas del laboratorio y observar cómo Wazuh lo detecta.
- Simular un ataque de fuerza bruta contra SSH y analizar la alerta generada en el SIEM.
- Realizar un ataque SQL Injection básico contra DVWA y verificar el registro del evento.
- Actuar como Blue Team: interpretar las alertas, determinar el origen del ataque y proponer medidas de mitigación.

### 🛠️ Actividades de administración y mantención

- Crear un snapshot de una VM antes de una prueba destructiva y restaurarla tras el experimento.
- Monitorizar el consumo de recursos de los servidores en Zabbix e identificar cuellos de botella.
- Diagnosticar un agente Wazuh que dejó de reportar y ejecutar los pasos para reconectarlo.
- Documentar un incidente de seguridad siguiendo un formato estándar de reporte.

---

## 🧰 Herramientas del proyecto

### ⚙️ Proxmox VE

Proxmox VE (Virtual Environment) es un hipervisor de Tipo 1 (bare-metal), lo que significa que se instala directamente sobre el hardware del servidor, sin necesitar un sistema operativo previo. Su función es crear y gestionar máquinas virtuales y contenedores desde una interfaz web.

**Características principales:**
- Soporta dos tecnologías de virtualización: KVM para máquinas virtuales completas y LXC para contenedores ligeros de Linux.
- Incluye gestión de almacenamiento con soporte para ZFS, lo que permite crear snapshots instantáneos.
- Permite formar clústeres: varios servidores Proxmox pueden gestionarse desde una sola interfaz.
- Es open source y gratuito para uso académico y empresarial.

**KVM (Kernel-based Virtual Machine):** tecnología que crea máquinas virtuales completas con hardware emulado. Cada VM tiene su propio kernel, lo que garantiza aislamiento total. Se usa para sistemas Windows, firewalls virtuales o cualquier sistema que requiera emulación de hardware completo.

**LXC (Linux Containers):** tecnología de contenedores que comparte el kernel del servidor anfitrión con el sistema virtualizado. Consume significativamente menos RAM y CPU que una VM, arranca en segundos y es ideal para servicios Linux como Wazuh o Zabbix. No puede virtualizar Windows.

---

### 🛡️ Wazuh

Wazuh es una plataforma open source de seguridad que funciona como SIEM (Security Information and Event Management) y XDR (Extended Detection and Response). Es la herramienta central del SOC: recolecta, analiza y correlaciona eventos de seguridad de todos los dispositivos monitorizados.

**Componentes:**

| Componente | Función |
|---|---|
| 🤖 **Wazuh Agent** | Programa instalado en cada máquina monitorizada. Recolecta logs del sistema, detecta cambios en archivos, monitoriza procesos y envía todo al Manager. |
| 🧠 **Wazuh Manager** | Cerebro del sistema. Recibe los eventos de los agentes, aplica reglas de detección y genera alertas cuando identifica comportamiento malicioso. |
| 🗄️ **Wazuh Indexer** | Base de datos basada en OpenSearch donde se almacenan y indexan todos los eventos para permitir búsquedas rápidas. |
| 📊 **Wazuh Dashboard** | Interfaz web de visualización. Muestra alertas en tiempo real, estadísticas, mapas de amenazas y permite investigar incidentes. |

**Capacidades de detección:**
- 🔍 Detección de intrusiones basada en reglas (más de 3.000 reglas predefinidas).
- 📁 Monitoreo de integridad de archivos: alerta si un archivo crítico del sistema es modificado.
- 🐛 Detección de vulnerabilidades: cruza el software instalado contra bases de datos de CVEs conocidos.
- 📋 Cumplimiento normativo: mapea eventos con estándares como PCI-DSS, HIPAA, GDPR.

---

### 📈 Zabbix

Zabbix es una plataforma open source de monitoreo de infraestructura. A diferencia de Wazuh, que se enfoca en seguridad, Zabbix monitorea la salud operativa de los sistemas: uso de CPU, memoria, espacio en disco, disponibilidad de servicios y rendimiento de red.

**Características principales:**
- Monitoreo sin agente (vía SNMP, ICMP) o con agente instalado en el sistema.
- Sistema de alertas y disparadores (triggers) configurables: por ejemplo, alertar si la CPU supera el 90% por más de 5 minutos.
- Dashboards y gráficos históricos del rendimiento de los equipos.
- Descubrimiento automático de dispositivos en la red.

**Diferencia con Wazuh:** Zabbix detecta que un servidor está sobrecargado o que un servicio cayó. Wazuh detecta que alguien intentó acceder sin autorización o que un archivo fue modificado de forma sospechosa. En un SOC real, ambas herramientas trabajan en paralelo.

---

### 🐳 Docker

Docker es una plataforma de contenedores de aplicaciones. Permite empaquetar una aplicación junto con todas sus dependencias en una unidad portable llamada contenedor, que puede ejecutarse de forma idéntica en cualquier sistema.

En este proyecto, Docker se utiliza en el servidor de laboratorio para levantar aplicaciones web intencionalmente vulnerables que los alumnos atacarán como parte de los ejercicios prácticos.

**Aplicaciones vulnerables utilizadas:**

| Aplicación | Descripción | Puerto |
|---|---|---|
| 🎯 **DVWA** | Aplicación PHP con vulnerabilidades web clásicas: SQL Injection, XSS, File Upload, etc. | 8080 |
| 🛒 **OWASP Juice Shop** | Tienda online vulnerable moderna. Cubre el Top 10 de OWASP con más de 100 desafíos. | 3000 |
| 🐐 **WebGoat** | Plataforma de aprendizaje de seguridad web de OWASP con lecciones interactivas. | 8888 |

---

### 💻 VirtualBox

VirtualBox es un hipervisor de Tipo 2 (hosted): se instala sobre un sistema operativo existente (como Windows o Linux) y permite crear máquinas virtuales dentro de él. En este proyecto se usa exclusivamente como **entorno de práctica previo**, para que los alumnos instalen y conozcan Proxmox VE en su propia laptop antes de configurar los servidores reales del rack.

---

## 🗺️ Arquitectura del sistema

```
                        RED INSTITUCIONAL INACAP
                                  │
                         [pfSense / Firewall]
                                  │
                ┌─────────────────┼─────────────────┐
                │                                   │
         VLAN Gestión                        VLAN Laboratorio
                │                                   │
    ┌───────────┴───────────┐          ┌────────────┴────────────┐
    │   Dell PowerEdge R720  │          │   HP ProLiant DL180 G6  │
    │   (SOC Principal)      │          │   (Laboratorio)         │
    │                        │          │                         │
    │  [LXC] Wazuh ──────────┼──agentes─┤  [VM] Ubuntu + Docker   │
    │  [LXC] Zabbix          │          │       DVWA              │
    │                        │          │       Juice Shop        │
    └────────────────────────┘          └─────────────────────────┘
    │   Dell PowerEdge R420  │
    │   (Red y servicios)    │
    │  [VM]  pfSense         │
    │  [LXC] Grafana         │
    └────────────────────────┘
```

> 🔒 El tráfico de ataque generado en la VLAN de laboratorio **nunca sale hacia la red institucional**. Solo se permite el paso de los logs del agente Wazuh hacia el SOC principal, por el puerto 1514.

---

## 🖥️ Hardware del proyecto

| Servidor | Modelo | Factor | Rol |
|---|---|---|---|
| Servidor 1 | Dell PowerEdge R720 | 2U Rack | SOC principal: Wazuh + Zabbix |
| Servidor 2 | Dell PowerEdge R420 | 1U Rack | Red y servicios de apoyo |
| Servidor 3 | HP ProLiant DL180 G6 | 2U Rack | Laboratorio de máquinas vulnerables |

Los tres servidores tienen instalado **Proxmox VE** como hipervisor. Los servicios corren como máquinas virtuales o contenedores dentro de cada uno.

---

## 📂 Estructura del repositorio

```
soc-inacap/
├── README.md                  ← Este documento
├── 00-investigacion/          ← Marco teórico completo
├── 01-equipamiento/           ← Inventario técnico y checklist de hardware
├── 02-software/               ← Stack de herramientas, versiones y requisitos
├── 03-planificacion/          ← Arquitectura, red, VLANs y tabla de IPs
├── 04-guia-proxmox/           ← Instalación paso a paso (VirtualBox → servidor real)
├── 05-guia-soc/               ← Instalación de Wazuh, Zabbix, Docker y lab vulnerable
├── 06-validacion/             ← Pruebas de funcionamiento y métricas
└── 07-pruebas/                ← Diario técnico: registro de avances y problemas
```

Cada carpeta contiene su propio `README.md` con el contenido completo de esa fase. Se recomienda seguirlas en orden.

---

## 🚀 Cómo usar este repositorio

**🎓 Si eres alumno y quieres aprender desde cero:**
Comienza por la [Fase 0 — Investigación](./00-investigacion/README.md). Ahí encontrarás todos los conceptos necesarios antes de tocar cualquier herramienta.

**🔁 Si quieres replicar el sistema en otro entorno:**
Lee la [Fase 3 — Planificación](./03-planificacion/README.md) para entender la arquitectura, luego sigue las guías de la Fase 4 y Fase 5.

**🔧 Si el sistema está instalado y necesitas mantenerlo o repararlo:**
Ve directamente a la [Fase 7 — Pruebas](./07-pruebas/README.md) donde está el registro histórico de problemas y soluciones, y a la [Fase 6 — Validación](./06-validacion/README.md) para verificar que todo funciona correctamente.

**💻 Si quieres practicar sin tocar los servidores reales:**
Sigue la [Fase 4 — Guía Proxmox, Parte A](./04-guia-proxmox/README.md), que explica cómo instalar Proxmox dentro de VirtualBox en tu propia laptop.

---

## 🚦 Estado del proyecto

| Fase | Descripción | Estado |
|---|---|---|
| 00 — Investigación | Marco teórico | ✅ Documentado |
| 01 — Equipamiento | Inventario de servidores | ✅ Documentado |
| 02 — Software | Stack tecnológico | ✅ Documentado |
| 03 — Planificación | Arquitectura y red | ✅ Documentado |
| 04 — Guía Proxmox | Instalación del hipervisor | 🔄 En progreso |
| 05 — Guía SOC | Instalación de Wazuh y Zabbix | ⏳ Pendiente |
| 06 — Validación | Pruebas de funcionamiento | ⏳ Pendiente |
| 07 — Pruebas | Diario técnico | 🔄 En progreso |

---

## 📝 Notas del docente

> Este repositorio se construye de forma incremental. A medida que se avanza en la instalación y configuración de los equipos reales en la sala de servidores, la documentación se actualiza con los hallazgos reales: problemas encontrados, soluciones aplicadas y ajustes a la arquitectura original.
>
> El objetivo es que este documento no sea solo una guía estática, sino un **registro vivo del proyecto** que refleje la experiencia real de construcción del SOC.

---

*🏫 Institución: INACAP La Serena — 📅 Actualizado: 2026*
