# Fase 0 — Investigación

## ¿Qué es un SOC?

Un **SOC (Security Operations Center)** es un equipo o instalación centralizada donde profesionales de seguridad monitorean, detectan, analizan y responden a incidentes de ciberseguridad en tiempo real.

En términos simples: es la "sala de control" de la seguridad de una organización. Desde ahí se pueden ver todos los eventos que ocurren en la red, los servidores y los dispositivos, y se actúa cuando algo sospechoso aparece.

### Funciones principales de un SOC

- Monitoreo continuo (24/7) de la infraestructura
- Detección de amenazas e intrusiones
- Análisis de logs (registros de eventos)
- Respuesta a incidentes
- Generación de reportes de seguridad

---

## ¿Qué es un SIEM?

Un **SIEM (Security Information and Event Management)** es el software central de un SOC. Recolecta logs de todos los dispositivos de la red, los analiza y genera alertas cuando detecta comportamientos anómalos.

### Flujo de un SIEM

```
Dispositivos/Servidores → Envían logs → SIEM los recolecta → Analiza → Genera alertas
```

**Ejemplo práctico:** Si alguien intenta iniciar sesión 50 veces en un servidor con contraseña incorrecta, el SIEM detecta ese patrón y genera una alerta de "fuerza bruta".

---

## ¿Qué es Wazuh?

**Wazuh** es una plataforma open source (gratuita) de seguridad que funciona como SIEM y XDR (Extended Detection and Response). Es la herramienta principal de este proyecto.

### Componentes de Wazuh

| Componente | Función |
|---|---|
| **Wazuh Manager** | Cerebro del sistema, recibe y procesa los eventos |
| **Wazuh Indexer** | Base de datos donde se guardan todos los logs |
| **Wazuh Dashboard** | Interfaz web para visualizar alertas y reportes |
| **Wazuh Agent** | Programa instalado en cada máquina monitorizada, envía logs al Manager |

### ¿Cómo funciona en nuestro proyecto?

```
Máquinas vulnerables (con agente Wazuh)
        ↓
   Wazuh Manager
        ↓
   Wazuh Indexer
        ↓
  Wazuh Dashboard  ←  El alumno ve las alertas aquí
```

---

## ¿Qué es Zabbix?

**Zabbix** es una herramienta open source de monitoreo de infraestructura. A diferencia de Wazuh (que se enfoca en seguridad), Zabbix monitorea el estado del hardware y los servicios.

### ¿Qué monitorea Zabbix en este proyecto?

- CPU, RAM y temperatura de los servidores
- Estado de los discos
- Disponibilidad de servicios (si Wazuh está activo, si la red funciona)
- Ancho de banda de la red

### Diferencia entre Wazuh y Zabbix

| | Wazuh | Zabbix |
|---|---|---|
| **Enfoque** | Seguridad (alertas de ataques) | Infraestructura (salud del sistema) |
| **¿Qué detecta?** | Intrusiones, malware, cambios sospechosos | CPU alta, disco lleno, servicio caído |
| **En nuestro SOC** | Ver ataques en tiempo real | Ver si los servidores están bien |

---

## ¿Qué es Proxmox VE?

**Proxmox VE** es un sistema operativo (hipervisor de Tipo 1) que se instala directamente sobre el hardware del servidor. Su función es crear y gestionar máquinas virtuales y contenedores.

En términos simples: es como instalar Windows, pero en vez de darte escritorio, te da un panel web desde donde puedes crear múltiples "computadores virtuales" dentro del servidor físico.

### Dos tipos de virtualización en Proxmox

| Tipo | Nombre técnico | Cuándo usarlo |
|---|---|---|
| **Máquina Virtual** | KVM | Para instalar Windows, firewalls, sistemas que requieren aislamiento total |
| **Contenedor** | LXC | Para servicios Linux ligeros (Wazuh, Zabbix). Consume mucha menos RAM |

---

## ¿Qué es Docker?

**Docker** es una plataforma para crear y ejecutar contenedores de aplicaciones. En este proyecto lo usaremos en el servidor de laboratorio para levantar aplicaciones vulnerables que los alumnos atacarán.

### Aplicaciones vulnerables que usaremos

| Aplicación | Descripción |
|---|---|
| **DVWA** (Damn Vulnerable Web Application) | Aplicación web con vulnerabilidades conocidas para practicar |
| **Juice Shop** | Tienda web vulnerable de OWASP para aprender ataques web |
| **Metasploitable** | Sistema Linux lleno de vulnerabilidades, ideal para practicar |

---

## Glosario de términos

| Término | Definición simple |
|---|---|
| **Log** | Registro de eventos que genera un sistema (ej: "usuario X inició sesión a las 14:32") |
| **Alerta** | Notificación que genera el SIEM cuando detecta algo sospechoso |
| **Agente** | Programa pequeño instalado en una máquina que envía sus logs al SIEM |
| **RAID** | Configuración de múltiples discos duros para tener respaldo automático |
| **Snapshot** | "Foto" del estado de una máquina virtual. Permite volver atrás si algo falla |
| **Hipervisor** | Software que permite crear y gestionar máquinas virtuales |
| **IaaS** | Infraestructura como Servicio — entregar recursos de cómputo virtualizados |
| **KVM** | Virtualización completa, crea máquinas virtuales con hardware emulado |
| **LXC** | Contenedores ligeros que comparten el kernel del sistema anfitrión |

---

## Referencias

- [Documentación oficial de Wazuh](https://documentation.wazuh.com)
- [Documentación oficial de Proxmox VE](https://pve.proxmox.com/wiki/Main_Page)
- [Documentación oficial de Zabbix](https://www.zabbix.com/documentation)
- [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)
- Norma ISO/IEC 17788 — Cloud Computing Vocabulary

---

*← [Volver al índice](../README.md) | Siguiente: [Equipamiento →](../01-equipamiento/README.md)*
