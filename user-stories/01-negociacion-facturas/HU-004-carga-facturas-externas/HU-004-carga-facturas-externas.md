# Carga de Facturas Electrónicas Externas desde DIAN

**By:** Melissa Arcila Restrepo

---

## Información General

| Campo | Valor |
|-------|-------|
| **Release objetivo** | v1.0 |
| **Epic** | Negociación de Facturas |
| **Estado** | BORRADOR |
| **Propietario** | |
| **Arquitecto** | |
| **Desarrolladores** | |
| **Tester** | |
| **Incidentes relacionados** | |
| **Arquitectura relacionada** | Integración con DIAN E-Factura |
| **Documento de referencia** | TDIEEFFact-Funcional Carga de Facturas Externas-020226-141802.pdf |

---

## Contexto

### Enunciado general de la historia

Como **Factor** (empresa de factoring), quiero poder **cargar facturas electrónicas externas** desde archivos XML (AttachedDocument) que cumplen con el estándar de e-facturación de la DIAN, para que el sistema valide, procese y almacene la información de la factura junto con su PDF en la base de datos, permitiendo la gestión y negociación de estas facturas en el sistema de factoring.

### Roles
- **Factor**: Empresa de factoring que carga las facturas electrónicas
- **Axces**: Sistema de factoring que procesa y almacena las facturas
- **DIAN E-Factura API**: Servicio externo para consultar y validar facturas electrónicas
- **Cliente (Emisor/Receptor)**: Empresa que emite o recibe la factura electrónica

### Característica / Funcionalidad
Carga, validación y almacenamiento de facturas electrónicas externas desde XML (AttachedDocument) con consulta automática del PDF a la DIAN y almacenamiento en base de datos.

### Razón / Resultado
Permitir que el sistema de factoring pueda procesar facturas electrónicas oficiales de la DIAN, garantizando la autenticidad mediante la validación del certificado digital, obteniendo automáticamente el PDF oficial desde la DIAN, y almacenando toda la información estructurada necesaria para la gestión y negociación de facturas.

---

## Escenarios

