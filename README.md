# Infrastructure & Security Labs (`Infrastructure-labs`)

Repositorio de documentación técnica, arquitectura y registros de decisiones para el despliegue de infraestructura híbrida empresarial (Linux, Windows Server, MikroTik y Zabbix).

---

## 🗂️ Estructura del Repositorio

| Módulo | Descripción | Estado |
| :--- | :--- | :---: |
| [**`01-base-infrastructure/debian-server`**](./01-base-infrastructure/debian-server) | Despliegue base de nodos Debian, Hardening SSH y gestión de usuarios. |  Completed |
| [**`02-networking-and-topology`**](./02-networking-and-topology) | Topología lógica de red, segmentación en VLANs y reglas de firewall en MikroTik. |  Completed |
| [**`03-monitoring-and-observability`**](./03-monitoring-and-observability) | Monitoreo unificado con Zabbix, agentes en Linux/Windows y SNMP en MikroTik. |  Completed |

---

## 📐 Resumen de la Topología

- **Router / Firewall:** MikroTik (VLANs 10, 20, 30, 40)
- **SOC / SIEM / Monitoreo:** Debian 13 con Docker + Zabbix Server (`10.10.40.10`)
- **DMZ:** Debian 13 (`10.10.10.10`)
- **Management / AD:** Windows Server (`10.10.20.10`)
