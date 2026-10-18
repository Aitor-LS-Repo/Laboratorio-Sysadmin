# Registro de incidencias

Problemas reales encontrados durante el montaje del laboratorio, con el formato **síntoma → diagnóstico → solución → aprendizaje**. Los campos entre `<>` son para completar con lo que hiciste exactamente.

---

## 1. Un usuario podía borrar la carpeta raíz de un recurso compartido

**Síntoma:** al acceder por ruta de red al share de TrueNAS, un usuario de dominio podía eliminar la carpeta raíz completa.

**Diagnóstico:** revisión de las ACL NFSv4 del dataset. El grupo tenía permisos que incluían `Delete` y `Delete Children` sobre el propio objeto raíz, no solo sobre su contenido.

**Solución:** `<describe qué ACE modificaste: quitar Delete / Delete Children, o añadir una ACE Deny con Delete aplicada solo a "este objeto">`.

**Aprendizaje:** en ACL NFSv4 hay que distinguir entre borrar el objeto y borrar sus hijos, y el orden de las entradas importa. Se prueba siempre con un usuario sin privilegios, no solo con el administrador.

---

## 2. Carpeta personal de AD que no se mapeaba

**Síntoma:** la unidad de red de la carpeta personal no se creaba ni se montaba al iniciar sesión.

**Diagnóstico:** en la pestaña Perfil del usuario se había escrito `\\servidor\%userlogon%`. Esa variable no existe, y además faltaba el nombre del recurso compartido.

**Solución:** usar la variable correcta y la ruta completa: `\\<servidor>\<share>\%username%`.

**Aprendizaje:** las variables de entorno de Windows hay que comprobarlas contra la documentación, no a ojo. Un valor inválido se interpreta como texto literal sin dar error.

---

## 3. Carpeta personal: sin permiso para crear o modificar, pero sí para borrar

**Síntoma:** el usuario no podía crear ni modificar archivos en el share, pero sí borrar la carpeta.

**Diagnóstico:** los permisos del grupo en la raíz eran solo de lectura y listado, y quedaba una entrada antigua con `Delete` / `Delete Children`. Es decir, los permisos estaban invertidos respecto a lo necesario.

**Solución:** `<describe el ajuste: Write Data y Append Data donde hacían falta, retirar Delete y Delete Children>`.

**Aprendizaje:** al endurecer permisos hay que revisar todas las entradas de la lista, no solo la que se acaba de tocar.

---

## 4. pfSense no podía autenticar contra Active Directory

**Síntoma:** en el log de autenticación aparecía `Could not bind to LDAP server ... check the bind credentials`.

**Diagnóstico:** el fallo era en el bind inicial con la cuenta de servicio, antes de comprobar al usuario final. Se revisaron credenciales, formato del DN, conectividad entre VLANs y reglas de firewall.

**Solución:** `<causa real y arreglo: contraseña, DN, regla de firewall, cambio a LDAPS...>`.

**Aprendizaje:** leer el log antes de tocar nada. Un mismo mensaje puede venir de credenciales, red o certificados, y conviene descartarlos por orden de probabilidad. La autenticación se prueba en **Diagnostics → Authentication** antes de configurar la VPN.

---

## 5. Filtro de grupo LDAP escrito en el campo equivocado

**Síntoma:** ninguna autenticación funcionaba, ni siquiera para miembros del grupo.

**Diagnóstico:** el filtro `memberOf=CN=VPN_Admins,...` estaba en el campo **Base DN** en lugar de en **Consulta extendida**.

**Solución:** dejar en Base DN solo la raíz del dominio y poner el filtro en la consulta extendida.

**Aprendizaje:** Base DN delimita dónde se busca; la consulta extendida filtra a quién se admite. El filtro debe usar un grupo de seguridad, no una OU.

---

## 6. BitLocker pedía PIN en cada arranque

**Síntoma:** el equipo solicitaba un PIN antes de iniciar Windows.

**Diagnóstico:** la directiva permitía PIN de inicio con TPM y el cifrado se había activado con ese protector.

**Solución:** en la GPO, `Configurar PIN de inicio del TPM` → no permitir; y en el equipo ya cifrado, sustituir el protector con `manage-bde -protectors`.

**Aprendizaje:** cambiar una GPO no modifica los protectores de un disco que ya está cifrado. Se verifica con `gpresult` qué directivas llegan realmente al equipo.

---

## Plantilla para nuevas incidencias

## N. Título breve del problema

**Síntoma:**

**Diagnóstico:**

**Solución:**

**Aprendizaje:**