| Número | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|--------|--------------------------------|----------|--------|-------------------------------------|------------|----|--------------|
| 1 | Cargar archivo XML (AttachedDocument) | El Factor accede a la funcionalidad de carga de facturas externas | El Factor selecciona y carga un archivo XML (AttachedDocument) | El sistema:<br>1. Valida que el archivo es formato XML válido<br>2. Valida que es un AttachedDocument<br>3. Valida la estructura del documento según estándar DIAN<br>4. Muestra indicador de carga/procesamiento<br>5. Extrae información básica del documento<br>6. Procede con las validaciones de reglas de negocio | ☐ | ☐ | ☐ |
| 2 | Validar certificado digital del usuario | El sistema recibe el archivo XML y el usuario está autenticado por certificado digital | El sistema procesa el AttachedDocument | El sistema valida que:<br>- El usuario está autenticado por certificado digital<br>- El NIT del certificado corresponde al NIT de la empresa emisora o receptora del UBL consultado<br>- Si la validación falla:<br>  * Error: "El certificado digital no corresponde al emisor o receptor de la factura"<br>  * No se procesa la factura<br>- Si la validación es exitosa:<br>  * Continúa con el proceso | ☐ | ☐ | ☐ |
| 3 | Validar firma digital del AttachedDocument | El sistema recibe el AttachedDocument | El sistema valida la firma digital | El sistema valida que:<br>- El AttachedDocument contiene la firma digital del emisor<br>- La firma digital es válida<br>- La firma corresponde al emisor o su Proveedor Tecnológico autorizado<br>- Si la validación falla:<br>  * Error: "La firma digital del documento no es válida"<br>  * No se procesa<br>- Si es válida:<br>  * Continúa el proceso | ☐ | ☐ | ☐ |
| 4 | Validar reglas de negocio | El AttachedDocument ha pasado las validaciones de certificado y firma | El sistema valida reglas de negocio (definidas por el cliente) | El sistema:<br>- Ejecuta las validaciones de reglas de negocio configuradas<br>- Ejemplos de reglas:<br>  * Monto mínimo/máximo<br>  * NITs permitidos<br>  * Tipos de documentos aceptados<br>  * Fechas válidas<br>- Si no cumple reglas:<br>  * Error específico: "No cumple regla: [descripción]"<br>  * No se procesa<br>- Si cumple:<br>  * Continúa al siguiente paso | ☐ | ☐ | ☐ |
| 5 | Extraer campos requeridos del XML | El AttachedDocument cumple todas las validaciones | El sistema parsea el XML | El sistema extrae los siguientes campos del XML:<br><br>**Campos obligatorios**:<br>- document_prefix (prefix)<br>- document_number (documentId)<br>- document_type_id (documentTypeCode)<br>- cufe (uuid) - CUFE del documento<br>- client_number (supplierNit) - NIT emisor<br>- debtor_number (receiverNit) - NIT receptor<br>- partnership_id (partnershipId) - NIT alianza<br>- document_date (documentDate)<br>- document_amount (total)<br>- due_date (paymentDueDate)<br>- withholding_tax_amount (withholdingTaxAmount)<br>- paid_amount (paidAmount)<br>- payment_means_id (paymentMeansId)<br>- document_currency_code (documentCurrencyCode)<br><br>Si algún campo obligatorio falta:<br>- Error: "Campo requerido faltante: [nombre_campo]"<br>- No se procesa | ☐ | ☐ | ☐ |
| 6 | Consultar PDF de la factura en DIAN | Los campos han sido extraídos exitosamente | El sistema consulta el PDF ante la DIAN | El sistema:<br>1. Consume el servicio de DIAN para obtener el PDF de la factura<br>2. Usa el CUFE/UUID del documento para la consulta<br>3. Incluye autenticación necesaria<br>4. Espera respuesta de la DIAN<br>5. Si se obtiene el PDF:<br>   - Descarga el archivo PDF<br>   - Valida que es un PDF válido<br>   - Genera URL temporal o referencia<br>6. Si NO se obtiene el PDF:<br>   - Registra advertencia en log<br>   - Continúa el proceso sin PDF<br>   - Marca campo pdf_url como null | ☐ | ☐ | ☐ |
| 7 | Guardar PDF en base de datos del Factor | El PDF fue obtenido exitosamente de la DIAN | El sistema almacena el PDF | El sistema:<br>- Guarda el archivo PDF en el almacenamiento del Factor<br>- Puede ser:<br>  * Base de datos (BLOB)<br>  * Sistema de archivos<br>  * Almacenamiento en la nube (S3, Azure Blob, etc.)<br>- Genera URL permanente para acceder al PDF<br>- Actualiza el campo pdf_url con la URL generada<br>- Si falla el guardado:<br>  * Error: "No se pudo guardar el PDF"<br>  * Se registra en log pero no detiene el proceso | ☐ | ☐ | ☐ |
| 8 | Guardar XML (AttachedDocument) en MongoDB | Los datos han sido procesados | El sistema almacena el XML en MongoDB | El sistema:<br>- Guarda el AttachedDocument completo en MongoDB (E-Factura)<br>- Estructura del documento en Mongo:<br>  * XML original completo<br>  * Campos extraídos (documento plano para consultas)<br>  * Metadata (fecha de carga, usuario, estado)<br>  * Referencias (pdf_url, xml_url)<br>- Genera xml_url con referencia al documento en Mongo<br>- Si falla:<br>  * Error crítico: "No se pudo guardar la factura en base de datos"<br>  * Rollback de operaciones previas<br>  * No se completa la carga | ☐ | ☐ | ☐ |
| 9 | Guardar factura en base de datos relacional (Axces) | El XML se guardó exitosamente en MongoDB | El sistema almacena los datos extraídos en BD relacional | El sistema:<br>- Inserta registro en tabla de facturas con todos los campos extraídos:<br>  * document_prefix<br>  * document_number<br>  * document_type_id<br>  * cufe<br>  * client_number<br>  * debtor_number<br>  * partnership_id<br>  * document_date<br>  * document_amount<br>  * due_date<br>  * withholding_tax_amount<br>  * paid_amount<br>  * pdf_url<br>  * xml_url<br>  * payment_means_id<br>  * document_currency_code<br>  * fecha_carga<br>  * estado (ej: "Cargada", "Pendiente")<br>- Establece relaciones con tablas de emisores/receptores<br>- Si falla:<br>  * Error crítico<br>  * Rollback completo | ☐ | ☐ | ☐ |
| 10 | Enviar factura (XML AD) al buzón del Factor | La factura se guardó exitosamente en todas las bases de datos | El sistema notifica al buzón del Factor | El sistema:<br>- Envía notificación al buzón del Factor indicando nueva factura<br>- Incluye información resumida:<br>  * Número de factura<br>  * Emisor<br>  * Receptor<br>  * Monto<br>  * CUFE<br>  * Estado de procesamiento<br>- Puede ser:<br>  * Notificación en la aplicación<br>  * Email<br>  * Webhook<br>- Si falla la notificación:<br>  * Se registra en log<br>  * No afecta el guardado de la factura | ☐ | ☐ | ☐ |
| 11 | Notificar al Factor resultado del procesamiento | Todo el proceso ha finalizado (exitoso o con error) | El sistema muestra resultado al usuario | El sistema muestra:<br><br>**Si es exitoso**:<br>- ✅ Mensaje: "Factura cargada exitosamente"<br>- Resumen:<br>  * Número de factura<br>  * Emisor<br>  * Receptor<br>  * Monto<br>  * Estado: "Cargada"<br>  * Enlace para ver PDF<br>  * Enlace para ver XML<br>- Opciones:<br>  * Ver detalle de la factura<br>  * Cargar otra factura<br>  * Ir a lista de facturas<br><br>**Si hay error**:<br>- ❌ Mensaje: "Error al cargar la factura"<br>- Detalle del error<br>- Sugerencias de corrección<br>- Opción de intentar nuevamente | ☐ | ☐ | ☐ |
| 12 | Validar que el CUFE no esté duplicado | El sistema extrae el CUFE del documento | Antes de guardar, se verifica unicidad | El sistema:<br>- Consulta en la base de datos si el CUFE ya existe<br>- Si existe:<br>  * Error: "La factura con CUFE [cufe] ya fue cargada"<br>  * Muestra información de la factura existente:<br>    - Fecha de carga anterior<br>    - Usuario que la cargó<br>    - Estado actual<br>  * Opción: Ver factura existente<br>  * No se procesa duplicado<br>- Si no existe:<br>  * Continúa con el guardado | ☐ | ☐ | ☐ |
| 13 | Manejo de errores de conexión con DIAN | El sistema intenta consultar el PDF en DIAN | Ocurre error de conexión o timeout | El sistema:<br>- Detecta el error de conexión<br>- Reintenta hasta 3 veces con backoff exponencial<br>- Si falla después de 3 intentos:<br>  * Registra advertencia: "No se pudo obtener PDF de DIAN"<br>  * Marca pdf_url como null<br>  * Marca campo "pdf_pendiente" = true<br>  * Continúa el proceso de guardado<br>  * La factura se guarda pero sin PDF<br>  * Se puede programar reintento posterior<br>- No bloquea la carga de la factura | ☐ | ☐ | ☐ |
| 14 | Visualizar factura cargada | La factura fue cargada exitosamente | El usuario accede a la lista de facturas o al detalle | El sistema muestra:<br>- Todas las facturas cargadas en la tabla<br>- Filtros disponibles:<br>  * Por emisor<br>  * Por receptor<br>  * Por rango de fechas<br>  * Por monto<br>  * Por estado<br>  * Por CUFE<br>- Acciones:<br>  * Ver PDF (si disponible)<br>  * Ver XML<br>  * Ver detalle completo<br>  * Descargar documentos<br>  * Iniciar negociación (si aplica) | ☐ | ☐ | ☐ |
| 15 | Validar formato y estructura del XML | El archivo XML ha sido cargado | El sistema valida la estructura | El sistema valida:<br>- El XML es bien formado (sintaxis correcta)<br>- Contiene las etiquetas esperadas del estándar DIAN<br>- Los namespaces son correctos<br>- La estructura cumple con el XSD del AttachedDocument<br>- Si no es válido:<br>  * Error: "El archivo XML no cumple con el formato esperado"<br>  * Detalle específico del error de parseo<br>  * No se procesa | ☐ | ☐ | ☐ |
| 16 | Registro de auditoría completo | Cualquier operación de carga (exitosa o fallida) | El sistema registra la operación | El sistema crea registro de auditoría con:<br>- Usuario que realizó la carga<br>- Fecha y hora<br>- Archivo cargado (nombre original)<br>- CUFE del documento<br>- Resultado (éxito/error)<br>- Detalles del error (si aplica)<br>- Tiempo de procesamiento<br>- IP del usuario<br>- Todos los pasos ejecutados<br>- Campos extraídos<br>- Referencia a BD donde se guardó<br><br>Para trazabilidad completa del proceso | ☐ | ☐ | ☐ |

