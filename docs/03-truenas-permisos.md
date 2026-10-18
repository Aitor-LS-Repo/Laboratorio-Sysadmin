# TrueNAS, recursos compartidos y permisos por grupo

Estado: `operativo`
Última revisión: `<AAAA-MM-DD>`

## Objetivo

Almacenamiento en red integrado en el dominio: cada usuario tiene su carpeta privada, hay una carpeta pública y los permisos se gestionan con grupos de AD.

## Dónde vive

| Dato | Valor |
|---|---|
| Máquina | VM TrueNAS en Proxmox |
| VLAN | 10 (Servidores) |
| Integración | Unido al dominio `lechus.priv` |
| Datasets | `NAS/Archivos` y `NAS/Publico`, cada uno compartido por SMB |

## Configuración

### 1. Unión al dominio

TrueNAS unido a Active Directory. Se comprueba en **Directory Services** que el estado es correcto y que los grupos de AD son visibles.

### 2. Estructura

```
NAS
├── Archivos          (share SMB)
│   ├── usuario1/     (solo usuario1)
│   └── usuario2/     (solo usuario2)
└── Publico           (share SMB)
```

Se usan datasets separados en lugar de subcarpetas para poder aplicar permisos y políticas de forma independiente.

### 3. Permisos de la raíz de `Archivos`

ACL de tipo NFSv4. Para el grupo `usuarios del dominio`, en la raíz:

- **Permitido:** Read Data, Execute, Read Attributes, Read Named Attributes, Read ACL, Synchronize.
- **No permitido:** Write Data, Append Data, Delete, Delete Children, Write Attributes, Write ACL, Write Owner.
- Herencia: **No Inherit**, para que este permiso solo de lectura no se propague a las carpetas de usuario.

Así cualquier usuario ve la lista de carpetas, pero solo entra en la suya.

### 4. Permisos de cada carpeta de usuario

- Se desactiva la herencia (Strip ACL) y se añade una ACE **Allow, Full Control** únicamente para ese usuario.

### 5. Carpeta personal automática

En el perfil del usuario en AD: unidad `G:` hacia `\\<servidor-nas>\Archivos\%username%`.
## Decisiones de diseño

### Uso de datasets independientes

Las carpetas personales y los recursos públicos se almacenan en datasets separados.

Motivos:
- Aplicar ACL independientes.
- Implementar cuotas en el futuro.
- Crear snapshots por área.
- Simplificar restauraciones.

## Validación

### Pruebas realizadas
- Acceso a carpeta personal autorizada.
- Denegación de acceso a carpetas de otros usuarios.
- Enumeración correcta de recursos compartidos.
- Protección de la raíz del recurso compartido.
- Resolución de identidades desde Active Directory..

## Mejoras pendientes

- Snapshots programados en TrueNAS.
- Cuotas por usuario.
- Backup de las VMs de Proxmox hacia este NAS y restauración comprobada.

## Problemas encontrados

Ver [incidencias](incidencias.md), puntos 1, 2 y 3.
