# Seguridad de equipos: LAPS y BitLocker

Estado: `operativo`
Última revisión: `<AAAA-MM-DD>`

## Objetivo

- **LAPS:** que cada equipo tenga una contraseña de administrador local distinta, aleatoria y guardada en AD.
- **BitLocker:** cifrar el disco de los equipos con TPM y guardar las claves de recuperación en AD.

## LAPS (Windows LAPS)

### Configuración

1. Extender el esquema de AD: `Update-LapsADSchema`.
2. Dar permiso a los equipos de una OU para actualizar su propia contraseña: `Set-LapsADComputerSelfPermission -Identity "<OU>"`.
3. Crear una GPO en **Configuración del equipo → Plantillas administrativas → Sistema → LAPS**: destino de la copia de la contraseña (Active Directory), complejidad, longitud y periodo de rotación.
4. Activar el cifrado de la contraseña almacenada.
5. Limitar quién puede leer las contraseñas con `Set-LapsADReadPasswordPermission`.
6. Vincular la GPO a la OU de equipos y ejecutar `gpupdate /force` en el cliente.

### Verificación

`Get-LapsADPassword -Identity "<equipo>" -AsPlainText` devuelve la contraseña, y coincide al iniciar sesión localmente.

## BitLocker con TPM

### Configuración

1. **TPM virtual en Proxmox:** en la VM cliente se añade un TPM 2.0 y se usa arranque UEFI (máquina q35 y BIOS OVMF).
2. **GPO** en **Componentes de Windows → Cifrado de unidad BitLocker → Unidades del sistema operativo**:
   - *Requerir autenticación adicional al iniciar*: habilitada, con TPM permitido.
   - Copia de seguridad de la clave de recuperación en AD DS, y obligatoria.
   - PIN de inicio: **no permitir** (modo transparente, solo TPM).
3. Vincular la GPO a la OU de equipos.
4. Activar BitLocker en el cliente. La GPO configura el comportamiento, pero no activa el cifrado por sí sola. Para automatizarlo se puede usar un script con `Enable-BitLocker` desplegado por GPO.

### Verificación

1. La clave de recuperación aparece en la pestaña **BitLocker Recovery** del objeto de equipo en AD.
2. `gpresult /h informe.html /scope:computer` confirma las directivas aplicadas.
3. `manage-bde -status` muestra el estado del cifrado y los protectores.

## Cuidado

- No activar BitLocker en servidores críticos sin tener antes la clave de recuperación guardada y comprobada.
- Cambiar la GPO no quita el protector de PIN de un disco ya cifrado: hay que sustituirlo con `manage-bde -protectors`.
- Si BitLocker está activo, Sysprep falla.

## Problemas encontrados

Ver [incidencias](incidencias.md), punto 6.