---

## Interacción con el usuario y prototipo

### Flujo completo de carga de factura:

#### 1. **Pantalla de Carga de Facturas**
```
┌──────────────────────────────────────────────┐
│  Carga de Facturas Electrónicas         [X] │
├──────────────────────────────────────────────┤
│                                              │
│  📄 Cargar Factura Electrónica (XML)        │
│                                              │
│  Seleccione el archivo AttachedDocument      │
│  (.xml) de la factura electrónica de DIAN   │
│                                              │
│  ┌────────────────────────────────┐          │
│  │ Arrastre el archivo aquí       │          │
│  │ o haga clic para seleccionar   │          │
│  └────────────────────────────────┘          │
│                                              │
│  [Seleccionar archivo XML]                   │
│                                              │
│  Requisitos:                                 │
│  • Archivo XML (AttachedDocument)           │
│  • Certificado digital válido               │
│  • Debe ser emisor o receptor de la factura │
│                                              │
└──────────────────────────────────────────────┘
```

#### 2. **Procesamiento con indicadores de progreso**
```
┌──────────────────────────────────────────────┐
│  Procesando factura...                       │
├──────────────────────────────────────────────┤
│  📄 Factura: SETP-12345                      │
│                                              │
│  ⏳ Pasos del proceso:                       │
│                                              │
│  ✅ Validando estructura XML                 │
│  ✅ Verificando certificado digital          │
│  ✅ Validando firma digital                  │
│  ✅ Validando reglas de negocio              │
│  ✅ Extrayendo campos                        │
│  🔄 Consultando PDF en DIAN...               │
│  ⏸️ Guardando en base de datos...            │
│  ⏸️ Enviando notificación...                 │
│                                              │
│  ████████████░░░░░░░░░  60%                  │
└──────────────────────────────────────────────┘
```

