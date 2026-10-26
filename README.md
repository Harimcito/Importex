# Importex — panel local de análisis de importaciones

Panel local para comparar precios en Chile, registrar ofertas de proveedores y estimar costo de importación. Las integraciones externas se activan sólo después de configurar sus credenciales.

## Iniciar

1. Abre una terminal en esta carpeta.
2. Ejecuta `python -m pip install -r requirements.txt` para habilitar extracción de texto PDF y Excel.
3. Ejecuta `python app.py`.
4. Abre `http://127.0.0.1:8765` en tu navegador.
5. Para detenerlo, vuelve a la terminal y pulsa `Ctrl+C`.

Los datos estructurados quedan en `comercio.sqlite3`, dentro de esta carpeta. Haz una copia de ese archivo para respaldar el catálogo, precios y cotizaciones.

## Qué incluye esta versión

- Catálogo inicial editable de resmas tamaño Carta y Oficio, ambas de 75 g/m² y 500 hojas.
- Registro de observaciones de precio chileno, con vendedor, producto, moneda, unidad, fecha y URL.
- Búsqueda manual de anuncios públicos de MercadoLibre Chile por su API. Los resultados no se guardan solos; el usuario revisa el formato y confirma cada referencia. MercadoLibre puede limitar o cambiar el acceso público.
- Recolección diaria programable de MercadoLibre. Con una aplicación OAuth de sólo lectura conectada, busca los términos definidos para hasta 10 productos a la hora elegida (por defecto 10:00 local), conserva precio, moneda, título, vendedor, enlace, identificador y fecha; evita duplicar la misma publicación en el mismo día. Los registros automáticos quedan “Por revisar” y no cuentan como coincidencia confirmada.
- Importación local de CSV del catálogo Convenio Marco de ChileCompra. El archivo se lee en el navegador, se muestra una muestra limitada a 500 filas y cada precio requiere selección de producto y confirmación. Es una referencia de compras institucionales, no un precio minorista.
- Consulta manual de estimación de flete con ComexChile. Pide origen, destino Arica, modalidad, bultos, dimensiones y peso; el resultado se conserva sólo en pantalla y no rellena por sí solo la calculadora. Sus valores pueden ser estimados o no publicados, y requieren confirmación directa con el proveedor logístico.
- Lectura asistida de páginas HTML públicas cuando exponen un precio estructurado. El precio detectado debe confirmarse antes de guardarse.
- Análisis opcional con OpenAI, sólo después de que el usuario lo solicite: envía el nombre/especificación del producto del catálogo y hasta 5.000 caracteres de título, URL y texto de una página pública. No envía correos. La clave se guarda cifrada con Windows DPAPI; el resultado es una sugerencia y el usuario confirma cualquier precio.
- Registro de cotizaciones de proveedores, incluyendo MOQ, Incoterm y condiciones de pago.
- Conversión visible de observaciones y cotizaciones en moneda original, USD y CLP. Las tasas de USD/CNY/CLP/BRL se consultan sólo cuando se pulsa el botón y quedan fechadas localmente. Si se guarda un token BDE, USD/CLP usa el dólar observado del Banco Central y CNY/BRL usan [Frankfurter](https://frankfurter.dev/); sin token BDE, las tasas provienen de Frankfurter.
- Lectura IMAP SSL/TLS diaria opcional de mensajes nuevos en INBOX y la carpeta Enviados que anuncie el servidor. Lee los mensajes de esas carpetas sin filtro de remitente usando modo sólo lectura y `BODY.PEEK`; no marca mensajes como leídos ni los modifica. La contraseña se cifra con Windows DPAPI.
- Los campos estructurados de correos detectados se guardan en una bandeja “Por revisar”. No se conservan el cuerpo ni los archivos originales. La cotización sólo pasa al registro comercial cuando el usuario la confirma.
- Importación de un archivo `.eml` seleccionado por el usuario; se proponen campos y sólo se guardan después de revisar y confirmar. El archivo original y el cuerpo del mensaje no se conservan.
- Extracción local de texto en adjuntos PDF, XLSX, XLSM, CSV y TXT compatibles, con límites de tamaño/filas/páginas. No se ejecutan macros ni se guardan los adjuntos. PDFs escaneados sin texto requieren OCR y quedan identificados como pendientes de lectura.
- Calculadora editable de CIF, arancel, IVA de importación, gastos locales, costo neto, contribución y equilibrio. Permite indicar si el FOB está cotizado por unidad o por caja y convierte usando las unidades de la caja.
- Guardado de múltiples escenarios de costos en SQLite, con sus supuestos completos para volver a abrirlos, compararlos y eliminarlos. Los escenarios quedan en el computador donde corre el panel.
- Plantilla inicial de hitos y reinversión del 40% sobre utilidad cobrada.
- Alertas diarias por correo a una hora local elegida, usando SMTP del correo del usuario (sin contratar un servicio de notificaciones). El resumen incluye sólo precios y cotizaciones guardados en las últimas 24 horas y se envía como máximo una vez al día, sólo si hay datos nuevos. El panel debe estar abierto y el computador encendido.
- La contraseña SMTP puede guardarse cifrada con Windows DPAPI, asociada al perfil de Windows que la configuró. La app no la muestra ni la envía a OpenAI. El correo de prueba sólo se envía cuando el usuario lo solicita desde el panel.

## Estructura de la interfaz

- `static/index.html` contiene la estructura semántica de las pantallas.
- `static/app.css` concentra estilos, diseño adaptable y estados de foco.
- `static/app.js` contiene navegación, ayuda contextual y comportamiento de la interfaz; las secciones consultan sus datos al abrirse para evitar peticiones innecesarias al iniciar.
- `app.py` sirve únicamente estos recursos estáticos permitidos y las rutas locales de la API.

## Límites actuales

- Para conectar MercadoLibre: crea una aplicación en [Mis Aplicaciones](https://developers.mercadolibre.cl/es_ar/configuracion-o-requisitos-previos), habilita permiso de lectura y acceso offline, registra una Redirect URI HTTPS fija y guarda Client ID/Secret en la sección Automatización diaria. Autoriza la aplicación, copia el parámetro `code` de la URL de retorno y completa el campo `state` que Importex genera. Los secretos y tokens se cifran con Windows DPAPI; MercadoLibre rota el refresh token de un solo uso y la app guarda cada token nuevo. La recolección funciona sólo mientras el panel está abierto y el PC encendido.
- Las consultas diarias son búsquedas de publicaciones, no cotizaciones formales de proveedores. Títulos como “resma carta” pueden referirse a gramajes, cantidades, marcas o paquetes distintos. Por eso se guarda cada observación para revisar y no se marca como verificada. El stock publicado por la API puede ser un rango referencial.
- MercadoLibre puede responder 401/403/429, revocar el acceso o exigir reautorización. El estado de la última ejecución queda visible en el panel. La documentación exige OAuth y aconseja incluir el token Bearer en las llamadas; no se automatiza un scraping para saltarse ese acceso.

- La conexión inicial usa el puerto 993, común para IMAP SSL/TLS en cPanel. El servidor y la cuenta se completan en la pantalla; verifica los valores exactos en tu hosting. [Guía de configuración de correo cPanel](https://docs.cpanel.net/cpanel/email/set-up-mail-client/)
- La primera activación establece un punto inicial y no importa correos antiguos. Importex procesa el cuerpo y adjuntos compatibles localmente, pero conserva sólo campos extraídos, asunto, remitente y carpeta para revisión; no guarda cuerpo ni adjuntos originales. La contraseña IMAP se escribe en el panel y queda cifrada con DPAPI; nunca se pega en el chat.
- DPAPI intenta primero cifrado ligado al perfil Windows. Si Importex corre bajo un perfil aislado sin esas claves, usa protección DPAPI de equipo; otras cuentas locales que tengan acceso al archivo de datos podrían descifrar ese secreto. El panel muestra cuándo ocurre ese modo.
- Para extraer texto PDF y Excel se instalan las dependencias de `requirements.txt`. La extracción de texto no es OCR: archivos escaneados deben procesarse aparte.
- La consulta de precio web es un lector genérico. No funciona de forma fiable en sitios con inicio de sesión, contenido cargado por JavaScript, bloqueo de robots o formatos particulares; 1688 puede requerir ingreso manual de precio CNY y MOQ.
- Las consultas manuales a MercadoLibre y ComexChile ocurren sólo al pulsar sus botones. Si activas la programación de MercadoLibre, se consultará a diario y sus datos concretos se guardarán automáticamente como “Por revisar”. Los resultados de ChileCompra se guardan sólo tras confirmar cada fila. No se automatiza el acceso de 1688 ni se copia código de otros servicios.
- El cotizador de ComexChile requiere peso y dimensiones reales por bulto. Su estimación puede no cubrir el tramo final a Arica, almacenaje, documentación, manejo portuario ni cargos aduaneros; revisa el desglose y la vigencia con un agente de carga antes de usarlo en una decisión.
- ChileCompra permite guardar localmente y validar el ticket de su API. La validación consulta sólo el listado de licitaciones del día; la importación CSV sigue disponible para referencias de precios. Las observaciones de compras públicas no son automáticamente comparables con venta minorista.
- En Precios Chile se guardan cifrados con Windows DPAPI el token del Banco Central y el ticket de ChileCompra. El botón de tipo de cambio no se ejecuta de forma automática. La validación de ChileCompra tampoco se programa; ambas requieren una acción explícita en el panel.
- El servidor SMTP, el puerto y el uso de SMTP pueden depender del plan de hosting. Comprueba en cPanel que el correo saliente esté incluido y los valores exactos antes de habilitar las alertas. El panel propone puerto 465 con SSL y permite 587 con STARTTLS; completa el servidor y la cuenta de tu hosting.
- Las alertas resumen datos que ya se guardaron manualmente en el panel. Aún no rastrean automáticamente páginas, sitios con sesión ni generan una evaluación de oportunidad basada en umbrales de margen.
- No hace pedidos, envía correos, guarda credenciales ni aplica automáticamente una preferencia arancelaria. La calculadora parte sin cotización FOB, flete, seguro, gastos locales, arancel confirmado ni precio de venta; exige rellenar esos supuestos antes de calcular.
- La llamada a OpenAI usa la API Responses. El modelo inicial mostrado es editable; el acceso y disponibilidad dependen de la cuenta/API del usuario. La clave API es un secreto y no se debe incluir en URLs, archivos compartidos ni mensajes.
- Las tasas de cambio son indicativas, no tasas bancarias ni aduaneras. Actualízalas y revisa su fecha antes de comparar; ingresa 0 manualmente si una partida de costo se cotiza como cero.

## Seguridad y uso

El servidor escucha únicamente en `127.0.0.1` y limita las rutas locales a `localhost`/`127.0.0.1`. No publiques ni reenvíes este servidor a Internet. Las solicitudes de búsqueda de precio sólo deben apuntar a páginas públicas; también se vuelven a revisar los destinos de redirección y se permiten sólo los puertos web 80/443. Evita pegar enlaces que contengan sesiones o credenciales.

Los archivos existentes bajo `sources/` pertenecen al espejo del proyecto y no se modificaron.
