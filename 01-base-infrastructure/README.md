# 01 - Base Infrastructure: Server Provisioning & Hardening

Este módulo documenta el aprovisionamiento base, especificaciones de hardware virtual y decisiones de endurecimiento (*hardening*) de todos los nodos de la infraestructura (Servidores y Appliances de Red).

---

## 1. Inventario y Especificaciones de Nodos Base

### 🌐 A. Router & Edge Firewall (`fw-edge01-MikroTik`)
* **Hostname:** `MikroTik-Router` / `fw-edge01`
* **Sistema Operativo:** MikroTik RouterOS v7 (CHR / Virtual Appliance)
* **Hypervisor:** VirtualBox
* **Recursos Virtuales:** 1 vCPU, 256 MB RAM, 128 MB Disk
* **Interfaces & Direccionamiento:**
  * `ether1` (WAN / Bridged): `192.168.0.39/24` (Acceso administrativo / NAT)
  * `vlan10` (DMZ): `10.10.10.1/24`
  * `vlan20` (MANAGEMENT): `10.10.20.1/24`
  * `vlan30` (LAN): `10.10.30.1/24`
  * `vlan40` (SOC): `10.10.40.1/24`
* **Función Principal:** Gateway de red, enrutamiento Inter-VLAN, NAT (Src-NAT / Dst-NAT), Firewall de borde y Agente SNMP.

---

### 🌐 B. Servidor DMZ (`SRV-DEBIAN-DMZ-01`)
* **Hostname:** `srv-debian01` / `SRV-DEBIAN-DMZ-01`
* **Sistema Operativo:** Debian 13 (Trixie) - Modo Consola (No GUI)
* **Hypervisor:** VirtualBox
* **Recursos Virtuales:** 2 vCPUs, 2048 MB RAM, 20 GB Disk (SATA VDI)
* **Direccionamiento IP:** `10.10.10.10/24` (VLAN 10 - DMZ) | Gateway: `10.10.10.1`
* **Función Principal:** Servidor web / servicios expuestos en DMZ.

---

### 🛡️ C. Servidor SOC & Monitoreo (`SRV-DEBIAN-SOC-01`)
* **Hostname:** `srv-debian01` / `SRV-DEBIAN-SOC-01`
* **Sistema Operativo:** Debian 13 (Trixie) - Modo Consola (No GUI)
* **Hypervisor:** VirtualBox
* **Recursos Virtuales:** 2 vCPUs, 4096 MB RAM, 30 GB Disk (SATA VDI)
* **Runtime de Contenedores:** Docker & Docker Compose
* **Direccionamiento IP:** `10.10.40.10/24` (VLAN 40 - SOC) | Gateway: `10.10.40.1`
* **Función Principal:** Servidor Zabbix Server (Containerized) y gestión de logs.

---

### 🏢 D. Controlador de Dominio & Management (`SRV-WINDOWS-AD-01`)
* **Hostname:** `SRV-WINDOWS-AD-01` / `srv-dc01-WS`
* **Sistema Operativo:** Windows Server 2022 (Server Core / GUI)
* **Hypervisor:** VirtualBox
* **Recursos Virtuales:** 2 vCPUs, 4096 MB RAM, 40 GB Disk (SATA VDI)
* **Direccionamiento IP:** `10.10.20.10/24` (VLAN 20 - Management) | Gateway: `10.10.20.1`
* **Función Principal:** Active Directory Domain Services (AD DS) y DNS Primario.

---

## 2. Registro de Decisiones de Arquitectura (Architecture Log)

### Controlador de Disco e Interfaces Virtuales
* **Servidores (SATA vs NVMe):** Se utiliza controlador SATA emulado por el hypervisor en todos los nodos finales para garantizar compatibilidad nativa con los drivers base.
* **RouterOS (Network Interfaces):** Configuración del adaptador principal en modo Puente (*Bridged*) para simular la interfaz WAN física y permitir mapeos NAT de gestión (*DST-NAT*).

### Gestión de Accesos y Privilegios

#### Router (RouterOS)
* **Acceso Administrativo:** Deshabilitación de servicios no cifrados (Telnet, FTP, HTTP) en `/ip service`, restringiendo la administración únicamente a WinBox, SSH cifrado y la API web segura (HTTPS).
* **SNMP Telemetry:** Habilitación de SNMP v2c limitado al rango de la VLAN 40 (`10.10.40.10`) para permitir el scraping seguro de métricas desde Zabbix.

#### Nodos Linux (Debian)
* **Desactivación de Root Directo:** Instalación y configuración de `sudo`, asignando el usuario estándar al grupo `sudo`.
* **Autenticación SSH:** Generación de llaves Ed25519 (`ssh-keygen -t ed25519`) para acceso remoto por firma *Challenge-Response*.

#### Nodo Windows Server
* **Administración Remota vía OpenSSH:** Instalación del servicio `OpenSSH.Server` vía PowerShell para permitir administración en consola unificada (Termius) sin requerir entorno gráfico (RDP).

---

## 3. Políticas de Hardening Aplicadas

### 🎛️ MikroTik RouterOS
1. **Reglas de Redirección (*DST-NAT*) seguras:** Mapeo explícito de puertos externos hacia las IPs de gestión internas validando la IP de destino WAN (`Dst. Address: 192.168.0.39`).
2. **Aislamiento Inter-VLAN:** Filtrado de tráfico no autorizado entre subredes desde `/ip firewall filter`.

### 🔒 Linux (Debian DMZ & SOC)
1. **Hardening de SSH (`/etc/ssh/sshd_config`):**
   * Deshabilitación del acceso directo de root: `PermitRootLogin no`.

### 🛡️ Windows Server
1. **Apertura Mínima de Puertos en Windows Firewall:**
   * Regla para SSH (TCP 22) y Zabbix Agent (TCP 10050) restringidas por protocolo y puerto local.
