# VPN de administradores: OpenVPN con LDAPS

Estado: `operativo`
Última revisión: `<AAAA-MM-DD>`

## Objetivo

Acceso remoto seguro a la red de administración, autenticando a los usuarios contra Active Directory y permitiendo la entrada solo a los miembros de un grupo concreto.

## Dónde vive

| Dato | Valor |
|---|---|
| Servicio | OpenVPN en pfSense |
| Protocolo y puerto | UDP 1194 |
| Servidor de autenticación | `AD-Autenticacion` (LDAP, contra el controlador de dominio) |
| Grupo autorizado | `VPN_Admins` |
| Red del túnel | `<rango>` |

## Requisitos previos

- Dominio funcionando y cuenta de servicio `pfssense` en AD.
- CA interna y certificado LDAPS en el controlador de dominio.
- Grupo de seguridad `VPN_Admins` con los usuarios autorizados como miembros.

## Configuración

### 1. Servidor de autenticación LDAP en pfSense

En **System → User Manager → Authentication Servers**:

| Campo | Valor |
|---|---|
| Hostname | `<nombre del controlador de dominio>` (debe coincidir con el certificado) |
| Puerto / transporte | 636, SSL/TLS cifrado |
| CA | CA interna importada en pfSense |
| Base DN | `DC=lechus,DC=priv` |
| Contenedores de autenticación | `OU=usuarios;DC=lechus;DC=priv` |
| Credenciales de enlace | DN de `pfssense` y su contraseña |
| Atributo de nombre de usuario | `samAccountName` |
| Atributo de nombre de grupo | `cn` |
| Atributo de miembro de grupo | `memberOf` |
| Consulta extendida | activada: `memberOf=CN=VPN_Admins,DC=lechus,DC=priv` |

La consulta extendida es la que restringe el acceso al grupo. El filtro va en su campo, no en Base DN.

### 2. Servidor OpenVPN

Creado con el asistente (**VPN → OpenVPN → Wizards**), usando `AD-Autenticacion` como método de autenticación y la CA interna para los certificados. El asistente crea la regla de WAN para UDP 1194.

### 3. Cliente

Perfil `.ovpn` exportado con la utilidad de exportación de clientes. No se sube al repositorio.

## Reglas de firewall

| Interfaz | Regla | Motivo |
|---|---|---|
| WAN | UDP 1194 hacia la dirección WAN | Entrada de la VPN |
| OpenVPN | Permite todo (pendiente de endurecer) | Acceso de administradores |

## Cómo compruebo que funciona

1. En **Diagnostics → Authentication**, con un usuario de `VPN_Admins`: autenticación correcta.
2. Con un usuario de dominio que no está en el grupo: rechazado.
3. Conexión real con el cliente OpenVPN y acceso a la VLAN de servidores.

## Mejoras pendientes

- Registro (log) de la regla de WAN y limitación de conexiones por origen.
- Restringir la regla de OpenVPN a servidores y puertos de gestión.
- WireGuard como VPN secundaria.
- RADIUS con NPS y MFA.

## Problemas encontrados

Ver [incidencias](incidencias.md), puntos 4 y 5.