#### 3. **Resultado Exitoso**
```
┌──────────────────────────────────────────────┐
│  ✅ Factura cargada exitosamente        [X] │
├──────────────────────────────────────────────┤
│                                              │
│  La factura se ha procesado correctamente    │
│                                              │
│  📋 Información de la factura:               │
│  ─────────────────────────────               │
│  Número: SETP-12345                          │
│  CUFE: abc123def456...                       │
│  Emisor: CADENA S.A. (890930534)            │
│  Receptor: PRESIZA S.A.S (901453011)        │
│  Fecha emisión: 2025-12-09                   │
│  Monto: $1,500,000.00 COP                    │
│  Vencimiento: 2026-01-08                     │
│  Estado: Cargada ✓                           │
│                                              │
│  📄 Documentos disponibles:                  │
│  • [Ver PDF] [Descargar PDF]                 │
│  • [Ver XML] [Descargar XML]                 │
│                                              │
│  [Ver detalle completo]                      │
│  [Cargar otra factura]                       │
│  [Ir a lista de facturas]                    │
└──────────────────────────────────────────────┘
```

#### 4. **Error de Validación**
```
┌──────────────────────────────────────────────┐
│  ❌ Error al cargar la factura          [X] │
├──────────────────────────────────────────────┤
│                                              │
│  No se pudo procesar la factura              │
│                                              │
│  ⚠️ Error encontrado:                        │
│  El certificado digital no corresponde al    │
│  emisor o receptor de la factura             │
│                                              │
│  Detalles:                                   │
│  • Su NIT: 800123456                         │
│  • NIT emisor: 890930534                     │
│  • NIT receptor: 901453011                   │
│                                              │
│  💡 Sugerencia:                              │
│  Verifique que está utilizando el            │
│  certificado digital correcto que             │
│  corresponda al emisor o receptor de la       │
│  factura.                                    │
│                                              │
│  [Intentar nuevamente]                       │
│  [Cancelar]                                  │
└──────────────────────────────────────────────┘
```

#### 5. **Error - Factura Duplicada**
```
┌──────────────────────────────────────────────┐
│  ⚠️ Factura ya existe                   [X] │
├──────────────────────────────────────────────┤
│                                              │
│  Esta factura ya fue cargada anteriormente   │
│                                              │
│  CUFE: abc123def456...                       │
│  Número: SETP-12345                          │
│                                              │
│  📅 Cargada el: 2025-12-01 10:30 AM          │
│  👤 Cargada por: admin@factoring.com         │
│  📊 Estado actual: En negociación            │
│                                              │
│  ¿Qué desea hacer?                           │
│                                              │
│  [Ver factura existente]                     │
│  [Cancelar]                                  │
└──────────────────────────────────────────────┘
```

#### 6. **Lista de Facturas Cargadas**
```
┌─────────────────────────────────────────────────────────┐
│  Facturas Electrónicas Cargadas                    [+]  │
├─────────────────────────────────────────────────────────┤
│  Filtros: [Emisor ▾] [Receptor ▾] [Fechas] [Buscar]   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  No.     │ Emisor      │ Receptor   │ Monto    │ PDF  │
│  Factura │             │            │          │      │
│──────────┼─────────────┼────────────┼──────────┼──────│
│ SETP-123 │ CADENA S.A. │ PRESIZA    │ $1.5M    │ [📄] │
│ CUFE:... │ 890930534   │ 901453011  │ 09/12/25 │ [📥] │
│          │             │            │          │ [👁️] │
│──────────┼─────────────┼────────────┼──────────┼──────│
│ PR-456   │ ENTREGA DE  │ DINDELCO   │ $2.3M    │ [📄] │
│ CUFE:... │ CARGA       │ S.A.S      │ 08/12/25 │ [📥] │
│          │ 800114437   │ 900439177  │          │ [👁️] │
└─────────────────────────────────────────────────────────┘

[📄] = Ver PDF    [📥] = Descargar    [👁️] = Ver detalle
```

### Diagrama de flujo técnico:

```
┌─────────────┐
│   Factor    │
│   /Axces    │
└──────┬──────┘
       │
       │ 1. Carga XML AD
       ↓
┌──────────────────┐
│   Validaciones   │
│   iniciales      │
│  • Formato XML   │
│  • Certificado   │
│  • Firma digital │
│  • Reglas negocio│
└──────┬───────────┘
       │
       │ ¿Cumple?
       ↓
    [SI]────┐      [NO]→ Error y detener
       │    │
       │    ↓
       │  ┌──────────────┐
       │  │ Extrae campos│
       │  │  del XML     │
       │  └──────┬───────┘
       │         │
       │         ↓
       │  ┌──────────────────┐
       │  │ Consulta PDF     │
       │  │ ante DIAN        │
       │  └──────┬───────────┘
       │         │
       │         ↓
       │     ¿Se obtuvo?
       │    [SI]  [NO]
       │     │     │
       │     ↓     ↓
       │ Guarda  Continúa
       │  PDF    sin PDF
       │     │     │
       │     └──┬──┘
       │        ↓
       │  ┌─────────────────┐
       │  │ Guarda XML en   │
       │  │ MongoDB         │
       │  └──────┬──────────┘
       │         │
       │         ↓
       │  ┌─────────────────┐
       │  │ Guarda datos en │
       │  │ BD relacional   │
       │  └──────┬──────────┘
       │         │
       │         ↓
       │  ┌─────────────────┐
       │  │ Envía a buzón   │
       │  │ del Factor      │
       │  └──────┬──────────┘
       │         │
       │         ↓
       │  ┌─────────────────┐
       │  │ Notifica        │
       │  │ resultado       │
       │  └─────────────────┘
       │
       └────────────────────→ FIN
```

### Consideraciones de UX:

- **Feedback continuo**: Mostrar progreso en cada paso del procesamiento
- **Mensajes claros**: Errores específicos con sugerencias de corrección
- **Validación temprana**: Detectar problemas lo antes posible
- **Manejo de duplicados**: Prevenir cargas duplicadas con información clara
- **Acceso rápido**: Enlaces directos a PDF y XML desde el resumen
- **Trazabilidad**: Mostrar toda la información del procesamiento
- **Notificaciones**: Alertar al Factor cuando hay nuevas facturas

---

## Definición de Terminado (DoD)

- [ ] El código cumple con los estándares de desarrollo del proyecto
- [ ] Se han implementado todos los escenarios de aceptación
- [ ] La integración con DIAN E-Factura funciona correctamente
- [ ] Las validaciones de certificado digital y firma funcionan
- [ ] La extracción de todos los campos requeridos es correcta
- [ ] El PDF se descarga y almacena correctamente
- [ ] El XML se guarda en MongoDB con estructura correcta
- [ ] Los datos se guardan en BD relacional correctamente
- [ ] El sistema maneja correctamente errores de conexión con DIAN
- [ ] La validación de duplicados (CUFE) funciona
- [ ] El registro de auditoría es completo
- [ ] Las pruebas unitarias tienen una cobertura mínima del 80%
- [ ] Las pruebas de integración con servicios externos funcionan
- [ ] La documentación técnica está actualizada
- [ ] El Product Owner ha validado la funcionalidad
- [ ] No existen bugs críticos pendientes

---

## Notas Técnicas

### Mapeo de Campos (Axces ↔ E-Factura):

| Campo en Axces | Campo en E-Factura | Tipo | Requerido | Descripción |
|----------------|-------------------|------|-----------|-------------|
| document_prefix | prefix | String | Sí | Prefijo del documento |
| document_number | documentId | String | Sí | Número de documento |
| document_type_id | documentTypeCode | String | Sí | Tipo de documento (ej: 01 = Factura) |
| cufe | uuid | String | Sí | CUFE del documento (único) |
| client_number | supplierNit | String | Sí | NIT del emisor de la factura |
| debtor_number | receiverNit | String | Sí | NIT del receptor de la factura |
| partnership_id | partnershipId | String | Sí | NIT de la alianza |
| document_date | documentDate | Date | Sí | Fecha de emisión |
| document_amount | total | Decimal | Sí | Valor total del documento |
| due_date | paymentDueDate | Date | Sí | Fecha de vencimiento |
| withholding_tax_amount | withholdingTaxAmount | Decimal | Sí | Valor total de retenciones |
| paid_amount | paidAmount | Decimal | Sí | Valor anticipos |
| pdf_url | pdfUrl | String | No | Enlace al PDF |
| xml_url | xmlUrl | String | No | Enlace al XML (MongoDB) |
| payment_means_id | paymentMeansId | String | Sí | Forma de pago |
| document_currency_code | documentCurrencyCode | String | Sí | Moneda del documento (COP, USD, etc.) |

### Endpoints API Sugeridos:

#### Cargar factura externa
```
POST /api/facturas/externas/cargar
Content-Type: multipart/form-data

Headers:
- Authorization: Bearer {token}
- X-Client-Certificate: {certificado_digital}
```

**Request**:
```
file: [archivo XML AttachedDocument]
```

