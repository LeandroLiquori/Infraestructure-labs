# 01 - Base Infrastructure: Server Provisioning & Hardening

Este módulo documenta el aprovisionamiento base, especificaciones de hardware virtual y decisiones de endurecimiento (*hardening*) de los servidores de la infraestructura.

---

## 1. Inventario y Especificaciones de Nodos Base

### 🌐 A. Servidor DMZ (`SRV-DEBIAN-DMZ-01`)
* **Hostname:** `srv-debian01` / `SRV-DEBIAN-DMZ-01`
* **Sistema Operativo:** Debian 13 (Trixie) - Modo Consola (No GUI)
* **Hypervisor:** VirtualBox
* **Recursos Virtuales:** 2 vCPUs, 2048 MB RAM, 20 GB Disk (SATA VDI)
* **Direccionamiento IP:** `10.10.10.10/24` (VLAN 10 - DMZ)
* **Función Principal:** Servidor web / servicios expuestos en DMZ.

---

### 🛡️ B. Servidor SOC & Monitoreo (`SRV-DEBIAN-SOC-01`)
* **Hostname:** `srv-debian01` / `SRV-DEBIAN-SOC-01`
* **Sistema Operativo:** Debian 13 (Trixie) - Modo Consola (No GUI)
* **Hypervisor:** VirtualBox
* **Recursos Virtuales:** 2 vCPUs, 4096 MB RAM, 30 GB Disk (SATA VDI)
* **Runtime de Contenedores:** Docker & Docker Compose
* **Direccionamiento IP:** `10.10.40.10/24` (VLAN 40 - SOC)
* **Función Principal:** Servidor Zabbix Server (Containerized) y gestión de logs.

---

### 🏢 C. Controlador de Dominio & Management (`SRV-WINDOWS-AD-01`)
* **Hostname:** `SRV-WINDOWS-AD-01` / `srv-dc01-WS`
* **Sistema Operativo:** Windows Server 2022 (Server Core / GUI)
* **Hypervisor:** VirtualBox
* **Recursos Virtuales:** 2 vCPUs, 4096 MB RAM, 40 GB Disk (SATA VDI)
* **Direccionamiento IP:** `10.10.20.10/24` (VLAN 20 - Management)
* **Función Principal:** Active Directory Domain Services (AD DS) y DNS Primario.

---

## 2. Registro de Decisiones de Arquitectura (Architecture Log)

### Controlador de Disco (SATA vs NVMe)
Se utiliza controlador SATA emulado por el hypervisor en todos los nodos para garantizar la compatibilidad nativa con los drivers del instalador base de Debian y Windows Server, sin requerir módulos de kernel adicionales durante la fase de bootstrap.

### Gestión de Accesos y Privilegios

#### Nodos Linux (Debian)
* **Desactivación de Root Directo:** Se instaló y configuró `sudo`, asignando al usuario estándar al grupo de administración mediante `usermod -aG sudo`.
* **Autenticación en GitHub / SSH:** Se generó un par de llaves criptográficas asimétricas Ed25519 (`ssh-keygen -t ed25519`) para autenticación por firma *Challenge-Response*, eliminando el uso de contraseñas estáticas en la terminal.

#### Nodo Windows Server
* **Administración Remota vía OpenSSH:** Se instaló la característica de servidor SSH nativa (`OpenSSH.Server`) vía PowerShell para permitir administración en consola unificada sin requerir entorno gráfico (RDP) desde Termius.

---

## 3. Políticas de Hardening Aplicadas

### 🔒 Linux (Debian DMZ & SOC)
1. **Hardening de SSH (`/etc/ssh/sshd_config`):**
   * Deshabilitación del acceso directo de root: `PermitRootLogin no`
   * Restricción de métodos de autenticación a llaves RSA/Ed25519 o usuarios privilegiados en `sudoers`.

### 🛡️ Windows Server
1. **Apertura Mínima de Puertos en Windows Firewall:**
   * Regla de entrada explícita para SSH (TCP 22):
     ```powershell
     New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
     ```
   * Regla de entrada explícita para Agente de Monitoreo (TCP 10050):
     ```powershell
     New-NetFirewallRule -DisplayName "Zabbix Agent" -Direction Inbound -Protocol TCP -LocalPort 10050 -Action Allow
     ```
