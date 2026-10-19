# Proxy inverso con Nginx Proxy Manager

Estado: `operativo` | Última revisión: `2026-10-18`

---

# Objetivo

Centralizar el acceso a los servicios web internos mediante nombres DNS y conexiones HTTPS válidas, utilizando certificados emitidos por la Autoridad de Certificación interna del dominio.

La solución permite acceder a los servicios corporativos utilizando nombres fáciles de recordar, evitando el uso de direcciones IP y puertos específicos.

---
![Diagrama de red](img/nginx.png)


# Dónde vive

| Dato | Valor |
|--------|--------|
| Servidor | docker01 |
| Sistema operativo | Ubuntu Server |
| Plataforma | Docker |
| Servicio | Nginx Proxy Manager |
| VLAN | 10 (Servidores) |
| DNS | Active Directory |
| Certificados | CA interna (AD CS) |

---

# Objetivos de diseño

## Acceso mediante nombres DNS

Antes:

```text
https://192.168.10.102:9443
http://192.168.10.102:3001
```

Después:

```text
https://portainer.lechus.priv
https://uptime.lechus.priv
```

---

## Cifrado centralizado

Todos los servicios utilizan certificados emitidos por la Autoridad de Certificación interna.

Esto permite:

- Utilizar HTTPS en todos los servicios.
- Evitar advertencias de seguridad.
- Simplificar la administración de certificados.
- Mantener una configuración homogénea.

---

## Punto único de publicación

Todos los servicios web internos se publican mediante un único proxy inverso.

Esto facilita:

- La incorporación de nuevos servicios.
- La gestión de certificados.
- La aplicación de configuraciones de seguridad comunes.
- La administración de accesos.

---

# Arquitectura

```text
                   Active Directory
                          │
        ┌─────────────────┴─────────────────┐
        │                                   │

        ▼                                   ▼

       DNS                           CA interna

        │                                   │
        └─────────────────┬─────────────────┘
                          │

                Nginx Proxy Manager

                          │

          ┌───────────────┴───────────────┐
          │                               │

          ▼                               ▼

 portainer.lechus.priv         uptime.lechus.priv

          │                               │

          ▼                               ▼

 Portainer (9443)              Uptime Kuma (3001)
```

---

# Resolución DNS

Se crearon registros DNS dentro de la zona:

```text
lechus.priv
```

## Registros configurados

| Nombre DNS | Destino |
|------------|----------|
| portainer.lechus.priv | docker01 |
| uptime.lechus.priv | docker01 |

Todos los registros resuelven a:

```text
192.168.10.102
```

---

# Certificados

## Plantilla personalizada

Se creó una plantilla específica para servicios web:

```text
WebServer-Interno
```

Características:

- Autenticación de servidor.
- Nombre del sujeto definido por el solicitante.
- Exportación de la clave privada habilitada.
- Permisos de inscripción para administradores del dominio.

---

## Certificado emitido

Se emitió un certificado válido para todos los subdominios del dominio interno.

Nombre común:

```text
Dominio completo lechus.priv
```

Nombres alternativos configurados:

```text
portainer.lechus.priv
uptime.lechus.priv
```

---

## Exportación

El certificado se exportó en formato:

```text
PFX
```

incluyendo:

- Certificado público.
- Clave privada.
- Cadena de confianza.

Posteriormente se importó en Nginx Proxy Manager.

---

# Configuración de Nginx Proxy Manager

## Puertos publicados

| Puerto | Función |
|----------|----------|
| 80 | HTTP |
| 81 | Administración |
| 443 | HTTPS |

---

# Servicio: Portainer

## Nombre DNS

```text
portainer.lechus.priv
```

## Destino

```text
192.168.10.102
```

## Puerto

```text
9443
```

## Protocolo

```text
HTTPS
```

## Opciones habilitadas

```text
Block Common Exploits
WebSockets Support
Force SSL
HTTP/2 Support
HSTS
```

---

# Servicio: Uptime Kuma

## Nombre DNS

```text
uptime.lechus.priv
```

## Destino

```text
192.168.10.102
```

## Puerto

```text
3001
```

## Protocolo

```text
HTTP
```

## Opciones habilitadas

```text
Block Common Exploits
Force SSL
HTTP/2 Support
HSTS
```

---

# Validación

## Resolución DNS

Comprobación realizada mediante:

```powershell
nslookup portainer.lechus.priv
nslookup uptime.lechus.priv
```

Resultado esperado:

```text
192.168.10.102
```

---

## Certificados

Se verificó que:

- El certificado fue emitido por la Autoridad de Certificación interna.
- El nombre DNS coincide con el certificado presentado.
- El navegador muestra una conexión segura.
- No aparecen advertencias de confianza.

---

## Acceso a Portainer

Dirección:

```text
https://portainer.lechus.priv
```

Resultado esperado:

- Acceso correcto a la interfaz.
- Certificado válido.
- Conexión HTTPS funcional.

---

## Acceso a Uptime Kuma

Dirección:

```text
https://uptime.lechus.priv
```

Resultado esperado:

- Acceso correcto a la interfaz.
- Certificado válido.
- Conexión HTTPS funcional.

---

# Beneficios obtenidos

- Eliminación del acceso mediante direcciones IP y puertos.
- Uso de nombres DNS corporativos.
- Centralización de certificados TLS.
- Simplificación de la publicación de servicios.
- Integración con Active Directory.
- Integración con la Autoridad de Certificación interna.
- Mejora de la experiencia de administración.
