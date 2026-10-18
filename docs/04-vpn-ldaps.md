# Infraestructura VPN corporativa

Estado: `operativo` | Última revisión: `18/10/2026`

---

# Objetivo

Proporcionar acceso remoto seguro a la infraestructura mediante dos servicios VPN independientes:

- VPN de administración.
- VPN de usuarios.

Ambas VPN utilizan autenticación centralizada mediante Active Directory y LDAPS, pero aplican distintos niveles de acceso según el perfil del usuario.

Este diseño permite separar las tareas de administración de sistemas del acceso remoto de usuarios finales, aplicando el principio de mínimo privilegio.

---

# Arquitectura general

```text
                          Internet
                              │
                              ▼
                          pfSense
                              │
      ┌───────────────────────┴────────────────────────┐
      │                                                │
      ▼                                                ▼
 VPN Administradores                            VPN Usuarios
     UDP 1194                                      UDP 1195
      │                                                │
      ▼                                                ▼
 Grupo VPN_Admins                            Grupo VPN_usuarios
      │                                                │
      ▼                                                ▼
 Recursos de gestión                         Recursos corporativos
```

---

# Tecnologías utilizadas

- pfSense CE
- OpenVPN
- Active Directory
- LDAP
- LDAPS
- Active Directory Certificate Services (AD CS)
- Certificados X.509
- VLANs
- Grupos de seguridad de Active Directory

---

# Integración con Active Directory

La autenticación de ambas VPN está centralizada en Active Directory.

pfSense no almacena usuarios locales para el acceso remoto.

Todas las validaciones se realizan mediante:

```text
LDAPS (LDAP sobre SSL/TLS)
```

contra el controlador de dominio.

---

# Configuración LDAP

## Servidor LDAP

| Parámetro | Valor |
|------------|---------|
| Tipo | LDAP |
| Servidor | `dc01.lechus.priv` |
| Puerto | `636` |
| Transporte | `SSL/TLS Encrypted` |
| Versión LDAP | `3` |
| Base DN | `DC=lechus,DC=priv` |
| Contenedor de autenticación | `OU=usuarios,DC=lechus,DC=priv` |
| Tiempo de espera | `25 segundos` |

---

## Certificados

La autenticación se realiza mediante LDAPS.

Para ello:

- Existe una CA interna corporativa.
- El controlador de dominio dispone de un certificado válido.
- La CA raíz se ha importado en pfSense.
- pfSense valida el certificado antes de establecer la conexión LDAP.

Esto garantiza la confidencialidad de las credenciales y evita ataques de intermediario.

---

## Cuenta de servicio

Las consultas LDAP se realizan mediante una cuenta dedicada:

```text
pfssense
```

Características:

- Sin privilegios administrativos.
- Utilizada únicamente para consultas LDAP.
- Principio de mínimo privilegio.

---

# VPN de administración

## Objetivo

Permitir la administración remota de la infraestructura.

Está diseñada para tareas de:

- Gestión de Active Directory.
- Administración de TrueNAS.
- Administración de Proxmox.
- Administración de servicios internos.
- Soporte técnico avanzado.

---

## Configuración

| Parámetro | Valor |
|------------|---------|
| Servicio | OpenVPN |
| Puerto | UDP 1194 |
| Autenticación | LDAPS |
| Grupo autorizado | VPN_Admins |

---

## Control de acceso

Consulta LDAP:

```text
memberOf=CN=VPN_Admins,DC=lechus,DC=priv
```

Solo los usuarios pertenecientes al grupo:

```text
VPN_Admins
```

pueden autenticarse.

---

## Accesos permitidos

Regla de firewall:

```text
vpn_admin → SERVIDORES_INTER
```

Permite el acceso a:

- Active Directory.
- TrueNAS.
- Proxmox.
- Servidores de infraestructura.
- Servicios internos de administración.

---

# VPN de usuarios

## Objetivo

Permitir a los usuarios acceder a recursos corporativos desde ubicaciones externas sin exponer la infraestructura de administración.

---

## Configuración

| Parámetro | Valor |
|------------|---------|
| Servicio | OpenVPN |
| Puerto | UDP 1195 |
| Red del túnel | 10.10.9.0/24 |
| Autenticación | LDAPS |
| Grupo autorizado | VPN_usuarios |

---

## Redes publicadas

```text
192.168.10.0/24
192.168.20.0/24
```

---

## Control de acceso

Consulta LDAP:

```text
memberOf=CN=VPN_usuarios,DC=lechus,DC=priv
```

Solo los miembros del grupo:

```text
VPN_usuarios
```

pueden conectarse.

---

## Accesos permitidos

### Recursos compartidos SMB

Regla:

```text
vpn_usuario → SERVIDORES_INTER → TCP 445
```

Permite acceder a:

- Carpetas personales.
- Recursos compartidos de TrueNAS.
- Almacenamiento corporativo.

---

### Equipos de usuario

Regla:

```text
vpn_usuario → CLIENTES_INTER
```

Permite el acceso a los equipos de usuario situados en la VLAN de clientes.

---

## Cifrado

### Algoritmos permitidos

```text
AES-256-GCM
AES-128-GCM
CHACHA20-POLY1305
```

### Integridad

```text
SHA256
```

---

# Flujo de autenticación

```text
1. El usuario inicia la conexión VPN.

2. OpenVPN envía la autenticación a pfSense.

3. pfSense consulta Active Directory mediante LDAPS.

4. Active Directory valida usuario y contraseña.

5. pfSense comprueba la pertenencia al grupo correspondiente:
      - VPN_Admins
      - VPN_usuarios

6. Si la validación es correcta:
      ► Se establece el túnel VPN.

7. Si falla cualquiera de las comprobaciones:
      ► Acceso denegado.
```

---

# Seguridad implementada

## Gestión centralizada

Todos los usuarios se gestionan desde Active Directory.

---

## Comunicación cifrada

La autenticación se realiza mediante:

```text
LDAPS (TCP 636)
```

---

## Control basado en grupos

La autorización depende de grupos de seguridad del dominio.

---

## Segmentación de acceso

Administradores y usuarios utilizan servicios VPN distintos.

Esto evita que los usuarios finales tengan acceso a sistemas críticos de administración.

---

# Validación

## VPN Administradores

- Usuario miembro de `VPN_Admins` puede conectarse.
- Usuario fuera del grupo es rechazado.
- Acceso a servidores de infraestructura.

---

## VPN Usuarios

- Usuario miembro de `VPN_usuarios` puede conectarse.
- Usuario fuera del grupo es rechazado.
- Obtención correcta de IP del túnel.
- Acceso a recursos

## Problemas encontrados

Ver [incidencias](incidencias.md), puntos 4 y 5.
