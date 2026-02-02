# Pruebas Funcionales - HU-004: Carga de Facturas Electrónicas Externas

**Historia de Usuario**: [HU-004-carga-facturas-externas.md](./HU-004-carga-facturas-externas.md)

**Fecha de creación**: 2025-12-09

**Responsable**: QA Team

---

## Objetivo

Validar que la funcionalidad de carga de facturas electrónicas externas desde archivos XML (AttachedDocument) de DIAN permita validar certificados digitales, firmas digitales, extraer campos, consultar PDFs en DIAN, y almacenar correctamente la información en las bases de datos con auditoría completa.

---

## Pre-requisitos

- Usuario con certificado digital válido autenticado en el sistema
- Acceso al ambiente de pruebas de DIAN E-Factura API
- Base de datos MongoDB para almacenamiento de XMLs
- Base de datos relacional con tablas de facturas
- Archivos XML de prueba (AttachedDocuments válidos)
- Sistema de almacenamiento para PDFs configurado
- Acceso a la pantalla de carga de facturas externas

---

## Casos de Prueba

### PF-004-01: Cargar archivo XML válido exitosamente

**Descripción**: Verificar que se puede cargar un archivo XML (AttachedDocument) válido y que el sistema lo procesa completamente.

**Datos de Entrada**:
- Archivo: `factura_valida_001.xml` (AttachedDocument válido)
- Contenido del XML:
  - Prefijo: SETP
  - Número: 12345
  - CUFE: abc123def456789...
  - NIT Emisor: 890930534 (CADENA S.A.)
  - NIT Receptor: 901453011 (PRESIZA S.A.S)
  - Monto: $1,500,000.00 COP
  - Fecha emisión: 2025-12-09
  - Fecha vencimiento: 2026-01-08
- Usuario: Certificado digital con NIT 890930534 (corresponde al emisor)

**Pasos**:
1. Iniciar sesión con certificado digital
2. Navegar a "Carga de Facturas Externas"
3. Seleccionar archivo XML
4. Hacer clic en "Cargar" o "Procesar"
5. Esperar a que termine el procesamiento

**Resultado Esperado**:
- ✅ El archivo se carga sin errores
- ✅ Se muestra progreso de los pasos:
  - Validando estructura XML ✓
  - Verificando certificado digital ✓
  - Validando firma digital ✓
  - Validando reglas de negocio ✓
  - Extrayendo campos ✓
  - Consultando PDF en DIAN ✓
  - Guardando en base de datos ✓
  - Enviando notificación ✓
- ✅ Se muestra mensaje de éxito
- ✅ Se muestra resumen de la factura con todos los datos
- ✅ Enlaces a PDF y XML están disponibles
- ✅ En BD relacional:
  - Existe registro con todos los campos
  - CUFE, número, emisor, receptor correctos
  - Estado = "Cargada"
- ✅ En MongoDB:
  - Existe documento con XML completo
  - Campos extraídos correctos
  - Metadata completa
- ✅ PDF descargado y almacenado
- ✅ Registro de auditoría creado

---

### PF-004-02: Validar formato XML incorrecto

**Descripción**: Verificar que se rechaza un archivo que no es XML válido.

**Datos de Entrada**:
- Archivo: `archivo_invalido.xml` (XML malformado)
- Contenido: XML con errores de sintaxis (etiquetas sin cerrar, caracteres inválidos)

**Pasos**:
1. Intentar cargar archivo con XML malformado

**Resultado Esperado**:
- ✅ Error detectado inmediatamente
- ✅ Mensaje: "El archivo XML no cumple con el formato esperado"
- ✅ Detalle específico del error de parseo
- ✅ No se procesa el archivo
- ✅ No se crea ningún registro en BD

---

### PF-004-03: Validar que no es AttachedDocument

**Descripción**: Verificar que se rechaza un XML que no es un AttachedDocument de DIAN.

