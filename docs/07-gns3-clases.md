# Laboratorio de GNS3 para clases

Estado: `en pruebas`
Última revisión: `<AAAA-MM-DD>`

## Objetivo

Dar a cada alumno una máquina con GNS3 donde hacer sus laboratorios de redes y sistemas, aislada del resto de mi infraestructura. Máximo uno o dos alumnos a la vez.

## Diseño

- Una **VM de GNS3 por alumno**, clonada desde una plantilla, cada una en su propia VLAN aislada.
- Acceso por VPN con un grupo de AD distinto al de administradores (`VPN_Alumnos`, pendiente de crear).
- Si una VM se estropea, se borra y se vuelve a clonar.
- Imágenes libres o de evaluación; no se distribuyen imágenes con licencia.

## Dónde vive

| Dato | Valor |
|---|---|
| VM plantilla | `GNS3-PLANTILLA` (ID 112) |
| Sistema | Xubuntu 26.04 LTS |
| Recursos | 4 núcleos (CPU tipo `host`), 10 GB de RAM, disco de 100 GB |
| Máquina | q35, BIOS SeaBIOS, VirtIO SCSI single |
| GNS3 | 2.2.59, instalado desde el PPA oficial |

## Configuración

### 1. Virtualización anidada

- En el nodo Proxmox: `cat /sys/module/kvm_intel/parameters/nested` debe devolver `Y`. Si no, `options kvm-intel nested=Y` en `/etc/modprobe.d/kvm-intel.conf` y recargar el módulo con las VMs apagadas.
- En la VM: tipo de CPU `host` y apagado completo (no reinicio) tras el cambio.
- Dentro de la VM: `grep -c vmx /proc/cpuinfo` mayor que 0, `/dev/kvm` presente y `kvm-ok` correcto.
- El usuario debe pertenecer a los grupos `kvm`, `ubridge` y `wireshark`.

### 2. Instalación de GNS3

```bash
add-apt-repository -y ppa:gns3/ppa
apt update
apt install -y gns3-gui gns3-server
```

Se responde **Sí** a las preguntas sobre ejecutar `ubridge` y capturar paquetes sin ser root. No se instala `gns3-iou`.

### 3. Consola de las máquinas

Visor VNC instalado: `apt install -y tigervnc-viewer`. El tipo de consola de las plantillas con interfaz gráfica es `vnc`.

### 4. Plantilla de Windows 11

- Plantilla QEMU con disco qcow2 de 64 GB, 4 GB de RAM, disco SATA y tarjeta de red e1000.
- Opción `-cpu host` para aprovechar KVM.
- Comprobación de TPM y Secure Boot saltada con las claves `LabConfig` del registro durante la instalación, si la versión de GNS3 no ofrece TPM.
- Tras instalar, se marca *Use as a linked base VM* para que cada nodo guarde solo sus cambios.

## Cómo compruebo que funciona

1. Un proyecto con dos VPCS y un switch hace ping entre ellos.
2. El nodo de Windows 11 arranca con aceleración KVM.

## Pendiente

- Cargar imágenes libres: FRRouting, VyOS, MikroTik CHR, contenedores Linux.
- Crear el grupo `VPN_Alumnos` y la VLAN aislada por alumno con reglas en pfSense.
- Convertir la VM en plantilla y clonarla por alumno.
- Medir el consumo real de RAM con dos nodos Windows encendidos.
- Resolver la parte fiscal y de facturación de las clases.

## Problemas encontrados

Ver [incidencias](incidencias.md), puntos 7 a 9.