**Response (Éxito)**:
```json
{
  "success": true,
  "message": "Factura cargada exitosamente",
  "data": {
    "facturaId": "uuid",
    "cufe": "abc123def456...",
    "documentNumber": "SETP-12345",
    "emisor": {
      "nit": "890930534",
      "razonSocial": "CADENA S.A."
    },
    "receptor": {
      "nit": "901453011",
      "razonSocial": "PRESIZA S.A.S"
    },
    "monto": 1500000.00,
    "moneda": "COP",
    "fechaEmision": "2025-12-09",
    "fechaVencimiento": "2026-01-08",
    "pdfUrl": "https://storage.axces.com/pdfs/uuid.pdf",
    "xmlUrl": "https://api.axces.com/facturas/xml/uuid",
    "estado": "Cargada"
  },
  "procesamiento": {
    "pasos_exitosos": [
      "validacion_xml",
      "validacion_certificado",
      "validacion_firma",
      "extraccion_campos",
      "consulta_pdf_dian",
      "guardado_pdf",
      "guardado_xml_mongo",
      "guardado_bd_relacional",
      "notificacion_buzon"
    ],
    "tiempo_total_ms": 3500
  }
}
```

**Response (Error)**:
```json
{
  "success": false,
  "error": {
    "code": "INVALID_CERTIFICATE",
    "message": "El certificado digital no corresponde al emisor o receptor de la factura",
    "details": {
      "nit_certificado": "800123456",
      "nit_emisor": "890930534",
      "nit_receptor": "901453011"
    }
  },
  "paso_fallido": "validacion_certificado"
}
```

#### Consultar PDF en DIAN
```
GET /api/dian/pdf/{cufe}
```

**Response**:
```
Binary PDF file
Content-Type: application/pdf
```

#### Obtener detalle de factura cargada
```
GET /api/facturas/{facturaId}
```

**Response**:
```json
{
  "id": "uuid",
  "cufe": "abc123...",
  "documentPrefix": "SETP",
  "documentNumber": "12345",
  "emisor": {...},
  "receptor": {...},
  "monto": 1500000.00,
  "moneda": "COP",
  "fechaEmision": "2025-12-09",
  "fechaVencimiento": "2026-01-08",
  "fechaCarga": "2025-12-09T10:30:00Z",
  "usuarioCarga": "admin@factoring.com",
  "estado": "Cargada",
  "pdfUrl": "...",
  "xmlUrl": "...",
  "todosLosCampos": {...}
}
```

### Lógica de Procesamiento:

```javascript
async function procesarFacturaExterna(xmlFile, userCertificate) {
  const resultado = {
    exitoso: false,
    pasos: [],
    errores: [],
    datos: null
  };

  try {
    // 1. Validar formato XML
    const xml = await parseXML(xmlFile);
    resultado.pasos.push('parse_xml_ok');

    // 2. Validar es AttachedDocument
    if (!esAttachedDocument(xml)) {
      throw new Error('No es un AttachedDocument válido');
    }
    resultado.pasos.push('validacion_attached_document_ok');

    // 3. Validar certificado digital
    const nitUsuario = extraerNITDeCertificado(userCertificate);
    const { supplierNit, receiverNit } = extraerNITsDelXML(xml);

    if (nitUsuario !== supplierNit && nitUsuario !== receiverNit) {
      throw new Error('Certificado no corresponde al emisor o receptor');
    }
    resultado.pasos.push('validacion_certificado_ok');

    // 4. Validar firma digital
    const firmaValida = await validarFirmaDigital(xml);
    if (!firmaValida) {
      throw new Error('Firma digital inválida');
    }
    resultado.pasos.push('validacion_firma_ok');

    // 5. Validar reglas de negocio
    const cumpleReglas = await validarReglasNegocio(xml);
    if (!cumpleReglas.valido) {
      throw new Error(`No cumple regla: ${cumpleReglas.razon}`);
    }
    resultado.pasos.push('validacion_reglas_ok');

    // 6. Extraer campos
    const campos = await extraerCamposRequeridos(xml);
    resultado.pasos.push('extraccion_campos_ok');

    // 7. Validar CUFE no duplicado
    const existente = await buscarPorCUFE(campos.cufe);
    if (existente) {
      throw new Error(`Factura duplicada. CUFE: ${campos.cufe}`);
    }
    resultado.pasos.push('validacion_cufe_ok');

    // 8. Consultar PDF en DIAN
    let pdfUrl = null;
    try {
      const pdf = await consultarPDFEnDIAN(campos.cufe);
      pdfUrl = await guardarPDF(pdf, campos.cufe);
      resultado.pasos.push('pdf_obtenido_ok');
    } catch (error) {
      console.warn('No se pudo obtener PDF de DIAN:', error);
      resultado.pasos.push('pdf_no_disponible');
      // Continuar sin PDF
    }

    // 9. Guardar XML en MongoDB
    const xmlUrl = await guardarXMLEnMongo({
      xml: xml.toString(),
      campos: campos,
      metadata: {
        fechaCarga: new Date(),
        usuario: nitUsuario,
        pdfUrl: pdfUrl
      }
    });
    resultado.pasos.push('guardado_mongo_ok');

    // 10. Guardar en BD relacional
    const facturaId = await guardarEnBDRelacional({
      ...campos,
      pdf_url: pdfUrl,
      xml_url: xmlUrl,
      fecha_carga: new Date(),
      usuario_carga: nitUsuario,
      estado: 'Cargada'
    });
    resultado.pasos.push('guardado_bd_ok');

    // 11. Notificar al buzón
    await notificarBuzonFactor(facturaId, campos);
    resultado.pasos.push('notificacion_ok');

    // 12. Auditoría
    await registrarAuditoria({
      accion: 'CARGA_FACTURA_EXTERNA',
      usuario: nitUsuario,
      facturaId: facturaId,
      cufe: campos.cufe,
      resultado: 'EXITOSO',
      pasos: resultado.pasos,
      tiempo: Date.now() - inicioTiempo
    });

    resultado.exitoso = true;
    resultado.datos = { facturaId, ...campos, pdfUrl, xmlUrl };

  } catch (error) {
    resultado.errores.push(error.message);

    // Auditoría de error
    await registrarAuditoria({
      accion: 'CARGA_FACTURA_EXTERNA',
      usuario: nitUsuario,
      resultado: 'ERROR',
      error: error.message,
      pasos: resultado.pasos,
      tiempo: Date.now() - inicioTiempo
    });
  }

  return resultado;
}
```

