# Red y firewall: pfSense, VLANs y reglas

Estado: `operativo`
Última revisión: `<AAAA-MM-DD>`

## Objetivo

Segmentar la red del laboratorio como en una empresa real: servidores, clientes y acceso remoto separados, con pfSense como router y cortafuegos entre zonas y política de mínimo privilegio.

## Dónde vive

| Dato | Valor |
|---|---|
| Máquina | VM `pfSense` en Proxmox |
| Versión | pfSense CE `<versión>` |
| Interfaces | WAN, LAN, SERVIDORES_INTER, CLIENTES_INTER, OpenVPN |
| Red de gestión del host Proxmox | `192.168.1.130/24`, puerta de enlace `192.168.1.1` |
| VLAN de servidores | `192.168.10.0/24` |
| VLAN de clientes | `<rango>` |

## Requisitos previos

- Proxmox instalado con el bridge `vmbr0` (puerto físico `nic0`).
- ISO de pfSense CE descargada de la web oficial (pfsense.org).

## Configuración

### 1. Bridge de Proxmox con soporte de VLAN

En **Nodo → Sistema → Red**, se edita `vmbr0` y se activa **Consciente de VLAN** (VLAN aware). Se aplica la configuración de red y se reinicia el bridge.

### 2. VM de pfSense

- Máquina creada con la ISO de pfSense CE (tipo Other OS, 2 núcleos, 2 GB de RAM, disco de unos 8-20 GB).
- Una interfaz hacia la WAN y otra interfaz **sin VLAN tag** (trunk), que recibe todas las VLAN.
- En pfSense, en **Interfaces → Assignments → VLANs**, se crea cada VLAN sobre la interfaz trunk y se asigna como interfaz con su propia IP de puerta de enlace.

### 3. Asignación de VLAN a las máquinas

Cada VM recibe su VLAN Tag en la tarjeta de red virtual de Proxmox:

| Máquina | VLAN |
|---|---|
| Controlador de dominio, TrueNAS, docker01 | 10 (Servidores) |
| Cliente Windows 11 | 20 (Clientes) |

## Reglas de firewall

| Interfaz | Regla | Comentario |
|---|---|---|
| WAN | Permite UDP 1194 hacia la dirección WAN | Solo la VPN. Creada por el asistente de OpenVPN |
| LAN | Permite todo desde LAN subnets (reglas por defecto) | Aceptable porque es la red de gestión |
| CLIENTES_INTER | Permite TCP/UDP hacia SERVIDORES_INTER solo en el alias `Puertos_AD` | Mínimo privilegio |
| CLIENTES_INTER | Permite salida a `! Redes_privadas` | Internet sí, otras redes privadas no |
| SERVIDORES_INTER | Permite desde LAN subnets y desde su propia subred | Pendiente de endurecer |
| OpenVPN | Permite todo | Pendiente de restringir |

## Cómo compruebo que funciona

1. Desde el cliente Windows 11 (VLAN 20), el inicio de sesión de dominio funciona contra el controlador de dominio.
2. Desde el cliente, no hay acceso a la red de gestión ni a otras redes privadas fuera de lo permitido.
3. Prueba negativa: un puerto que no está en `Puertos_AD` se bloquea.

## Mejoras pendientes

- Activar el registro (log) de la regla de WAN y limitar las conexiones nuevas por origen.
- Sustituir la salida abierta de SERVIDORES_INTER por reglas concretas (actualizaciones, NTP).
- Restringir la interfaz OpenVPN a los destinos y puertos necesarios y activar su registro.
- Instalar pfBlockerNG.
- IDS/IPS con Suricata.

## Problemas encontrados

Ver [incidencias](incidencias.md).
