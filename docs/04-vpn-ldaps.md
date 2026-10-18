# Autenticación LDAP/LDAPS para OpenVPN

Estado: `operativo` | Última revisión: `2026-10-18`

---

# Objetivo

Centralizar la autenticación de usuarios VPN en Active Directory, evitando la gestión de credenciales locales en pfSense y permitiendo controlar el acceso mediante grupos de seguridad del dominio.

La autenticación se realiza mediante LDAPS (LDAP sobre SSL/TLS), garantizando la confidencialidad de las credenciales durante el proceso de validación.

---

# Arquitectura

```text
Usuario VPN
      │
      ▼
OpenVPN (pfSense)
      │
      ▼
LDAPS (Puerto 636)
      │
      ▼
Active Directory
      │
      ▼
Grupo VPN_usuarios
```

---

# Tecnologías utilizadas

- Active Directory
- LDAP
- LDAPS
- Active Directory Certificate Services (AD CS)
- OpenVPN
- pfSense
- Certificados X.509

---

# Componentes

| Elemento | Función |
|-----------|----------|
| Active Directory | Almacén centralizado de usuarios y grupos |
| Cuenta de servicio pfSense | Consulta LDAP sin privilegios administrativos |
| CA interna | Emisión y validación de certificados |
| LDAPS | Canal cifrado entre pfSense y Active Directory |
| Grupo VPN_usuarios | Control de acceso a la VPN |
| OpenVPN | Servicio de acceso remoto |

---

# Configuración del servidor LDAP

## Parámetros principales

| Parámetro | Valor |
|------------|---------|
| Tipo | LDAP |
| Servidor | `dc01.lechus.priv` |
| Puerto | `636` |
| Transporte | `SSL/TLS Encrypted` |
| Versión LDAP | `v3` |
| Autoridad certificadora | `CA-2` |
| Base DN | `DC=lechus,DC=priv` |
| Contenedor de autenticación | `OU=usuarios,DC=lechus,DC=priv` |
| Tiempo de espera | `25 segundos` |

---

# Validación del certificado

La comunicación LDAP utiliza cifrado SSL/TLS.

Para ello:

1. El controlador de dominio dispone de un certificado emitido por la CA interna.
2. La CA raíz se importa en pfSense.
3. pfSense valida la identidad del controlador de dominio antes de establecer la sesión LDAP.

Esto evita ataques de interceptación y garantiza la autenticidad del servidor LDAP.

---

# Cuenta de servicio

La autenticación LDAP se realiza mediante una cuenta de servicio dedicada:

```text
pfssense
```

Características:

- Sin privilegios administrativos.
- Utilizada únicamente para consultas LDAP.
- Contraseña independiente de cuentas personales.
- Acceso limitado al directorio.

---

# Control de acceso mediante grupos

La autenticación y la autorización están separadas.

## Autenticación

Active Directory valida:

- Usuario
- Contraseña

## Autorización

pfSense verifica que el usuario pertenezca al grupo:

```text
VPN_usuarios
```

mediante la consulta LDAP:

```text
memberOf=CN=VPN_usuarios,DC=lechus,DC=priv
```

Solo los usuarios pertenecientes a este grupo pueden establecer una conexión VPN válida.

---

# Parámetros LDAP utilizados

| Parámetro | Valor |
|------------|---------|
| Username Attribute | `samAccountName` |
| Group Attribute | `cn` |
| Group Membership Attribute | `memberOf` |
| Search Scope | Entire Subtree |

Estos atributos permiten localizar usuarios y comprobar su pertenencia a grupos de Active Directory.

---

# Flujo de autenticación

```text
1. Usuario introduce credenciales VPN.

2. OpenVPN envía la solicitud a pfSense.

3. pfSense consulta Active Directory mediante LDAPS.

4. Active Directory valida usuario y contraseña.

5. pfSense verifica la pertenencia al grupo VPN_usuarios.

6. Si ambas comprobaciones son correctas:
      ► Se establece la VPN.

7. Si alguna comprobación falla:
      ► Acceso denegado.
```

---

# Ventajas del diseño

## Gestión centralizada

Los usuarios se administran desde Active Directory.

No existen credenciales duplicadas en pfSense.

---

## Control por grupos

Los permisos VPN se gestionan añadiendo o eliminando usuarios de:

```text
VPN_usuarios
```

sin modificar la configuración de OpenVPN.

---

## Comunicación cifrada

Las credenciales nunca viajan mediante LDAP sin cifrar.

Toda la comunicación utiliza:

```text
LDAPS (TCP 636)
```

---

## Menor superficie de ataque

Solo los usuarios autorizados mediante grupos del dominio pueden autenticarse.

---

# Validación

## Validación positiva

✅ Usuario miembro de:

```text
VPN_usuarios
```

Conexión VPN establecida correctamente.

---

## Validación negativa

✅ Usuario válido del dominio pero fuera del grupo:

```text
VPN_usuarios
```

Acceso rechazado.

---

## Validación del servicio LDAP

En pfSense:

```text
Diagnostics
→ Authentication
```

Resultado:

✅ Usuario autenticado correctamente.

---

# Lecciones aprendidas

Durante la implantación fue necesario:

- Implementar una CA interna.
- Emitir certificados válidos para el controlador de dominio.
- Importar la CA raíz en pfSense.
- Configurar correctamente los atributos LDAP.
- Aplicar filtrado por grupos mediante consultas extendidas.

---

# Mejoras previstas

- WireGuard como VPN adicional.
- Integración con NPS/RADIUS.
- Autenticación multifactor (MFA).
- Restricción de acceso por perfiles de usuario.
- Registro detallado de accesos.
- Limitación de intentos de conexión.

## Problemas encontrados

Ver [incidencias](incidencias.md), puntos 4 y 5.
