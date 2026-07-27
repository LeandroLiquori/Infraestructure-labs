# 02 - Plan de Redes y Topología Lógica (VLANs & Routing)

## 1. Diagrama de Arquitectura
![Topología de Red](topology.png)

## 2. Matriz de Direccionamiento IP (IP Scheme)

| Segmento / VLAN | ID VLAN | Red / CIDR | Gateway (Router) | Host Asignado | IP Host |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **DMZ** | 10 | `10.10.10.0/24` | `10.10.10.1` | `srv-debian01` | `10.10.10.10` |
| **Management** | 20 | `10.10.20.0/24` | `10.10.20.1` | `srv-freeipa01` | `10.10.20.10` |
| **LAN Corporativa** | 30 | `10.10.30.0/24` | `10.10.30.1` | `PC-W11-01` | `10.10.30.10` |
| **SOC & Monitoreo** | 40 | `10.10.40.0/24` | `10.10.40.1` | `srv-soc01` | `10.10.40.10` |

## 3. Políticas de Tráfico y Aislamiento (Firewall Policy Draft)
- **VLAN 10 (DMZ):** Acceso público/externo permitido. Aislamiento estricto hacia VLAN 20 y VLAN 40.
- **VLAN 20 (Management):** Solo accesible por SSH/WinRM desde la VLAN 30 (LAN) previa autenticación.
- **VLAN 30 (LAN):** Salida a Internet vía NAT en el router MikroTik. Acceso restringido a servicios de DMZ y Management.
- **VLAN 40 (SOC):** Acceso de lectura/ingestión de logs (RSYSLOG / Wazuh Agent) desde todas las VLANs. Bloqueado el tráfico iniciado desde otras VLANs hacia la consola web del SOC.
