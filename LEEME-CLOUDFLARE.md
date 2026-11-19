# SURCOS STREAMING 1.1 — IMAP y panel exclusivo de buzones

La página sigue siendo de consulta directa: no se añadió registro, login de
clientes, lista de usuarios, vencimientos ni API de administración de usuarios.
script.js, styles.css e imágenes se mantienen idénticos a la versión API.
En index.html solo se añadió un enlace flotante Administrar buzones a /admin.html.
El cuerpo de los correos y la redirección de enlaces usan el mismo script.js.

## 1. Subir al repositorio y desplegar

Haz una copia de tu versión actual. Descomprime este ZIP y sube el CONTENIDO de
surcosstreaming-main a la raíz del repositorio que publica tu página, reemplazando los
archivos correspondientes. No subas el ZIP como tal ni otra carpeta por encima.

Sube las carpetas functions y lib completas. Conserva el nuevo package-lock.json:
instala la dependencia postal-mime. No subas node_modules, dist, .env ni .dev.vars.
Si ya tienes buzones registrados en esta modalidad, conserva tu mailboxes.json:
NO lo sustituyas por el archivo vacío del ZIP.

En Cloudflare Pages conserva la integración con GitHub:

| Campo | Valor |
|---|---|
| Rama de producción | main |
| Framework | None |
| Build command | npm run build |
| Build output directory | dist |
| Root directory | Vacío: raíz del repositorio |

No necesitas D1, KV, proxy IMAP ni flags de Node.js. El cliente usa la API TCP
nativa de Cloudflare, con TLS directo hacia imap.gmail.com:993.

## 2. Variables en Cloudflare

Settings > Variables and Secrets, entorno Production:

| Nombre | Valor | Tipo |
|---|---|---|
| GITHUB_TOKEN | Token con Contents: Read and write sobre el repositorio | Secreto |
| GITHUB_REPO | virtualformacion/surcosstreaming, o propietario/repositorio real | Texto |
| GITHUB_FILE_PATH | script.js (se conserva por compatibilidad con la configuración común) | Texto |
| GITHUB_BRANCH | main | Texto |
| MAILBOX_ENCRYPTION_KEY | Secreto aleatorio de al menos 32 caracteres | Secreto |

Este proyecto NO modifica script.js para crear usuarios. GITHUB_FILE_PATH no se
usa para guardar usuarios; los buzones se escriben en mailboxes.json.
Si tu repositorio se llama surcosstreaming-cloudflare, cambia GITHUB_REPO por
virtualformacion/surcosstreaming-cloudflare. Debe ser el mismo que publica esta página.

Configura MAILBOX_ENCRYPTION_KEY una sola vez en Cloudflare, nunca en GitHub.
Guárdala también en tu gestor de contraseñas. Se recomienda un valor aleatorio
de 64 caracteres hexadecimales; con Node instalado puedes generarlo localmente
con npm run key y copiarlo al secreto. No lo uses como comando de compilación.
No cambies esa clave después de guardar buzones: se necesita para descifrarlos.
Las claves de aplicación de Gmail se introducen en el panel, no como variables.
GMAIL_CLIENT_ID, GMAIL_CLIENT_SECRET y GMAIL_REFRESH_TOKEN ya no se utilizan.
Después de guardar variables y secretos, vuelve a desplegar.

## 3. Entrar al panel de buzones

Pulsa Administrar buzones en la página principal o abre /admin.html.

Usuario inicial: admin
Contraseña inicial: 0000

Puedes cambiarlos en lib/admin-config.js, dentro de FIXED_ADMIN, y guardar el
commit para volver a desplegar. No hay vencimiento del administrador en esta
página sencilla. Es el único login: no se añade autenticación a la consulta pública.

Solo podrás gestionar buzones; no hay botones para crear ni editar usuarios.

## 4. Agregar y gestionar Gmail

Crea una clave de aplicación en la cuenta Google del buzón:
https://myaccount.google.com/apppasswords
Necesitas verificación en dos pasos y que las políticas de la cuenta permitan
claves de aplicación. Usa la clave de 16 letras, no la contraseña normal de Gmail.
Puedes pegarla con espacios: se eliminan automáticamente.

En el panel:
1. Escribe un nombre y el correo real del buzón.
2. Pega su clave de aplicación.
3. Deja Carpeta vacía para detectar Todos los correos de Gmail por su atributo IMAP.
4. Deja Buzón activo marcado y pulsa Guardar buzón.
5. Se prueba la conexión antes de guardar un buzón activo.
6. Usa Probar conexión para comprobar después el acceso y la carpeta.

Puedes agregar, editar, activar, desactivar, probar y eliminar registros.
Al editar, clave vacía conserva la existente. Cambiar el correo requiere su nueva
clave de aplicación. Eliminar no borra correos ni revoca la clave en Google.
Las claves no se muestran ni se devuelven al navegador al listar.
Se admiten hasta 10 buzones activos, consultados como máximo de dos en dos.

