# Active Directory, DNS y DHCP

Estado: `operativo`
Última revisión: `<18/10/2026>`

## Objetivo

Dominio corporativo con identidad centralizada, resolución de nombres y direccionamiento, sobre el que se apoyan el resto de servicios (almacenamiento, VPN, cifrado).

## Dónde vive

| Dato | Valor |
|---|---|
| Servidor | VM Windows Server en Proxmox |
| Dominio | `lechus.priv` |
| IP del controlador de dominio | `192.168.10.2` (VLAN 10) |
| Roles | AD DS, DNS, DHCP |
| Certificados | CA interna (AD Certificate Services) |

## Estructura del directorio

```
lechus.priv
├── usuarios
│   ├── Administracion
│   ├── Administradores
│   ├── Comercial
│   └── IT
├── usuarios administrativos
├── Equipos_Nuevos
├── Servidores
└── Equipo lugar 1
```

Grupos de seguridad más relevantes:

| Grupo | Uso |
|---|---|
| `GS_IT`, `GS_Comercial`, `GS_Administracion` | Permisos de carpetas por departamento |
| `Publico` | Acceso a la carpeta pública |
| `VPN_Admins` | Usuarios autorizados a conectarse por VPN |
| `Admin Local` | Administración local de equipos |

Cuenta de servicio: `pfssense`, usada por pfSense para consultar el directorio. Contraseña que no caduca y sin privilegios de administración.

## Configuración

### 1. DNS y DHCP

- DNS integrado con AD, con registros dinámicos.
- DHCP gestionado por el servidor Windows con el ámbito `<ámbito>`.

### 2. Carpetas personales

En la pestaña **Perfil** de cada usuario se define la carpeta particular: unidad `G:` apuntando a `\\<servidor-nas>\Archivos\%username%`. La variable correcta es `%username%`.

### 3. LDAPS

Certificado emitido por la CA interna para el controlador de dominio, para que pfSense autentique por LDAPS en vez de LDAP sin cifrar.

## Integraciones
pfSense ↔ Active Directory
- La autenticación VPN no utiliza cuentas administrativas. A través de LDAP, se usan las cuentas de dominio.
- Se creó una cuenta de servicio dedicada con permisos mínimos de lectura para realizar consultas LDAP y LDAPS.

## Cómo compruebo que funciona

1. Un cliente Windows 11 se une al dominio e inicia sesión con un usuario de AD.
2. El usuario recibe su unidad `G:` con su carpeta personal.
3. `gpresult /r /scope:computer` muestra las GPO aplicadas al equipo.

## Mantenimiento

- Copias de seguridad: `<pendiente: backup de la VM desde Proxmox>`
- Segundo controlador de dominio: pendiente.
- Análisis de seguridad del dominio con PingCastle: pendiente.

## Problemas encontrados

Ver [incidencias](incidencias.md).
