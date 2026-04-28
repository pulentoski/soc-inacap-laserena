

## Registro de pruebas del proyecto

Esta sección es el **diario técnico** del proyecto. Aquí se registra todo lo que ocurre durante la instalación y configuración: problemas encontrados, soluciones aplicadas, decisiones tomadas y lecciones aprendidas.

> 📝 Actualizar este archivo cada vez que se realice una prueba o se avance en el proyecto.

---

## Formato de registro

Cada prueba se registra con la siguiente estructura:

```
### [FECHA] — [DESCRIPCIÓN CORTA]
- **Equipo/Entorno:** (laptop, Dell R720, etc.)
- **Objetivo:** qué se intentó hacer
- **Resultado:** qué pasó
- **Problema encontrado:** (si hubo alguno)
- **Solución:** cómo se resolvió
- **Estado:** ✅ Exitoso / ❌ Fallido / 🔄 En progreso
```

---

## Pruebas registradas

---

### [2026-04-05] — Instalación de Proxmox en VirtualBox (laptop Ubuntu)

- **Equipo/Entorno:** Laptop con Ubuntu, VirtualBox, usuario normal del sistema / root
- **Objetivo:** Instalar Proxmox VE 9.1 en VirtualBox para práctica previa a los servidores reales
- **Configuración inicial de la VM:**
  - Nombre: `proxmox`
  - RAM: 4759 MB
  - CPU: 2 núcleos
  - Disco: 102 GB (VDI)
  - Red: Adaptador puente
  - ISO: proxmox-ve_9.1-1.iso

**Problema 1 — Nested VT-x bloqueado**

La casilla "Nested VT-x/AMD-V" aparecía gris en VirtualBox. Se intentó el comando como root pero falló:
```
VBoxManage: error: Could not find a registered machine named 'proxmox'
```
Causa: las VMs están registradas bajo el usuario normal del sistema, no bajo root.

Solución:
```bash
su - TU_USUARIO -c "VBoxManage list vms"
# Resultado: "proxmox" {e3a7bed9-83fd-49e5-8bc7-75eed7d0bddf}

su - TU_USUARIO -c "VBoxManage modifyvm 'proxmox' --nested-hw-virt on"

su - TU_USUARIO -c "VBoxManage showvminfo proxmox | grep -i nested"
# Resultado: Nested VT-x/AMD-V: enabled ✅
```

**Resultado — Proxmox instalado correctamente**

- IP asignada: `IP_PROXMOX_VIRTUALBOX`
- Versión: Proxmox VE 9.1.1
- Nodo: `pve`
- Almacenamiento: `local (pve)` + `local-zfs (pve)`
- Acceso web: `https://IP_PROXMOX_VIRTUALBOX:8006` ✅

---

**Problema 2 — Error 401 en apt update tras ejecutar script post-install**

Se ejecutó el script de post-instalación de community-scripts:
```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/tools/pve/post-pve-install.sh)"
```

El script dejó los repositorios enterprise activos en archivos `.sources` que no procesó:
```
Err: https://enterprise.proxmox.com/debian/pve trixie InRelease
  401  Unauthorized
```

Investigación:
```bash
grep -r "enterprise.proxmox" /etc/apt/
# Reveló: pve-enterprise.sources y ceph.sources activos sin "Enabled: no"
```

Solución:
```bash
echo "Enabled: no" >> /etc/apt/sources.list.d/pve-enterprise.sources
echo "Enabled: no" >> /etc/apt/sources.list.d/ceph.sources
apt update
# Resultado: 117 packages can be upgraded ✅
```

**Lección aprendida:** En Proxmox 9.x los repositorios enterprise están en archivos `.sources` además de los `.list`. El script de community-scripts solo deshabilita los `.list`.

---

**Avance actual — Descarga de plantilla Debian 12**

Se inició descarga de plantilla `debian-12-standard_12.12-1_amd64.tar.zst` (118 MB) desde la interfaz gráfica:
`local (pve)` → CT Templates → Templates → debian-12-standard → Download

- **Estado:** 🔄 En progreso — descarga en curso

---

## Plantilla para próximas pruebas

Copiar y pegar para cada nueva prueba:

```markdown
### [FECHA] — [DESCRIPCIÓN]
- **Equipo/Entorno:**
- **Objetivo:**
- **Resultado:**
- **Problema encontrado:**
- **Solución:**
- **Estado:** ✅ / ❌ / 🔄
```

---

## Problemas conocidos y soluciones

| # | Problema | Entorno | Solución | Estado |
|---|---|---|---|---|
| 1 | VT-x anidado bloqueado en VirtualBox | Laptop | Usar `VBoxManage modifyvm` en terminal admin | ✅ Documentado |
| 2 | | | | |
| 3 | | | | |

---

## Decisiones técnicas tomadas

| Fecha | Decisión | Alternativa descartada | Razón |
|---|---|---|---|
| 2026-04-05 | Usar Proxmox VE como hipervisor | VMware ESXi | Open source, sin licencia, mejor para academia |
| 2026-04-05 | Instalar Wazuh en LXC | Instalación bare metal | Menor consumo de recursos |
| 2026-04-05 | Docker para laboratorio vulnerable | VMs individuales | Más fácil de gestionar y resetear |

---

## Log de versiones instaladas

| Componente | Versión planeada | Versión instalada | Fecha | Notas |
|---|---|---|---|---|
| Proxmox VE | 8.x | | | |
| Wazuh | 4.7.x | | | |
| Zabbix | 6.4 LTS | | | |
| Docker CE | 24.x+ | | | |
| Ubuntu Server | 22.04 LTS | | | |

---

## Notas del docente

> Espacio libre para notas rápidas durante el proyecto.

```
[2026-04-05] - Inicio del proyecto. Comenzando con prueba de Proxmox en VirtualBox
              en laptop ASUS TUF. Los servidores del rack serán configurados
              una vez dominado el entorno virtual.
```

---

*← [Validación](../06-validacion/README.md) | [Volver al índice →](../README.md)*
