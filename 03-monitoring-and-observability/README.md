# 03 - Monitoreo y Observabilidad (Zabbix & Telemetría)

## 1. Arquitectura de Monitoreo

Se implementó una solución de monitoreo centralizada utilizando **Zabbix Server 7.0 (Dockerizado)** ubicado en el segmento **VLAN 40 (SOC)**. La recolección de métricas se realiza mediante dos estrategias principales:

1. **SNMP v2c:** Para dispositivos de red unificados (Router MikroTik).
2. **Zabbix Agent:** Para servidores finales (Debian Linux y Windows Server).

---

## 2. Inventario de Nodos Monitoreados

| Hostname | Sistema Operativo | IP | Método | Template Aplicado | Status |
| :--- | :--- | :--- | :---: | :--- | :---: |
| `MikroTik-Router` | RouterOS | `10.10.40.1:161` | SNMP v2c | `MikroTik by SNMP` |  OK |
| `SRV-DEBIAN-SOC-01` | Debian 13 | `10.10.40.10:10050` | Zabbix Agent | `Linux by Zabbix agent` |  OK |
| `SRV-DEBIAN-DMZ-01` | Debian 13 | `10.10.10.10:10050` | Zabbix Agent | `Linux by Zabbix agent` |  OK |
| `SRV-WINDOWS-AD-01` | Windows Server | `10.10.20.10:10050` | Zabbix Agent | `Windows by Zabbix agent` |  OK |

---

## 3. Guía de Instalación y Hardening de Agentes

### A. Servidores Linux (Debian)
1. Instalación del agente oficial:
   ```bash
   sudo apt update && sudo apt install zabbix-agent -y

### B. Configuracion en /etc/zabbix/zabbbix_agentd.conf

Server=127.0.0.1,10.10.40.10,172.16.0.0/12
ServerActive=10.10.40.10
Hostname=SRV-DEBIAN-DMZ-01 (Dependiendo del nombre colocado para cada servidor en zabbix)

### Servidor Windows (Active Directory)
Get-WindowsCapability -Online -Name "OpenSSH.Server*" | Add-WindowsCapability -Online
Start-Service sshd
Set-Service -Name sshd -StartupType 'Automatic'
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22

Invoke-WebRequest -Uri "[https://cdn.zabbix.com/zabbix/binaries/stable/6.4/6.4.15/zabbix_agent-6.4.15-windows-amd64-openssl.msi](https://cdn.zabbix.com/zabbix/binaries/stable/6.4/6.4.15/zabbix_agent-6.4.15-windows-amd64-openssl.msi)" -OutFile "$env:TEMP\zabbix_ageInvoke-WebRequest -Uri "[https://cdn.zabbix.com/zabbix/binaries/stable/6.4/6.4.15/zabbix_agent-6.4.15-windows-amd64-openssl.msi](https://cdn.zabbix.com/zabbix/binaries/stable/6.4/6.4.15/zabbix_agent-6.4.15-windows-amd64-openssl.msi)" -OutFile "$env:TEMP\zabbix_agent.msi"

msiexec /i "$env:TEMP\zabbix_agent.msi" /qn SERVER="10.10.40.10,10.10.20.1,172.16.0.0/12" SERVERACTIVE="10.10.40.10" HOSTNAME="SRV-WINDOWS-AD-01" ENABLEPATH=1

New-NetFirewallRule -DisplayName "Zabbix Agent" -Direction Inbound -Protocol TCP -LocalPort 10050 -Action Allown

### 4. Registro de Resoluciones Técnicas (Troubleshooting Log)

Resolución de rechazo de conexiones por NAT e Inter-VLAN (Access Permissions)
Problema: Los agentes de la DMZ y Management rechazaban las consultas pasivas del Zabbix Server (Received empty response from Zabbix Agent... assuming agent dropped connection).

Causa Raíz: Al estar Zabbix Server corriendo en Docker y cruzando el router MikroTik, las peticiones llegaban con la IP del Gateway del router (10.10.X.1) o con la subred del bridge de Docker (172.16.0.0/12), las cuales no estaban autorizadas en los agentes.

Solución: Se incluyeron las subredes del bridge de Docker y las IPs de los gateways del MikroTik en la directiva Server= de todos los archivos de configuración (zabbix_agentd.conf).

Corrección en Regla de Redirección (DST-NAT)
Problema: Fallo de timeout al intentar conectar Termius al puerto SSH de Windows mediante la WAN del MikroTik.

Causa Raíz: Discrepancia en la IP de destino de la regla DST-NAT (192.169.0.39 vs 192.168.0.39).

Solución: Corrección del octeto en el campo Dst. Address de la regla de NAT en RouterOS.