**Datos de Entrada**:
- Archivo: `otro_xml.xml` (XML válido pero no AttachedDocument)
- Contenido: XML bien formado pero con estructura diferente

**Pasos**:
1. Cargar archivo XML que no es AttachedDocument

**Resultado Esperado**:
- ✅ Error: "No es un AttachedDocument válido"
- ✅ No se procesa
- ✅ Sugerencia: "Debe cargar un archivo AttachedDocument de factura electrónica DIAN"

---

### PF-004-04: Validar certificado digital - NIT no coincide

**Descripción**: Verificar que se rechaza cuando el NIT del certificado no corresponde al emisor ni receptor.

**Datos de Entrada**:
- Archivo XML con:
  - NIT Emisor: 890930534
  - NIT Receptor: 901453011
- Usuario autenticado con certificado digital:
  - NIT: 800123456 (diferente a emisor y receptor)

**Pasos**:
1. Cargar factura con certificado que no corresponde

**Resultado Esperado**:
- ✅ Error en paso "Verificando certificado digital"
- ✅ Mensaje: "El certificado digital no corresponde al emisor o receptor de la factura"
- ✅ Detalle mostrado:
  - Su NIT: 800123456
  - NIT emisor: 890930534
  - NIT receptor: 901453011
- ✅ Sugerencia: "Verifique que está utilizando el certificado digital correcto"
- ✅ No se procesa la factura

---

### PF-004-05: Validar certificado digital - Certificado coincide con emisor

**Descripción**: Verificar que se acepta cuando el certificado corresponde al NIT del emisor.

**Datos de Entrada**:
- XML con NIT Emisor: 890930534
- Certificado con NIT: 890930534 (coincide con emisor)

**Pasos**:
1. Cargar factura

**Resultado Esperado**:
- ✅ Validación de certificado pasa exitosamente
- ✅ Continúa con el procesamiento

---

### PF-004-06: Validar certificado digital - Certificado coincide con receptor

**Descripción**: Verificar que se acepta cuando el certificado corresponde al NIT del receptor.

**Datos de Entrada**:
- XML con NIT Receptor: 901453011
- Certificado con NIT: 901453011 (coincide con receptor)

**Pasos**:
1. Cargar factura

**Resultado Esperado**:
- ✅ Validación de certificado pasa exitosamente
- ✅ Continúa con el procesamiento

---

### PF-004-07: Validar firma digital inválida

**Descripción**: Verificar que se rechaza un AttachedDocument con firma digital inválida.

**Datos de Entrada**:
- Archivo XML con firma digital corrupta o inválida

**Pasos**:
1. Cargar archivo con firma inválida

**Resultado Esperado**:
- ✅ Error en paso "Validando firma digital"
- ✅ Mensaje: "La firma digital del documento no es válida"
- ✅ No se procesa
- ✅ Sugerencia: "Solicite el documento original al emisor"

---

### PF-004-08: Validar firma digital válida

**Descripción**: Verificar que se acepta un AttachedDocument con firma digital válida.

**Datos de Entrada**:
- Archivo XML con firma digital válida del emisor o proveedor tecnológico autorizado

**Pasos**:
1. Cargar archivo

**Resultado Esperado**:
- ✅ Validación de firma pasa exitosamente
- ✅ Continúa con el procesamiento

---

### PF-004-09: Validar reglas de negocio - Monto mínimo no cumplido

**Descripción**: Verificar que se aplican reglas de negocio configuradas (ejemplo: monto mínimo).

**Datos de Entrada**:
- XML con monto: $50,000 COP
- Regla configurada: Monto mínimo = $100,000 COP

**Pasos**:
1. Cargar factura con monto menor al mínimo

**Resultado Esperado**:
- ✅ Error en paso "Validando reglas de negocio"
- ✅ Mensaje: "No cumple regla: Monto mínimo de $100,000"
- ✅ Detalle: Monto de la factura: $50,000
- ✅ No se procesa

