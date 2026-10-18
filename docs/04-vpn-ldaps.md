# Infraestructura VPN corporativa

Estado: `operativo`

Última revisión: `2026-10-18`

---

# Objetivo

Proporcionar acceso remoto seguro a la infraestructura corporativa mediante dos servicios VPN independientes:

- VPN de administración.
- VPN de usuarios.

La separación permite aplicar distintos niveles de acceso, reducir la superficie de ataque y mantener aisladas las tareas administrativas de las actividades de usuario.

---

# Arquitectura general

```text
                          Internet
                              │
                              ▼
                          pfSense
                              │
         ┌────────────────────┴────────────────────┐
         │                                         │
         ▼                                         ▼

 VPN Administradores                        VPN Usuarios
     UDP 1194                                 UDP 1195

         │                                         │

         ▼                                         ▼

   Grupo VPN_Admins                      Grupo VPN_usuarios

         │                                         │

         ▼                                         ▼

 Recursos de gestión                    Recursos corporativos
```

---

# Objetivos de la segmentación

## VPN Administradores

Diseñada para personal técnico encargado de:

- Administración de servidores.
- Gestión de Active Directory.
- Administración de TrueNAS.
- Gestión de Proxmox.
- Mantenimiento de infraestructura.

---

## VPN Usuarios

Diseñada para usuarios finales que necesitan:

- Acceso remoto a recursos corporativos.
- Acceso a almacenamiento compartido.
- Acceso a aplicaciones internas.
- Trabajo remoto seguro.

Los usuarios no tienen acceso a sistemas de administración.

---

# Tecnologías utilizadas

- pfSense CE
- OpenVPN
- Active Directory
- LDAP
- LDAPS
- Active Directory Certificate Services
- Certificados X.509
- VLANs
- Grupos de seguridad de Active Directory

---

# Autenticación centralizada

Ambas VPN utilizan Active Directory como sistema centralizado de autenticación.

La validación se realiza mediante:

```text
LDAPS (LDAP sobre SSL/TLS)
```

Esto evita el envío de credenciales sin cifrar y elimina la necesidad de mantener usuarios locales en el firewall.

---

# Servidor LDAP de autenticación

## Configuración

| Parámetro | Valor |
|------------|---------|
| Tipo | LDAP |
| Servidor | `dc01.lechus.priv` |
| Puerto | `636` |
| Transporte | `SSL/TLS Encrypted` |
| Versión LDAP | `3` |
| CA utilizada | `CA-2` |
| Base DN | `DC=lechus,DC=priv` |
| Contenedor de autenticación | `OU=usuarios,DC=lechus,DC=priv` |
| Tiempo de espera | `25 segundos` |

---

# Certificados

Para permitir LDAPS:

- Se desplegó una CA interna.
- El controlador de dominio dispone de un certificado de servidor.
- La CA raíz se importó en pfSense.
- pfSense valida el certificado antes de iniciar la comunicación LDAP.

---

# Cuenta de servicio

Las consultas LDAP se realizan mediante una cuenta de servicio dedicada:

```text
pfssense
```

Características:

- Sin privilegios administrativos.
- Contraseña independiente.
- Utilizada únicamente para consultas LDAP.
- Acceso mínimo necesario al directorio.

---

# VPN de administración

## Objetivo

Permitir la gestión remota de la infraestructura.

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

Solo los usuarios miembros del grupo:

```text
VPN_Admins
```

pueden establecer una conexión válida.

---

## Recursos accesibles

- Active Directory
- DNS
- TrueNAS
- Proxmox
- Servidores de la VLAN 10
- Sistemas de administración

---

# VPN de usuarios

## Objetivo

Permitir acceso remoto a recursos corporativos sin exponer sistemas de administración.

## Configuración

| Parámetro | Valor |
|------------|---------|
| Servicio | OpenVPN |
| Puerto | UDP 1195 |
| Red VPN | 10.10.9.0/24 |
| Autenticación | LDAPS |
| Grupo autorizado | VPN_usuarios |

---

## Redes publicadas

```text
192.168.10.0/24
192.168.20.0/24
```

---

## Algoritmos de cifrado

Configurados:

```text
AES-256-GCM
AES-128-GCM
CHACHA20-POLY1305
```

---

## Integridad

```text
SHA256
```

---

## Control de acceso

Consulta LDAP:

```text
memberOf=CN=VPN_usuarios,DC=lechus,DC=priv
```

Solo los usuarios pertenecientes al grupo:

```text
VPN_usuarios
```

pueden autenticarse.

---

# Flujo de autenticación

```text
Usuario VPN
      │
      ▼
OpenVPN
      │
      ▼
pfSense
      │
      ▼
LDAPS (TCP 636)
      │
      ▼
Active Directory
      │
      ▼
Verificación de pertenencia a grupo
      │
      ▼
Acceso permitido o denegado
```

---

# Medidas de seguridad aplicadas

## Active Directory

- Gestión centralizada de usuarios.
- Gestión centralizada de grupos.

## LDAP seguro

- Comunicación cifrada mediante LDAPS.
- Validación de certificados.

## Autorización

- Control de acceso basado en grupos.
- Separación entre perfiles técnicos y usuarios finales.

## pfSense

- Firewall perimetral.
- Reglas específicas para OpenVPN.
- Segmentación mediante VLANs.

---

# Validaciones realizadas

## VPN Administradores

✅ Usuario miembro de VPN_Admins puede autenticarse.

✅ Usuario fuera de VPN_Admins es rechazado.

✅ Acceso a recursos de administración.

---

## VPN Usuarios

✅ Usuario miembro de VPN_usuarios puede autenticarse.

✅ Usuario fuera de VPN_usuarios es rechazado.

✅ Obtención correcta de dirección IP del túnel.

✅ Acceso a recursos corporativos autorizados.

---

# Beneficios obtenidos

- Autenticación centralizada.
- Eliminación de usuarios locales en pfSense.
- Control mediante grupos de Active Directory.
- Separación de perfiles administrativos y usuarios finales.
- Menor superficie de ataque.
- Comunicación cifrada extremo a extremo.
- Gestión simplificada de accesos remotos.

---

# Mejoras previstas

- WireGuard como VPN adicional.
- Integración NPS/RADIUS.
- MFA para administradores.
- Restricción granular de recursos por grupo.
- Registro avanzado de accesos VPN.
- Alertas y monitorización mediante Uptime Kuma.
## Problemas encontrados

Ver [incidencias](incidencias.md), puntos 4 y 5.
