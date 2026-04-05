# Fase 6 — Validación

## Objetivo

Verificar que el SOC está funcionando correctamente: que los agentes envían logs, que Wazuh genera alertas ante ataques reales y que Zabbix monitorea la salud de los servidores.

> ⚠️ Esta fase asume que Wazuh, Zabbix y el laboratorio vulnerable ya están instalados. Ver [Fase 5](../05-guia-soc/README.md).

---

## Checklist de validación general

Antes de las pruebas de ataque, verificar que todo el stack está operativo:

### Proxmox

- [ ] Interfaz web de Proxmox accesible en los 3 servidores
- [ ] Todos los contenedores y VMs están en estado **Running**
- [ ] El almacenamiento tiene espacio suficiente (>50% libre)

### Wazuh

- [ ] Dashboard de Wazuh accesible en `https://192.168.10.11`
- [ ] Los agentes aparecen como **Active** en el panel de agentes
- [ ] La sección "Security Events" muestra eventos recientes

### Zabbix

- [ ] Frontend de Zabbix accesible en `http://192.168.10.13/zabbix`
- [ ] Los 3 servidores aparecen como hosts monitorizados
- [ ] No hay triggers activos de error crítico

### Laboratorio vulnerable

- [ ] DVWA accesible en `http://10.0.0.10:8080`
- [ ] Juice Shop accesible en `http://10.0.0.10:3000`
- [ ] Agente Wazuh instalado y activo en la VM del laboratorio

---

## Prueba 1 — Verificar agente Wazuh

### Objetivo
Confirmar que el agente de Wazuh en las máquinas virtuales está enviando eventos al Manager.

### Pasos

1. Abre el Dashboard de Wazuh
2. Ve a **Agents** en el menú lateral
3. Verifica que todos los agentes aparecen en verde (**Active**)
4. Haz clic en un agente → **Events** → verifica que hay eventos recientes

### Resultado esperado

```
Estado del agente: Active
Última conexión: hace X segundos
Eventos recientes: visibles en el dashboard
```

---

## Prueba 2 — Detección de fuerza bruta SSH

### Objetivo
Simular un ataque de fuerza bruta contra SSH y verificar que Wazuh genera una alerta.

### Pasos

Desde la máquina de Kali Linux (o cualquier laptop en la VLAN de laboratorio):

```bash
# Simular múltiples intentos de login SSH fallidos
# Reemplaza IP_VICTIMA con la IP de la VM objetivo
for i in {1..20}; do
  ssh usuario_falso@IP_VICTIMA
done
```

### Resultado esperado en Wazuh

En el Dashboard de Wazuh → **Security Events**, debería aparecer una alerta similar a:

```
Rule: 5760 - SSHD: Multiple failed login attempts
Level: 10 (High)
Description: Multiple authentication failures for user
```

---

## Prueba 3 — Detección de escaneo de puertos

### Objetivo
Simular un escaneo de red con Nmap y verificar que Wazuh lo detecta.

### Pasos

```bash
# Escaneo básico desde la máquina atacante
nmap -sV -O IP_VICTIMA

# Escaneo agresivo (genera más eventos)
nmap -A IP_VICTIMA
```

### Resultado esperado en Wazuh

```
Rule: Snort/Suricata/Syslog alert
Level: varies
Description: Port scan detected from IP_ATACANTE
```

---

## Prueba 4 — Detección de ataque web (DVWA)

### Objetivo
Realizar un ataque SQL Injection básico contra DVWA y verificar que Wazuh lo registra.

### Pasos

1. Abre DVWA en el navegador: `http://10.0.0.10:8080`
2. Inicia sesión (admin/password)
3. Ve a **DVWA Security** → pon el nivel en **Low**
4. Ve a **SQL Injection**
5. En el campo de entrada, escribe: `1' OR '1'='1`
6. Observa el comportamiento de la aplicación

### Resultado esperado en Wazuh

```
Rule: Web attack - SQL injection
Level: 6 (Medium)
Description: SQL injection attempt detected
```

---

## Prueba 5 — Monitoreo de hardware con Zabbix

### Objetivo
Verificar que Zabbix muestra el estado real del hardware de los servidores.

### Pasos

1. Abre Zabbix Frontend
2. Ve a **Monitoring → Hosts**
3. Verifica que los 3 servidores aparecen (Dell R720, Dell R420, HP DL180)
4. Haz clic en uno → **Latest data**
5. Verifica que hay datos de: CPU usage, Memory usage, Disk usage

---

## Prueba 6 — Simulación de falla de servicio

### Objetivo
Detener el agente Wazuh en una máquina y verificar que tanto Wazuh como Zabbix generan alertas.

### Pasos

En la máquina monitorizada:

```bash
# Detener el agente Wazuh
systemctl stop wazuh-agent
```

### Resultado esperado

**En Wazuh Dashboard:**
```
Agent Status: Disconnected
Alert: Agent stopped sending events
```

**En Zabbix:**
```
Trigger: Wazuh agent is not running on HOST
Severity: High
```

---

## Métricas de validación final

Una vez completadas las pruebas, documentar los resultados:

| Prueba | Resultado | Tiempo de detección | Nivel de alerta | Observaciones |
|---|---|---|---|---|
| Agente Wazuh activo | | | | |
| Fuerza bruta SSH | | | | |
| Escaneo de puertos | | | | |
| SQL Injection | | | | |
| Monitoreo Zabbix | | | | |
| Falla de servicio | | | | |

---

## Preguntas de evaluación para los alumnos

1. ¿Qué diferencia hay entre una alerta de nivel 3 y una de nivel 10 en Wazuh?
2. ¿Por qué es importante que el laboratorio vulnerable esté en una VLAN separada?
3. ¿Qué información entrega el agente Wazuh que no entrega Zabbix?
4. ¿Cómo responderías ante una alerta de fuerza bruta real en producción?
5. ¿Qué es un falso positivo en seguridad? ¿Viste alguno durante las pruebas?

---

*← [Guía SOC](../05-guia-soc/README.md) | Siguiente: [Pruebas →](../07-pruebas/README.md)*