---

### PF-004-10: Validar reglas de negocio - Todas las reglas cumplidas

**Descripción**: Verificar que continúa el procesamiento cuando todas las reglas se cumplen.

**Datos de Entrada**:
- XML que cumple todas las reglas configuradas

**Pasos**:
1. Cargar factura

**Resultado Esperado**:
- ✅ Validación de reglas pasa
- ✅ Continúa el procesamiento

---

### PF-004-11: Extraer todos los campos requeridos

**Descripción**: Verificar que se extraen correctamente todos los campos del XML.

**Datos de Entrada**:
- XML con todos los campos completos

**Pasos**:
1. Cargar factura
2. Verificar campos extraídos

**Resultado Esperado**:
- ✅ Todos los campos se extraen correctamente:
  - document_prefix, document_number, document_type_id
  - cufe, client_number, debtor_number
  - partnership_id, document_date, document_amount
  - due_date, withholding_tax_amount, paid_amount
  - payment_means_id, document_currency_code
- ✅ Los valores coinciden con el XML
- ✅ Tipos de datos correctos (fechas, decimales, strings)

---

### PF-004-12: Error al extraer campo obligatorio faltante

**Descripción**: Verificar que se detecta cuando falta un campo obligatorio en el XML.

**Datos de Entrada**:
- XML sin el campo `paymentDueDate` (fecha de vencimiento)

**Pasos**:
1. Cargar factura con campo faltante

**Resultado Esperado**:
- ✅ Error en paso "Extrayendo campos"
- ✅ Mensaje: "Campo requerido faltante: due_date (paymentDueDate)"
- ✅ No se procesa

---

### PF-004-13: Consultar PDF en DIAN exitosamente

**Descripción**: Verificar que el PDF se descarga correctamente desde DIAN.

**Datos de Entrada**:
- XML con CUFE válido y registrado en DIAN
- DIAN responde con PDF

**Pasos**:
1. Cargar factura
2. Sistema consulta PDF en DIAN

**Resultado Esperado**:
- ✅ Paso "Consultando PDF en DIAN" exitoso
- ✅ PDF descargado
- ✅ PDF guardado en almacenamiento
- ✅ pdf_url generada y almacenada
- ✅ pdf_pendiente = false
- ✅ PDF accesible desde el enlace

---

### PF-004-14: DIAN no responde - Continuar sin PDF

**Descripción**: Verificar que el procesamiento continúa si DIAN no responde.

**Datos de Entrada**:
- XML válido
- DIAN no disponible (timeout o error 500)

**Pasos**:
1. Cargar factura
2. DIAN no responde en paso de consulta PDF

**Resultado Esperado**:
- ✅ Se muestra advertencia: "No se pudo obtener PDF de DIAN"
- ✅ El procesamiento continúa
- ✅ pdf_url = null
- ✅ pdf_pendiente = true
- ✅ Factura se guarda exitosamente
- ✅ En resumen se indica: "PDF no disponible (se reintentará posteriormente)"
- ✅ Opción para "Reintentar obtener PDF"

---

### PF-004-15: Reintentar consulta de PDF

**Descripción**: Verificar que se puede reintentar la obtención del PDF para una factura que se cargó sin PDF.

**Datos de Entrada**:
- Factura cargada con pdf_pendiente = true

**Pasos**:
1. Ir al detalle de la factura
2. Hacer clic en "Reintentar obtener PDF"

**Resultado Esperado**:
- ✅ Sistema consulta nuevamente DIAN
- ✅ Si es exitoso:
  - PDF descargado y guardado
  - pdf_url actualizada
  - pdf_pendiente = false
  - Mensaje: "PDF obtenido exitosamente"
- ✅ Si falla: Mensaje de error, pdf_pendiente permanece true

---

### PF-004-16: Guardar XML en MongoDB

**Descripción**: Verificar que el XML se guarda correctamente en MongoDB.