### Validaciones de Seguridad:

```javascript
// Validar certificado digital
function validarCertificadoDigital(certificado, nitEmisor, nitReceptor) {
  // 1. Verificar que el certificado es válido
  if (!certificado || !certificado.isValid()) {
    throw new SecurityError('Certificado digital inválido');
  }

  // 2. Extraer NIT del certificado
  const nitCertificado = certificado.getSubject().organizationIdentifier;

  // 3. Validar que corresponde al emisor o receptor
  if (nitCertificado !== nitEmisor && nitCertificado !== nitReceptor) {
    throw new SecurityError(
      `El NIT del certificado (${nitCertificado}) no corresponde al emisor (${nitEmisor}) ni al receptor (${nitReceptor})`
    );
  }

  return true;
}

// Validar firma digital del AttachedDocument
async function validarFirmaDigital(xml) {
  // 1. Extraer firma del documento
  const firma = xml.querySelector('ds:Signature');

  if (!firma) {
    throw new SecurityError('El documento no contiene firma digital');
  }

  // 2. Validar firma con librería de validación
  const validador = new XMLDSigValidator();
  const esValida = await validador.validate(firma, xml);

  if (!esValida) {
    throw new SecurityError('La firma digital no es válida');
  }

  // 3. Verificar que la firma es del emisor o proveedor tecnológico autorizado
  const emisorFirma = firma.getSignerIdentity();
  // Validar contra lista de proveedores autorizados

  return true;
}
```

### Estructura en MongoDB:

```javascript
// Colección: facturas_externas
{
  "_id": ObjectId("..."),
  "cufe": "abc123def456...",
  "xml_original": "<AttachedDocument>...</AttachedDocument>",
  "campos": {
    "document_prefix": "SETP",
    "document_number": "12345",
    "document_type_id": "01",
    "client_number": "890930534",
    "debtor_number": "901453011",
    "partnership_id": "900123456",
    "document_date": ISODate("2025-12-09"),
    "document_amount": 1500000.00,
    "due_date": ISODate("2026-01-08"),
    "withholding_tax_amount": 75000.00,
    "paid_amount": 0,
    "payment_means_id": "1",
    "document_currency_code": "COP"
  },
  "metadata": {
    "fecha_carga": ISODate("2025-12-09T10:30:00Z"),
    "usuario_carga": "890930534",
    "pdf_url": "https://storage.axces.com/pdfs/uuid.pdf",
    "pdf_obtenido_dian": true,
    "estado": "Cargada",
    "pasos_procesamiento": [
      "validacion_xml",
      "validacion_certificado",
      "validacion_firma",
      "extraccion_campos",
      "consulta_pdf_dian",
      "guardado_pdf",
      "guardado_bd_relacional"
    ],
    "tiempo_procesamiento_ms": 3500
  },
  "auditoria": {
    "creado_en": ISODate("2025-12-09T10:30:00Z"),
    "creado_por": "890930534",
    "ip_origen": "192.168.1.100"
  }
}
```

### Índices en MongoDB:

```javascript
// Para búsquedas rápidas
db.facturas_externas.createIndex({ "cufe": 1 }, { unique: true });
db.facturas_externas.createIndex({ "campos.document_number": 1 });
db.facturas_externas.createIndex({ "campos.client_number": 1 });
db.facturas_externas.createIndex({ "campos.debtor_number": 1 });
db.facturas_externas.createIndex({ "campos.document_date": -1 });
db.facturas_externas.createIndex({ "metadata.estado": 1 });
```

### Esquema BD Relacional:

