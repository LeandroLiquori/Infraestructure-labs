# 02 - Plan de Redes y Topología Lógica (VLANs & Routing)

## 1. Diagrama de Arquitectura
![Topología de Red](DiagramaDeRed.drawio.png)

## 📊 Tabla de Subredes e Infraestructura

| VLAN | Nombre | Subred | IP Gateway | IP Servidores | Función Principal |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **VLAN 10** | DMZ | `10.10.10.0/24` | `10.10.10.1` | `10.10.10.10` (`srv-debian01`) | Servidor expuesto (DST-NAT 2222) |
| **VLAN 20** | MANAGEMENT | `10.10.20.0/24` | `10.10.20.1` | `10.10.20.10` (`srv-dc01`) | Active Directory (lab.local) + DNS |
| **VLAN 30** | LAN | `10.10.30.0/24` | `10.10.30.1` | Dynamic (DHCP) | Clientes / Workstations |
| **VLAN 40** | SOC | `10.10.40.0/24` | `10.10.40.1` | - | Monitoreo y SIEM |

## 3. Políticas de Tráfico y Aislamiento (Firewall Policy Draft)
- **VLAN 10 (DMZ):** Acceso público/externo permitido. Aislamiento estricto hacia VLAN 20 y VLAN 40.
- **VLAN 20 (Management):** Solo accesible por SSH/WinRM desde la VLAN 30 (LAN) previa autenticación.
- **VLAN 30 (LAN):** Salida a Internet vía NAT en el router MikroTik. Acceso restringido a servicios de DMZ y Management.
- **VLAN 40 (SOC):** Acceso de lectura/ingestión de logs (RSYSLOG / Wazuh Agent) desde todas las VLANs. Bloqueado el tráfico iniciado desde otras VLANs hacia la consola web del SOC.