**Datos de Entrada**:
- XML procesado con todos los campos extraídos

**Pasos**:
1. Cargar factura hasta el paso de guardado en Mongo

**Resultado Esperado**:
- ✅ Documento creado en colección `facturas_externas` en MongoDB
- ✅ Estructura del documento:
  - _id: ObjectId
  - cufe: valor único
  - xml_original: XML completo como string
  - campos: objeto con todos los campos extraídos
  - metadata: fecha_carga, usuario_carga, pdf_url, estado, etc.
  - auditoria: creado_en, creado_por, ip_origen
- ✅ Documento se puede consultar por CUFE

---

### PF-004-17: Error al guardar en MongoDB

**Descripción**: Verificar el manejo de errores cuando falla el guardado en MongoDB.

**Datos de Entrada**:
- XML válido
- MongoDB no disponible o error de conexión

**Pasos**:
1. Cargar factura
2. Simular error en MongoDB

**Resultado Esperado**:
- ✅ Error crítico: "No se pudo guardar la factura en base de datos"
- ✅ Se detiene el procesamiento
- ✅ No se guarda en BD relacional
- ✅ No se crea notificación
- ✅ Rollback de operaciones previas (ej: PDF guardado)
- ✅ Registro en log de error

---

### PF-004-18: Guardar en BD relacional

**Descripción**: Verificar que los datos se guardan correctamente en la BD relacional.

**Datos de Entrada**:
- Campos extraídos del XML
- xml_url y pdf_url disponibles

**Pasos**:
1. Cargar factura hasta el paso de guardado en BD relacional

**Resultado Esperado**:
- ✅ Registro insertado en tabla `facturas_externas`
- ✅ Todos los campos poblados correctamente
- ✅ Valores coinciden con los extraídos del XML
- ✅ pdf_url y xml_url correctos
- ✅ fecha_carga = timestamp actual
- ✅ usuario_carga = NIT del usuario
- ✅ estado = "Cargada"
- ✅ created_at y updated_at poblados

---

### PF-004-19: Validar CUFE duplicado

**Descripción**: Verificar que se detecta y rechaza una factura con CUFE duplicado.

**Datos de Entrada**:
- XML con CUFE: abc123def456...
- Factura con ese CUFE ya existe en BD (cargada anteriormente)

**Pasos**:
1. Intentar cargar factura con CUFE duplicado

**Resultado Esperado**:
- ✅ Error: "La factura con CUFE abc123def456... ya fue cargada"
- ✅ Se muestra información de la factura existente:
  - Fecha de carga: 2025-12-01 10:30 AM
  - Usuario que la cargó: admin@factoring.com
  - Estado actual: En negociación
  - Número de factura
- ✅ Opciones:
  - [Ver factura existente]
  - [Cancelar]
- ✅ No se procesa el duplicado
- ✅ No se crea registro adicional

---

### PF-004-20: Notificación al buzón del Factor

**Descripción**: Verificar que se envía notificación al buzón del Factor.

**Datos de Entrada**:
- Factura procesada exitosamente

**Pasos**:
1. Completar carga de factura
2. Verificar buzón de notificaciones

**Resultado Esperado**:
- ✅ Notificación enviada al buzón
- ✅ Contenido de la notificación:
  - "Nueva factura cargada"
  - Número de factura
  - Emisor y NIT
  - Receptor y NIT
  - Monto
  - CUFE
  - Enlace para ver detalle
- ✅ Si falla la notificación:
  - Se registra en log
  - No afecta el guardado de la factura

---

### PF-004-21: Visualizar factura en lista de facturas

**Descripción**: Verificar que la factura cargada aparece en la lista de facturas.

**Datos de Entrada**:
- Factura cargada: SETP-12345

**Pasos**:
1. Cargar factura exitosamente
2. Navegar a "Lista de Facturas" o "Facturas Cargadas"

