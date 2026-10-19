# Red y firewall: pfSense, VLANs y reglas

Estado: `operativo` | Última revisión: `18/10/2026`

## Objetivo

Segmentar la red del laboratorio como en una empresa real: servidores, clientes y acceso remoto separados, con pfSense como router y cortafuegos entre zonas y política de mínimo privilegio.

## Dónde vive

| Dato | Valor |
|---|---|
| Máquina | VM `pfSense` en Proxmox |
| Versión | pfSense CE `2.9.0` |
| Interfaces | WAN, LAN, SERVIDORES_INTER, CLIENTES_INTER, OpenVPN |
| Red de gestión del host Proxmox | `192.168.1.130/24`, puerta de enlace `192.168.1.1` |
| VLAN de servidores | `192.168.10.0/24` |
| VLAN de clientes | `192.168.20.0/24` |

## Configuración

### 1. Bridge de Proxmox con soporte de VLAN

- En **Nodo → Sistema → Red**, se edita `vmbr0` se deja como está ya que va a ser la conexión WAN.
- En **Nodo → Sistema → Red**, se edita `vmbr1` y se activa **Consciente de VLAN** (VLAN aware). Se aplica la configuración de red y se reinicia el bridge.

### 2. Asignación de VLAN a las máquinas

Cada VM recibe su VLAN Tag en la tarjeta de red virtual de Proxmox:

| Máquina | VLAN |
|---|---|
| Controlador de dominio, TrueNAS, docker01 | 10 (Servidores) |
| Cliente | 20 (Clientes) |
| Administración | 1 (Administración de red y PFsense) |

## Reglas de firewall

| Interfaz | Regla | Comentario |
|---|---|---|
| WAN | Permite UDP 1194 hacia la dirección WAN | Solo la VPN. Creada por el asistente de OpenVPN |
| LAN | Permite todo desde LAN subnets (reglas por defecto) | Aceptable porque es la red de gestión |
| CLIENTES_INTER | Permite TCP/UDP hacia SERVIDORES_INTER solo en el alias `Puertos_AD` | Mínimo privilegio |
| CLIENTES_INTER | Permite salida a `! Redes_privadas` | Internet sí, otras redes privadas no |
| SERVIDORES_INTER | Permite desde LAN subnets y desde su propia subred | Pendiente de endurecer |
| OpenVPN Administración | Permite todo | Se deja que puede acceder a la red de servidores |
| OpenVPN Usuarios | Acceso desde fuera a usuarios | Se aplican las restricciones de CLIENTES_INTER |

## Validación realizadas
- Inicio de sesión contra Active Directory desde VLAN de clientes.
- Resolución DNS del dominio.
- Acceso controlado a recursos autorizados.
- Bloqueo de puertos fuera del conjunto permitido.

## Mejoras pendientes

- Activar el registro (log) de la regla de WAN y limitar las conexiones nuevas por origen.
- Sustituir la salida abierta de SERVIDORES_INTER por reglas concretas (actualizaciones, NTP).
- Restringir la interfaz OpenVPN a los destinos y puertos necesarios y activar su registro.
- Instalar pfBlockerNG.
- IDS/IPS con Suricata.

## Problemas encontrados

Ver [incidencias](incidencias.md).
