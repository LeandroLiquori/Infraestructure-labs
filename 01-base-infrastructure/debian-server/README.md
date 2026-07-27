# 01 - Base Infrastructure: Debian Server (srv-debian01)

## 1. Especificaciones del Nodo
- **Hostname:** srv-debian01
- **Hypervisor:** VirtualBox
- **Sistema Operativo:** Debian 13 (Trixie) - Modo Consola (No GUI)
- **Recursos Virtuales:** 2 vCPUs, 2048 MB RAM, 20 GB Disk (SATA VDI)
- **Modo de Red:** Adaptador Puente (Bridged)

## 2. Registro de Decisiones de Arquitectura (Architecture Log)

### Controlador de Disco (SATA vs NVMe)
Se utiliza controlador SATA emulado por el hypervisor para garantizar la compatibilidad nativa con los drivers del instalador base de Debian sin requerir módulos de kernel adicionales durante la fase de bootstrap.

### Interfaz de Red en Modo Bridged (Puente)
Se configuró la placa de red virtual en modo Bridged para exponer el servidor directamente en el segmento LAN físico, permitiendo asignación de IP independiente y gestión SSH in-band desde estaciones de trabajo en la misma red local.

### Gestión de Accesos y Privilegios
- **Desactivación de Root Directo:** Se instaló y configuró `sudo`, asignando al usuario estándar al grupo de administración mediante `usermod -aG sudo`.
- **Autenticación en GitHub:** Se generó un par de llaves criptográficas asimétricas Ed25519 (`ssh-keygen -t ed25519`) para autenticación por firma *Challenge-Response*, eliminando el uso de contraseñas estáticas en la terminal.

### Hardening
- **Hardening de SSH:** Se deshabilitó el acceso directo del superusuario modificando `PermitRootLogin no` en `/etc/ssh/sshd_config`, obligando a los administradores a ingresar con credenciales personales y escalar privilegios vía `sudo`.