**Resultado Esperado**:
- ✅ La factura SETP-12345 aparece en la tabla
- ✅ Información visible:
  - Número de factura
  - Emisor
  - Receptor
  - Monto
  - Fecha emisión
  - Estado
  - CUFE (parcial o completo)
- ✅ Acciones disponibles:
  - Ver PDF
  - Ver XML
  - Ver detalle
  - Descargar documentos

---

### PF-004-22: Ver detalle completo de factura

**Descripción**: Verificar que se puede acceder al detalle completo de la factura.

**Datos de Entrada**:
- Factura: SETP-12345

**Pasos**:
1. Desde la lista, hacer clic en "Ver detalle"

**Resultado Esperado**:
- ✅ Se abre pantalla de detalle con:
  - Todos los datos de la factura
  - Información del emisor
  - Información del receptor
  - Montos (total, retenciones, anticipos)
  - Fechas (emisión, vencimiento, carga)
  - CUFE completo
  - Usuario que cargó
  - Estado actual
  - Enlaces a PDF y XML
  - Opción de descargar documentos
  - Historial de acciones (auditoría)

---

### PF-004-23: Descargar PDF de la factura

**Descripción**: Verificar que se puede descargar el PDF.

**Datos de Entrada**:
- Factura con PDF disponible

**Pasos**:
1. Ir al detalle de la factura
2. Hacer clic en "Descargar PDF" o ícono de descarga

**Resultado Esperado**:
- ✅ Se descarga archivo PDF
- ✅ El PDF es válido y se puede abrir
- ✅ El contenido corresponde a la factura

---

### PF-004-24: Descargar XML de la factura

**Descripción**: Verificar que se puede descargar el XML original.

**Datos de Entrada**:
- Factura cargada

**Pasos**:
1. Ir al detalle
2. Hacer clic en "Descargar XML"

**Resultado Esperado**:
- ✅ Se descarga archivo XML
- ✅ El XML descargado es idéntico al cargado originalmente
- ✅ El XML es válido y se puede abrir

---

### PF-004-25: Filtrar facturas por emisor

**Descripción**: Verificar que se pueden filtrar facturas por emisor.

**Datos de Entrada**:
- Múltiples facturas de diferentes emisores en el sistema

**Pasos**:
1. Ir a lista de facturas
2. Aplicar filtro por emisor: "CADENA S.A."

**Resultado Esperado**:
- ✅ Se muestran solo facturas donde emisor = "CADENA S.A."
- ✅ Las demás facturas se ocultan
- ✅ Contador indica cantidad de facturas filtradas

---

### PF-004-26: Filtrar facturas por receptor

**Descripción**: Verificar filtrado por receptor.

**Datos de Entrada**:
- Filtro: Receptor = "PRESIZA S.A.S"

**Pasos**:
1. Aplicar filtro por receptor

**Resultado Esperado**:
- ✅ Se muestran solo facturas del receptor seleccionado
- ✅ Funciona correctamente

---

### PF-004-27: Filtrar por rango de fechas

**Descripción**: Verificar filtrado por fechas de emisión.

**Datos de Entrada**:
- Rango: 2025-12-01 a 2025-12-31

**Pasos**:
1. Aplicar filtro de fechas

**Resultado Esperado**:
- ✅ Se muestran solo facturas emitidas en ese rango
- ✅ Funciona correctamente

---

### PF-004-28: Buscar por CUFE

**Descripción**: Verificar búsqueda por CUFE.

**Datos de Entrada**:
- CUFE: abc123def456...

**Pasos**:
1. Ingresar CUFE en buscador
2. Buscar

**Resultado Esperado**:
- ✅ Se encuentra la factura con ese CUFE
- ✅ Se muestra directamente el detalle
- ✅ Búsqueda es rápida (< 1 segundo)

---

### PF-004-29: Buscar por número de factura

**Descripción**: Verificar búsqueda por número.

**Datos de Entrada**:
- Número: SETP-12345

