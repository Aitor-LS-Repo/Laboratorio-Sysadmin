# Directivas de Grupo (GPO) implantadas

Estado: `operativo` | Última revisión: `18/10/2026`

## Objetivo

Aplicar configuraciones de seguridad, administración y automatización de forma centralizada sobre todos los equipos unidos al dominio.

Las directivas permiten mantener una configuración homogénea, reducir tareas manuales y aplicar medidas de seguridad de forma consistente en toda la infraestructura.

---

# GPO: AD_Equipos

## Objetivo

Reducir el riesgo de acceso no autorizado a equipos desatendidos.

## Configuración aplicada

### Protección de sesión

- Activación obligatoria del protector de pantalla.
- Solicitud de contraseña al reanudar la sesión.
- Bloqueo automático tras 300 segundos de inactividad.

## Beneficios

- Protección de sesiones abiertas.
- Reducción del riesgo de accesos no autorizados.
- Aplicación uniforme en todos los equipos del dominio.

## Validación

Comprobación mediante:

```cmd
gpresult /r
```

Resultado esperado:

- Protector de pantalla activo.
- Solicitud de credenciales al desbloquear.
- Tiempo de espera de 5 minutos.

---

# GPO: ADMIN USER

## Objetivo

Facilitar la administración centralizada de estaciones de trabajo y permitir la ejecución de herramientas de automatización basadas en PowerShell.

## Configuración aplicada

### PowerShell

- Ejecución de scripts habilitada.
- Política de ejecución configurada para permitir scripts.

### Administración local

Mediante Group Policy Preferences se agrega automáticamente:

```text
LECHUS\Admin Local
```

al grupo local:

```text
Administradores
```

de los equipos afectados por la directiva.

## Beneficios

- Administración homogénea de estaciones de trabajo.
- Eliminación de configuraciones manuales.
- Simplificación de tareas de soporte.
- Posibilidad de automatización mediante scripts.

## Validación

Verificar la pertenencia al grupo local:

```cmd
net localgroup Administradores
```

Resultado esperado:

```text
LECHUS\Admin Local
```

aparece como miembro del grupo local de administradores.

---

# GPO: BitLocker

## Objetivo

Proteger los equipos del dominio mediante cifrado de disco gestionado centralmente desde Active Directory.

## Configuración aplicada

### Cifrado de unidades

- BitLocker habilitado para unidades del sistema operativo.
- Uso de TPM como mecanismo principal de protección.

### Recuperación

Se almacena automáticamente en Active Directory:

- Contraseña de recuperación de 48 dígitos.
- Clave de recuperación de 256 bits.
- Paquete de claves de recuperación.

### Protección TPM

Configuración aplicada:

- Permitir TPM.
- Sin PIN de arranque.
- Sin clave USB de arranque.

## Beneficios

- Protección frente a robo o pérdida de dispositivos.
- Recuperación centralizada.
- Administración integrada con Active Directory.
- Experiencia transparente para el usuario.

## Validación

Comprobar el estado:

```cmd
manage-bde -status
```

Verificar en Active Directory:

```text
Equipo
 └─ Pestaña BitLocker Recovery
```

Resultado esperado:

- Claves de recuperación almacenadas correctamente.

---

# GPO: LAPS - Equipos

## Objetivo

Gestionar automáticamente las contraseñas de administrador local de los equipos unidos al dominio.

## Configuración aplicada

### Contraseñas

- Longitud mínima: 12 caracteres.
- Complejidad habilitada:
  - Mayúsculas.
  - Minúsculas.
  - Números.
  - Caracteres especiales.
- Rotación automática cada 30 días.

### Almacenamiento

- Copia de seguridad en Active Directory.
- Cifrado habilitado para la información almacenada.

### Cuenta administrada

Cuenta local gestionada:

```text
usuario
```

## Beneficios

- Eliminación de contraseñas locales compartidas.
- Reducción de movimientos laterales.
- Gestión centralizada de credenciales.
- Mayor trazabilidad administrativa.

## Validación

Consulta desde PowerShell:

```powershell
Get-LapsADPassword -Identity NOMBRE-EQUIPO
```

Resultado esperado:

- Visualización de la contraseña actual.
- Fecha de expiración de la contraseña.

---

# Directiva de dominio predeterminada

## Objetivo

Aplicar configuraciones básicas comunes a todo el dominio.

## Funciones

- Directivas de contraseña.
- Configuración Kerberos.
- Configuración base de seguridad del dominio.

## Beneficios

- Homogeneidad de configuración.
- Seguridad de autenticación.
- Gestión centralizada de usuarios.

---

# Tecnologías utilizadas

- Active Directory
- Group Policy Management (GPMC)
- Group Policy Preferences
- BitLocker
- TPM
- Microsoft LAPS
- PowerShell
- Windows Server

---

# Capacidades demostradas

Mediante estas directivas se han puesto en práctica conocimientos de:

- Administración de Active Directory.
- Diseño y aplicación de GPO.
- Seguridad de endpoints.
- Cifrado de equipos corporativos.
- Gestión de privilegios.
- Automatización mediante PowerShell.
- Administración centralizada de estaciones de trabajo.
- Hardening básico de sistemas Windows.

---

# Mejoras previstas

- Auditoría avanzada mediante GPO.
- Configuración corporativa de Windows Firewall.
- Despliegue automatizado de software.
- Integración con WSUS.
- Automatización de configuración inicial de equipos.
