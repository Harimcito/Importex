# Comercio — panel local de validación de importaciones

Primera versión local para comparar precios en Chile, registrar ofertas de proveedores y estimar costo de importación. No requiere cuenta externa ni dependencias de terceros.

## Iniciar

1. Abre una terminal en esta carpeta.
2. Ejecuta `python app.py`.
3. Abre `http://127.0.0.1:8765` en tu navegador.
4. Para detenerlo, vuelve a la terminal y pulsa `Ctrl+C`.

Los datos estructurados quedan en `comercio.sqlite3`, dentro de esta carpeta. Haz una copia de ese archivo para respaldar el catálogo, precios y cotizaciones.

## Qué incluye esta versión

- Catálogo inicial editable de resmas A4/oficio y cuaderno.
- Registro de observaciones de precio chileno, con vendedor, producto, moneda, unidad, fecha y URL.
- Lectura asistida de páginas HTML públicas cuando exponen un precio estructurado. El precio detectado debe confirmarse antes de guardarse.
- Análisis opcional con OpenAI, sólo después de que el usuario lo solicite: envía el nombre/especificación del producto del catálogo y hasta 5.000 caracteres de título, URL y texto de una página pública. No envía correos. La clave se ingresa en pantalla, se mantiene sólo durante la solicitud local y no se guarda en la base; el resultado es una sugerencia y el usuario confirma cualquier precio.
- Registro de cotizaciones de proveedores, incluyendo MOQ, Incoterm y condiciones de pago.
- Lectura IMAP SSL/TLS sólo al pulsar el botón, limitada a remitentes/dominios que el usuario autorice en la pantalla. Usa modo sólo lectura y `BODY.PEEK`; no marca mensajes como leídos ni los modifica. La contraseña se usa en esa conexión, se descarta al terminar y no se guarda en la base.
- Importación de un archivo `.eml` seleccionado por el usuario; se proponen campos y sólo se guardan después de revisar y confirmar. El archivo original y el cuerpo del mensaje no se conservan.
- Calculadora editable de CIF, arancel, IVA de importación, gastos locales, costo neto, contribución y equilibrio.
- Plantilla inicial de hitos y reinversión del 40% sobre utilidad cobrada.

## Límites actuales

- La conexión inicial usa el puerto 993, común para IMAP SSL/TLS en cPanel. El servidor y la cuenta se completan en la pantalla; verifica los valores exactos en tu hosting. [Guía de configuración de correo cPanel](https://docs.cpanel.net/cpanel/email/set-up-mail-client/)
- El usuario debe escribir una lista de remitentes/dominios permitidos antes de conectar; con la lista vacía el panel no busca mensajes. Para iniciar sesión se introduce la contraseña del buzón en el panel local; no se guarda. No enviar contraseñas por el chat.
- No extrae el texto de adjuntos PDF/Excel; el parser lee el cuerpo de texto del correo y omite adjuntos. Los correos cuyo precio sólo esté en un archivo adjunto aún requieren revisión manual.
- La consulta de precio web es un lector genérico. No funciona de forma fiable en sitios con inicio de sesión, contenido cargado por JavaScript, bloqueo de robots o formatos particulares; 1688 puede requerir ingreso manual de precio CNY y MOQ.
- No hace pedidos, envía correos, guarda credenciales ni aplica automáticamente un tipo de cambio o preferencia arancelaria.
- La llamada a OpenAI usa la API Responses. El modelo inicial mostrado es editable; el acceso y disponibilidad dependen de la cuenta/API del usuario. La clave API es un secreto y no se debe incluir en URLs, archivos compartidos ni mensajes.
- Los valores iniciales de la calculadora son editables y sólo sirven como ejemplo; reemplázalos por cotizaciones vigentes.

## Seguridad y uso

El servidor escucha únicamente en `127.0.0.1`. No publiques ni reenvíes este servidor a Internet. Las solicitudes de búsqueda de precio sólo deben apuntar a páginas públicas. Evita pegar enlaces que contengan sesiones o credenciales.

Los archivos existentes bajo `sources/` pertenecen al espejo del proyecto y no se modificaron.