**Pasos**:
1. Buscar por número

**Resultado Esperado**:
- ✅ Se encuentra la factura
- ✅ Si hay múltiples con mismo prefijo, se muestran todas

---

### PF-004-30: Registro de auditoría completo - Carga exitosa

**Descripción**: Verificar que se crea registro de auditoría completo para carga exitosa.

**Datos de Entrada**:
- Factura cargada exitosamente

**Pasos**:
1. Cargar factura
2. Consultar tabla de auditoría

**Resultado Esperado**:
- ✅ Registro creado en `auditoria_carga_facturas`
- ✅ Campos poblados:
  - factura_id: UUID de la factura
  - cufe: CUFE del documento
  - accion: "CARGA_FACTURA_EXTERNA"
  - usuario: NIT del usuario
  - resultado: "EXITOSO"
  - detalles: JSON con información completa
  - pasos_ejecutados: Array con todos los pasos
  - tiempo_procesamiento_ms: Tiempo total
  - ip_usuario: IP del usuario
  - fecha_hora: Timestamp

---

### PF-004-31: Registro de auditoría - Carga fallida

**Descripción**: Verificar auditoría cuando la carga falla.

**Datos de Entrada**:
- Factura con error (ej: certificado no corresponde)

**Pasos**:
1. Intentar cargar factura que falla

**Resultado Esperado**:
- ✅ Registro de auditoría creado
- ✅ resultado: "ERROR"
- ✅ error_mensaje: Descripción del error
- ✅ pasos_ejecutados: Solo los pasos que se completaron antes del error
- ✅ detalles: Información del error

---

### PF-004-32: Manejo de timeout con DIAN

**Descripción**: Verificar que se manejan timeouts con DIAN correctamente.

**Datos de Entrada**:
- XML válido
- DIAN tarda más de 30 segundos en responder

**Pasos**:
1. Cargar factura
2. Simular timeout en consulta DIAN

**Resultado Esperado**:
- ✅ Sistema espera hasta timeout configurado
- ✅ Después de timeout, reintenta hasta 3 veces
- ✅ Si fallan los 3 intentos:
  - Advertencia: "No se pudo obtener PDF de DIAN (timeout)"
  - Continúa sin PDF
  - pdf_pendiente = true
- ✅ El procesamiento no se detiene

---

### PF-004-33: Validar estructura XSD del AttachedDocument

**Descripción**: Verificar que el XML cumple con el XSD de AttachedDocument.

**Datos de Entrada**:
- XML con estructura que no cumple XSD de DIAN

**Pasos**:
1. Cargar XML con estructura incorrecta

**Resultado Esperado**:
- ✅ Error: "El XML no cumple con el esquema AttachedDocument de DIAN"
- ✅ Detalle del error de validación XSD
- ✅ No se procesa

---

### PF-004-34: Cargar factura con moneda diferente a COP

**Descripción**: Verificar que se soportan facturas en monedas diferentes.

**Datos de Entrada**:
- XML con document_currency_code: USD
- Monto: 1,500 USD

**Pasos**:
1. Cargar factura en USD

**Resultado Esperado**:
- ✅ Factura se procesa correctamente
- ✅ Moneda almacenada: "USD"
- ✅ Monto se muestra con formato correcto: $1,500.00 USD
- ✅ En lista de facturas se distingue la moneda

---

### PF-004-35: Validar longitud del CUFE

**Descripción**: Verificar que el CUFE tiene la longitud esperada.

**Datos de Entrada**:
- XML con CUFE muy corto o muy largo (fuera de estándar)

**Pasos**:
1. Cargar XML con CUFE inválido

**Resultado Esperado**:
- ✅ Error: "El CUFE no tiene el formato válido"
- ✅ No se procesa

---

### PF-004-36: Cargar múltiples facturas secuencialmente

**Descripción**: Verificar que se pueden cargar múltiples facturas una tras otra.