Todos los registros se guardan en mailboxes.json en GitHub. Sus claves de
aplicación se cifran con AES-256-GCM; nombre, correo, carpeta y estado sí son
visibles en el repositorio. El secreto para descifrar permanece en Cloudflare.
El archivo no se publica en dist, pero si tu repositorio es público se pueden
ver los metadatos y los textos cifrados allí. No se guarda ninguna clave real en
este ZIP. Perder MAILBOX_ENCRYPTION_KEY exige volver a introducir las claves de
aplicación; rotar GITHUB_TOKEN no cambia el cifrado.

Cada cambio crea un commit que puede activar una compilación de Pages. Las
consultas nuevas leen el archivo de GitHub sin esperar al nuevo despliegue.
Los conflictos de edición se protegen con el SHA del archivo: si dos personas
guardan a la vez, recarga la lista y repite el cambio rechazado.

## 5. Resultados y alcance

Se conservan los asuntos originales de SURCOS STREAMING, los prefijos de enlaces Netflix,
la prioridad de código sobre enlace y la ventana original de 10 minutos.
Se revisan los últimos 99 mensajes de CADA buzón activo, se reúnen por fecha y
se devuelve el código admitido más reciente o, si no hay, el enlace admitido.

La conexión usa EXAMINE y BODY.PEEK: no borra mensajes ni los marca como leídos.
Por defecto Todos los correos incluye archivados y enviados; Spam y Papelera no
se agregan automáticamente. Si Todos los correos no es visible en IMAP, activa
Mostrar en IMAP para esa carpeta en Gmail. Puedes indicar INBOX para solo recibidos.

Se leen cuerpos MIME en HTML/texto, Base64, quoted-printable y mensajes adjuntos
message/rfc822. Se mantiene el resultado body o link que ya entiende la página.
Cuando no hay HTML se devuelve el texto disponible, sin reproducir el snippet
propietario que genera Gmail API.

Se comprueban destinatarios To/Cc y cabeceras de entrega, bloques Para:/To: de
reenvíos y mensajes adjuntos reenviados. Se admiten puntos Gmail, googlemail.com
y extensiones +tag conservando cada etiqueta: usuario+uno y usuario+dos no se
confunden. Si el reenvío elimina la referencia al destinatario original no se
puede recuperar con certeza: prueba tus formatos reales.

Si un buzón falla se consultan los demás. Si no hay resultado y la consulta quedó
incompleta se informa del error. Las sesiones tienen tiempos de espera y cierran
sus conexiones. Para acotar memoria se admiten hasta 4 MiB por mensaje, 12 MiB por
buzón y 24 MiB por consulta; un exceso se informa como fallo del buzón.
Siguen aplicándose las cuotas de CPU/ejecución de Cloudflare y las de Gmail/GitHub.

## 6. Archivos principales

| Archivo | Función |
|---|---|
| admin.html y admin-login.js | Login del administrador y panel solo de buzones |
| mailbox-admin.js | Formularios, botones y listado |
| lib/admin-config.js | Usuario admin y clave inicial 0000, editables |
| lib/admin-auth.js | Validación del administrador |
| functions/api/getLastEmail.js | Entrada de consulta por IMAP |
| functions/api/mailboxes.js | API de gestión y prueba de buzones |
| lib/imap-protocol.js | Conexión TCP TLS 993 y lectura del protocolo |
| lib/imap-mail.js y lib/mail-rules.js | MIME, destinatarios y reglas originales |
| lib/mailbox-store.js | Lectura/escritura de mailboxes.json en GitHub |
| lib/mailbox-vault.js | Validación y cifrado de claves |
| lib/mailbox-handler.js y lib/consult-handler.js | Operaciones del panel y consulta |
| lib/encoding.js | Base64 para almacenar el JSON en GitHub |
| mailboxes.json | Registros iniciales vacíos; no sobrescribir después de configurar |
| build.mjs, package.json y package-lock.json | Publicación y dependencia MIME |
| index.html | Solo se añadió el enlace al administrador |
| script.js, styles.css e imágenes | Conservados sin cambios |

## 7. Validación

Las nueve pruebas automatizadas cubren los asuntos y enlaces originales,
prioridad de resultados, reenvíos, alias, MIME, cifrado, gestión GitHub y datos
IMAP fragmentados en un socket simulado. Wrangler valida la compilación de Pages.
No se probó una cuenta real de Gmail ni se modificó tu GitHub desde aquí.

Después de desplegar: entra con admin/0000, agrega un buzón, prueba la conexión,
consulta un código directo, un reenvío, una dirección +tag y una redirección.
Comprueba también editar/desactivar/eliminar un buzón. Conserva un respaldo de
la versión API mientras verificas tus casos reales.

Referencias:
https://developers.cloudflare.com/workers/runtime-apis/tcp-sockets/
https://support.google.com/accounts/answer/185833
https://developers.google.com/workspace/gmail/imap/imap-extensions
https://github.com/postalsys/postal-mime
