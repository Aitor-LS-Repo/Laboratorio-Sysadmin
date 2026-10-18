# Seguridad de equipos: Microsoft LAPS y BitLocker

Estado: `operativo` | Última revisión: `2026-10-18`

---

# Objetivo

Implementar medidas de seguridad para estaciones de trabajo unidas al dominio centradas en:

- Protección de credenciales locales.
- Protección de la información almacenada.
- Gestión centralizada desde Active Directory.
- Reducción de riesgos de escalada lateral.

La solución se basa en:

- Microsoft LAPS para la gestión de contraseñas administrativas locales.
- BitLocker para el cifrado completo del disco.

---

# Tecnologías utilizadas

- Active Directory
- Group Policy (GPO)
- Microsoft LAPS
- BitLocker
- TPM 2.0
- Windows Server
- PowerShell
- Proxmox VE

---

# Arquitectura

```text
Active Directory
        │
        ├── Microsoft LAPS
        │        │
        │        └── Contraseñas locales rotatorias
        │
        └── BitLocker
                 │
                 └── Claves de recuperación
                     almacenadas en AD
```

---

# Microsoft LAPS

## Objetivo

Eliminar el uso de contraseñas locales compartidas y garantizar que cada equipo disponga de una contraseña administrativa única.

---

## Diseño

Cada estación de trabajo mantiene su propia contraseña local.

La contraseña:

- Es diferente en cada equipo.
- Se genera automáticamente.
- Se rota periódicamente.
- Se almacena en Active Directory.
- Solo puede ser consultada por personal autorizado.

---

## Configuración aplicada

### Almacenamiento

```text
Active Directory
```

---

### Requisitos

- Esquema de Active Directory ampliado mediante:

```powershell
Update-LapsADSchema
```

- Permisos delegados para que los equipos actualicen su propia contraseña.

---

### Política aplicada

| Parámetro | Valor |
|------------|---------|
| Destino de copia | Active Directory |
| Cifrado de contraseña | Activado |
| Longitud mínima | 12 caracteres |
| Complejidad | Activada |
| Rotación | 30 días |

---

## Beneficios

- Eliminación de credenciales compartidas.
- Reducción de movimientos laterales.
- Gestión centralizada.
- Rotación automática de credenciales.
- Auditoría del acceso administrativo.

---

## Validación

Consulta de contraseña:

```powershell
Get-LapsADPassword -Identity EQUIPO
```

Resultado esperado:

✅ Contraseña recuperada desde Active Directory.

✅ Inicio de sesión local funcional.

---

# BitLocker

## Objetivo

Proteger los equipos frente a:

- Robo.
- Pérdida.
- Acceso físico no autorizado.

mediante cifrado completo de disco.

---

## Diseño

La infraestructura utiliza:

```text
BitLocker + TPM
```

sin PIN adicional.

Las claves de recuperación son almacenadas automáticamente en Active Directory.

---

## Configuración aplicada

### TPM

Configuración:

```text
TPM 2.0
```

Implementado mediante:

```text
Proxmox
→ TPM Virtual
→ OVMF (UEFI)
→ Máquina Q35
```

---

### Protección de arranque

| Parámetro | Valor |
|------------|---------|
| TPM | Permitido |
| PIN | No utilizado |
| Clave USB | No utilizada |
| Modo | TPM Only |

---

### Recuperación

Configuración aplicada:

- Almacenamiento obligatorio en AD.
- Contraseña de recuperación de 48 dígitos.
- Clave de recuperación de 256 bits.
- Paquete de recuperación almacenado en Active Directory.

---

## Beneficios

- Protección de la información corporativa.
- Recuperación centralizada.
- Gestión desde Active Directory.
- Transparencia para el usuario.
- Cumplimiento de buenas prácticas de seguridad.

---

## Validación

### Estado de BitLocker

```cmd
manage-bde -status
```

Resultado esperado:

- Unidad cifrada.
- TPM configurado como protector.

---

### Recuperación

Comprobación mediante:

```text
Usuarios y equipos de Active Directory
→ Equipo
→ BitLocker Recovery
```

Resultado esperado:

✅ Claves de recuperación almacenadas correctamente.

---

### Aplicación de GPO

```cmd
gpresult /h informe.html /scope:computer
```

Resultado esperado:

✅ Políticas correctamente aplicadas.

---

# Riesgos mitigados

## Microsoft LAPS

- Contraseñas locales compartidas.
- Escalada lateral entre equipos.
- Accesos persistentes mediante cuentas locales.

---

## BitLocker

- Robo de portátiles.
- Extracción de discos.
- Acceso físico a equipos apagados.

---

# Integración con la infraestructura

Esta solución se integra con:

- Active Directory.
- GPO corporativas.
- VLAN de clientes.
- VPN administrativa.
- Administración centralizada del dominio.

---

# Lecciones aprendidas

- LAPS requiere preparar previamente el esquema de Active Directory.
- El cifrado BitLocker debe validarse antes de su despliegue masivo.
- Las claves de recuperación deben comprobarse periódicamente.
- El TPM virtual de Proxmox permite reproducir escenarios empresariales en entornos virtualizados.

---

# Mejoras previstas

- Automatización de activación mediante PowerShell.
- Auditoría periódica de equipos cifrados.
- Supervisión de cumplimiento mediante scripts.
- Informes centralizados sobre estado de BitLocker y LAPS.
``

## Problemas encontrados

Ver [incidencias](incidencias.md), punto 6.