**Datos de Entrada**:
- 3 archivos XML diferentes

**Pasos**:
1. Cargar factura 1
2. Esperar a que termine
3. Cargar factura 2
4. Esperar a que termine
5. Cargar factura 3

**Resultado Esperado**:
- ✅ Las 3 facturas se procesan exitosamente
- ✅ No hay conflictos entre cargas
- ✅ Cada factura tiene su propio registro independiente
- ✅ Las 3 aparecen en la lista

---

### PF-004-37: Verificar performance con archivo XML grande

**Descripción**: Verificar que el sistema maneja XMLs grandes eficientemente.

**Datos de Entrada**:
- XML de 5 MB (con muchos ítems de línea)

**Pasos**:
1. Cargar XML grande

**Resultado Esperado**:
- ✅ El archivo se procesa sin timeout
- ✅ Tiempo de procesamiento < 10 segundos
- ✅ No hay errores de memoria
- ✅ El sistema sigue siendo responsive

---

### PF-004-38: Validar caracteres especiales en campos de texto

**Descripción**: Verificar que se manejan correctamente caracteres especiales.

**Datos de Entrada**:
- XML con razón social: "EMPRESA & CIA S.A.S."
- Con caracteres: &, ñ, á, é, í, ó, ú

**Pasos**:
1. Cargar factura

**Resultado Esperado**:
- ✅ Caracteres especiales se procesan correctamente
- ✅ Se almacenan correctamente en BD
- ✅ Se muestran correctamente en la interfaz
- ✅ No hay caracteres corruptos

---

### PF-004-39: Ver XML original en el navegador

**Descripción**: Verificar que se puede visualizar el XML directamente.

**Datos de Entrada**:
- Factura cargada

**Pasos**:
1. Ir al detalle
2. Hacer clic en "Ver XML"

**Resultado Esperado**:
- ✅ Se abre el XML en el navegador o en una modal
- ✅ El XML está formateado y es legible
- ✅ Se puede copiar el contenido
- ✅ Opción de descargar desde la vista

---

### PF-004-40: Verificar índices en MongoDB para búsquedas rápidas

**Descripción**: Verificar que las búsquedas son rápidas gracias a los índices.

**Datos de Entrada**:
- 1000 facturas cargadas en MongoDB

**Pasos**:
1. Buscar por CUFE
2. Buscar por NIT emisor
3. Buscar por fecha

**Resultado Esperado**:
- ✅ Todas las búsquedas son rápidas (< 500ms)
- ✅ Los índices están creados:
  - cufe (unique)
  - campos.document_number
  - campos.client_number
  - campos.debtor_number
  - campos.document_date
  - metadata.estado

---

## Matriz de Cobertura

| Escenario de Aceptación | Casos de Prueba Relacionados | Estado |
|--------------------------|------------------------------|--------|
| 1. Cargar XML | PF-004-01, PF-004-02, PF-004-03 | ☐ |
| 2. Validar certificado digital | PF-004-04, PF-004-05, PF-004-06 | ☐ |
| 3. Validar firma digital | PF-004-07, PF-004-08 | ☐ |
| 4. Validar reglas de negocio | PF-004-09, PF-004-10 | ☐ |
| 5. Extraer campos | PF-004-11, PF-004-12 | ☐ |
| 6. Consultar PDF en DIAN | PF-004-13, PF-004-14, PF-004-15, PF-004-32 | ☐ |
| 7. Guardar PDF | PF-004-13, PF-004-23 | ☐ |
| 8. Guardar XML en MongoDB | PF-004-16, PF-004-17 | ☐ |
| 9. Guardar en BD relacional | PF-004-18 | ☐ |
| 10. Enviar notificación | PF-004-20 | ☐ |
| 11. Notificar resultado | PF-004-01, PF-004-04 (errores) | ☐ |
| 12. Validar CUFE duplicado | PF-004-19 | ☐ |
| 13. Manejo de errores DIAN | PF-004-14, PF-004-32 | ☐ |
| 14. Visualizar facturas | PF-004-21, PF-004-22 | ☐ |
| 15. Validar estructura XML | PF-004-02, PF-004-33 | ☐ |
| 16. Auditoría | PF-004-30, PF-004-31 | ☐ |
| Búsqueda y filtros | PF-004-25 a PF-004-29 | ☐ |
| Descargas | PF-004-23, PF-004-24, PF-004-39 | ☐ |
| Performance | PF-004-37, PF-004-40 | ☐ |