```sql
CREATE TABLE facturas_externas (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  document_prefix VARCHAR(10),
  document_number VARCHAR(50) NOT NULL,
  document_type_id VARCHAR(10) NOT NULL,
  cufe VARCHAR(255) NOT NULL UNIQUE,
  client_number VARCHAR(20) NOT NULL, -- NIT emisor
  debtor_number VARCHAR(20) NOT NULL, -- NIT receptor
  partnership_id VARCHAR(20),
  document_date DATE NOT NULL,
  document_amount DECIMAL(18,2) NOT NULL,
  due_date DATE NOT NULL,
  withholding_tax_amount DECIMAL(18,2),
  paid_amount DECIMAL(18,2),
  pdf_url TEXT,
  xml_url TEXT,
  payment_means_id VARCHAR(10),
  document_currency_code VARCHAR(3) DEFAULT 'COP',
  fecha_carga TIMESTAMP DEFAULT NOW(),
  usuario_carga VARCHAR(20),
  estado VARCHAR(50) DEFAULT 'Cargada',
  pdf_pendiente BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),

  -- Índices
  INDEX idx_cufe (cufe),
  INDEX idx_document_number (document_number),
  INDEX idx_client_number (client_number),
  INDEX idx_debtor_number (debtor_number),
  INDEX idx_document_date (document_date DESC),
  INDEX idx_estado (estado),
  INDEX idx_fecha_carga (fecha_carga DESC),

  -- Constraints
  CONSTRAINT chk_positive_amount CHECK (document_amount > 0),
  CONSTRAINT chk_valid_currency CHECK (document_currency_code IN ('COP', 'USD', 'EUR'))
);

-- Tabla de auditoría
CREATE TABLE auditoria_carga_facturas (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  factura_id UUID REFERENCES facturas_externas(id),
  cufe VARCHAR(255),
  accion VARCHAR(50) NOT NULL,
  usuario VARCHAR(20) NOT NULL,
  resultado VARCHAR(20) NOT NULL, -- EXITOSO, ERROR
  detalles JSONB,
  pasos_ejecutados TEXT[],
  error_mensaje TEXT,
  tiempo_procesamiento_ms INT,
  ip_usuario VARCHAR(45),
  fecha_hora TIMESTAMP DEFAULT NOW(),

  INDEX idx_factura_id (factura_id),
  INDEX idx_cufe (cufe),
  INDEX idx_usuario (usuario),
  INDEX idx_fecha_hora (fecha_hora DESC)
);
```

---

## Dependencias

- **Servicios Externos**:
  - DIAN E-Factura API (consulta de facturas y PDFs)
  - Servicio de validación de certificados digitales
  - Servicio de validación de firmas digitales XML

- **Infraestructura**:
  - MongoDB para almacenamiento de XMLs
  - Base de datos relacional (PostgreSQL/MySQL) para datos estructurados
  - Sistema de almacenamiento para PDFs (S3, Azure Blob, etc.)

- **Seguridad**:
  - Sistema de autenticación con certificados digitales
  - Validador de firmas digitales (XMLDSig)
  - Parser XML seguro

- **Componentes del sistema**:
  - Componente de carga de archivos
  - Sistema de notificaciones/buzón
  - Sistema de auditoría
  - Visor de PDF
  - Visor de XML

---

## Riesgos y Mitigaciones

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|--------------|---------|------------|
| DIAN no disponible al consultar PDF | Alta | Medio | Implementar reintentos con backoff; permitir carga sin PDF; programar reintento posterior |
| Certificado digital inválido o expirado | Media | Alto | Validación temprana; mensaje claro de error; guía para renovar certificado |
| XML malformado o corrupto | Media | Medio | Validación exhaustiva de estructura; mensajes de error descriptivos |
| Ataques de inyección XML (XXE) | Baja | Crítico | Usar parser XML seguro; deshabilitar entidades externas; sanitizar entrada |
| Firma digital falsificada | Baja | Crítico | Validación estricta de firmas; verificar cadena de certificados |
| Almacenamiento de PDFs consume mucho espacio | Alta | Medio | Implementar compresión; limpieza periódica de PDFs antiguos; usar almacenamiento en la nube |
| Duplicación de CUFE por error de timing | Baja | Medio | Usar constraint UNIQUE en BD; validar antes de insert |
| Timeout en procesamiento de archivos grandes | Media | Medio | Procesamiento asíncrono; indicadores de progreso; límite de tamaño de archivo |
| Pérdida de datos si falla después de guardar en Mongo | Baja | Alto | Usar transacciones distribuidas o patrón Saga; rollback en caso de error |

---

## Casos de Uso Adicionales

### Reprocesar PDF faltante
Si una factura se cargó sin PDF (por error de DIAN):
```
GET /api/facturas/{facturaId}/reprocesar-pdf
```
- Reintenta consulta a DIAN
- Actualiza pdf_url si es exitoso
- Marca pdf_pendiente = false

### Descargar XML original
```
GET /api/facturas/{facturaId}/xml/descargar
```
- Recupera XML de MongoDB
- Retorna como archivo descargable

### Validar factura después de cargada
```
POST /api/facturas/{facturaId}/revalidar
```
- Ejecuta nuevamente validaciones
- Útil si cambian reglas de negocio

---

**Fecha de creación**: 2025-12-09
**Última actualización**: 2025-12-09
