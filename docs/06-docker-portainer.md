# Contenedores: Docker, Portainer y Uptime Kuma

Estado: `operativo`
Última revisión: `<AAAA-MM-DD>`

## Objetivo

Plataforma de contenedores para desplegar servicios internos de forma reproducible, con panel de gestión y monitorización con alertas.

## Dónde vive

| Dato | Valor |
|---|---|
| Máquina | VM `docker01` en Proxmox, Ubuntu Server |
| VLAN | 10 (Servidores) |
| IP | `<ip>` |
| Servicios | Portainer (9443), Uptime Kuma (3001) |

## Configuración

1. VM con Ubuntu Server, IP fija y OpenSSH.
2. Docker Engine y Compose instalados desde el repositorio oficial de Docker.
3. Stack definido en un `docker-compose.yml` (ver carpeta `docker/`).
4. Portainer: usuario administrador creado en el primer acceso.
5. Uptime Kuma: monitores sobre pfSense, controlador de dominio, TrueNAS, puerto LDAPS y la web de Proxmox, con notificaciones por `<canal>`.

## Cómo compruebo que funciona

1. `docker ps` muestra los contenedores en ejecución.
2. Entro en Portainer y en Uptime Kuma desde un equipo autorizado.
3. Apago a propósito un servicio monitorizado y compruebo que llega la alerta.

## Seguridad

- Portainer monta el socket de Docker, lo que da control total sobre el host. Solo es accesible desde VPN o LAN.
- Los puertos de administración no se abren a la VLAN de clientes.

## Pendiente

- Nginx Proxy Manager con certificados de la CA interna y acceso por nombre.
- Contenedor de PowerShell para automatizaciones (sin el módulo ActiveDirectory, que es solo de Windows).

## Problemas encontrados

Ver [incidencias](incidencias.md).