---

## Datos de Prueba Sugeridos

### Archivos XML de Prueba

**1. factura_valida_001.xml**
- CUFE único
- Todos los campos completos
- Firma digital válida
- NIT emisor: 890930534
- NIT receptor: 901453011
- Monto: $1,500,000 COP

**2. factura_valida_002.xml**
- Diferente CUFE
- Moneda: USD
- Monto: $2,500 USD

**3. factura_sin_firma.xml**
- XML válido pero sin firma digital (para pruebas de error)

**4. factura_xml_malformado.xml**
- XML con errores de sintaxis

**5. factura_cufe_duplicado.xml**
- Mismo CUFE que factura_valida_001.xml

**6. factura_monto_bajo.xml**
- Monto: $50,000 COP (para probar regla de monto mínimo)

### Certificados Digitales de Prueba

| NIT | Nombre | Uso |
|-----|--------|-----|
| 890930534 | CADENA S.A. | Emisor - Pruebas válidas |
| 901453011 | PRESIZA S.A.S | Receptor - Pruebas válidas |
| 800123456 | EMPRESA NO RELACIONADA | Pruebas de error (NIT no corresponde) |

### Datos en BD Relacional (Pre-cargados)

**Factura existente para pruebas de duplicado:**
- CUFE: test-cufe-123456789
- Número: SETP-999
- Estado: Cargada
- Fecha carga: 2025-12-01

---

## Criterios de Aceptación de Pruebas

- ✅ Todos los casos de prueba deben pasar exitosamente
- ✅ La integración con DIAN funciona correctamente
- ✅ Las validaciones de certificado y firma son estrictas
- ✅ Todos los campos se extraen correctamente
- ✅ El manejo de errores es robusto
- ✅ Los registros se guardan correctamente en ambas BDs
- ✅ La auditoría es completa
- ✅ No hay errores en consola del navegador
- ✅ La cobertura de código debe ser >= 80%
- ✅ Las pruebas deben pasar en: Chrome, Firefox, Safari, Edge
- ✅ El tiempo de procesamiento de una factura < 5 segundos (con DIAN disponible)
- ✅ Las búsquedas son rápidas (< 1 segundo)
- ✅ No hay pérdida de datos durante el procesamiento

---

## Configuración del Ambiente de Pruebas

**Servicios Externos**:
- DIAN E-Factura API de pruebas configurada
- Endpoints de prueba disponibles
- Certificados digitales de prueba instalados

**Bases de Datos**:
- MongoDB con índices creados
- BD relacional con tablas y constraints
- Datos de prueba pre-cargados

**Almacenamiento**:
- Sistema de archivos o cloud storage configurado para PDFs

**Herramientas**:
- Generador de certificados digitales de prueba
- Herramientas para firmar XMLs de prueba
- Validador de XMLs contra XSD de DIAN

---

## Notas de Ejecución

**Ambiente de Pruebas**: [Especificar URL]

**DIAN API**: [URL del ambiente de pruebas de DIAN]

**MongoDB**: [Instancia de pruebas]

**BD Relacional**: [Instancia de pruebas]

**Certificados**: Ubicación de certificados de prueba

**Fecha de Ejecución**: _______________________

**Ejecutado por**: _______________________

**Observaciones**:
_________________________________________________________________
_________________________________________________________________
_________________________________________________________________

---

**Última actualización**: 2025-12-09
