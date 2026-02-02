# Carga de Facturas Electrónicas Externas

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
| **Arquitectura relacionada** | Integración con DIAN, Sistema de Notificaciones, Buzón del Factor |
| **Documento de referencia** | DO-Funcional Fact. Externas-020226-145745.pdf |

---

## Contexto

### Enunciado general de la historia

Como **Cliente** (con perfil Client o Client Administrator), quiero poder **cargar facturas electrónicas externas** (emitidas con otro proveedor tecnológico) en formato XML (AttachedDocument) a través de la plataforma Axces, para que el sistema valide automáticamente que cumplan las reglas de negocio, verifique la información contra la DIAN, y almacene las facturas en la base de datos con origen "EXT", permitiendo su posterior negociación en el sistema de factoring.

### Roles
- **Cliente**: Empresa que emitió facturas con otro proveedor tecnológico y desea cargarlas en Axces para negociación
- **Usuario Client/Client Administrator**: Perfil con permisos para acceder al módulo "Cargar Facturas"
- **Equipo de Riesgos**: Equipo que recibe notificaciones cuando hay errores de validación con la DIAN
- **Factor**: Destinatario final de las facturas en su buzón para emitir eventos RADIAN

### Característica / Funcionalidad
Carga de facturas electrónicas externas mediante archivo XML con validaciones automáticas de reglas de negocio y verificación contra servicios API de la DIAN.

### Razón / Resultado
Permitir que los clientes puedan incorporar al sistema de factoring facturas que fueron emitidas con otros proveedores tecnológicos (no E-Factura), garantizando la validez y consistencia de la información mediante validaciones automáticas contra la DIAN antes de su almacenamiento y posterior negociación.

---

## Escenarios

| Número | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|--------|--------------------------------|----------|--------|-------------------------------------|------------|----|--------------|
| 1 | Acceso al módulo de Carga de Facturas | Usuario con perfil Client o Client Administrator accede a la plataforma Axces | Usuario selecciona la opción "Cargar Facturas" en el menú lateral | El sistema muestra la interfaz de carga con:<br>- Título: "Subir Facturas"<br>- Instrucciones para subir archivo XML (AttachedDocument)<br>- Las instrucciones incluyen:<br>  * Es obligatorio subir un XML (AttachedDocument) aprobado por la DIAN<br>  * Solo se permiten archivos en formato XML<br>  * No se reciben facturas vencidas<br>  * Las facturas deben tener fechas de vencimiento<br>- Nota: El AttachedDocument es el documento que genera la DIAN después de validar la factura<br>- Área de carga: Arrastrar y soltar o seleccionar archivo<br>- Botón "FINALIZAR" (inicialmente deshabilitado) | ☐ | ☐ | ☐ |
| 2 | Cargar archivo XML por arrastrar y soltar | Usuario está en la interfaz de carga de facturas | Usuario arrastra un archivo XML desde su escritorio/equipo y lo suelta en el área de carga | El sistema:<br>1. Acepta el archivo<br>2. Muestra el nombre del archivo cargado<br>3. Inicia automáticamente el proceso de validación de reglas<br>4. Muestra indicador de procesamiento<br>5. El proceso de validación se ejecuta de manera automática sin necesidad de que el usuario presione ningún botón adicional | ☐ | ☐ | ☐ |
| 3 | Cargar archivo XML por selección manual | Usuario está en la interfaz de carga de facturas | Usuario hace clic en el área de carga o en "Seleccionar archivo" y elige un archivo XML desde su equipo | El sistema:<br>1. Abre el explorador de archivos del sistema operativo<br>2. Permite seleccionar archivo (filtro: .xml)<br>3. Una vez seleccionado, carga el archivo automáticamente<br>4. Muestra el nombre del archivo<br>5. Inicia automáticamente el proceso de validación de reglas | ☐ | ☐ | ☐ |
| 4 | Validación 1 - Formato XML | El archivo ha sido cargado | El sistema inicia las validaciones automáticamente | El sistema valida que:<br>- El archivo sea formato XML válido<br>- Si NO es XML válido:<br>  * Estado: Rechazada ❌<br>  * Descripción: "Este archivo no es un xml"<br>  * No se procesa el archivo<br>  * Se muestra en pantalla el resultado<br>- Si es XML válido:<br>  * Continúa con la siguiente validación | ☐ | ☐ | ☐ |
| 5 | Validación 2 - Factura no existe en BD | El archivo es XML válido | El sistema valida unicidad | El sistema verifica que:<br>- La factura (por CUFE u otro identificador único) NO exista previamente en la base de datos de Axces<br>- Esto evita cargas duplicadas de facturas que ya fueron procesadas<br>- Si la factura YA existe:<br>  * Estado: Rechazada ❌<br>  * Descripción: "La factura ya existe en la base de datos de Axces, debido a que se cargó anteriormente"<br>  * No se procesa<br>- Si NO existe:<br>  * Continúa con la siguiente validación | ☐ | ☐ | ☐ |
| 6 | Validación 3 - Codificación UTF-8 | El XML es válido y no está duplicado | El sistema valida la codificación | El sistema verifica que:<br>- El archivo XML esté codificado en formato UTF-8<br>- Valida la declaración XML o la codificación real del archivo<br>- Si NO es UTF-8:<br>  * Estado: Rechazada ❌<br>  * Descripción: "El archivo XML no está codificado en formato UTF-8"<br>  * No se procesa<br>- Si es UTF-8:<br>  * Continúa con la siguiente validación | ☐ | ☐ | ☐ |
| 7 | Validación 4 - AttachedDocument de factura electrónica | El XML tiene codificación correcta | El sistema valida el tipo de documento | El sistema verifica que:<br>- El documento sea un AttachedDocument (Invoice + Application Response)<br>- Debe contener tanto la factura (Invoice) como la respuesta de aplicación (Application Response)<br>- Si NO cumple:<br>  * Estado: Rechazada ❌<br>  * Descripción: "El documento no es un AttachedDocument válido de factura electrónica"<br>  * No se procesa<br>- Si cumple:<br>  * Continúa con la siguiente validación | ☐ | ☐ | ☐ |
| 8 | Validación 5 - Factura a crédito con vencimiento mayor a 10 días | El documento es un AD válido | El sistema valida plazo de vencimiento | El sistema verifica que:<br>- La factura sea a crédito (ID = 2 en forma de pago)<br>- La fecha de vencimiento (Payment Due Date) sea superior a 10 días contados desde la fecha actual de carga<br>- Fórmula: Payment Due Date > Actual Date + 10<br>- Si NO cumple:<br>  * Estado: Rechazada ❌<br>  * Descripción: "La factura no cumple con el plazo mínimo de vencimiento (debe ser superior a 10 días desde hoy)"<br>  * No se procesa<br>- Si cumple:<br>  * Continúa con la siguiente validación | ☐ | ☐ | ☐ |
| 9 | Validación 6 - Pagador existe en BD | El plazo de vencimiento es válido | El sistema valida el pagador | El sistema verifica que:<br>- El pagador (NIT del receptor de la factura) esté creado previamente en la base de datos de Axces<br>- Busca el NIT del receptor en la tabla de pagadores/receptores<br>- Si NO existe:<br>  * Estado: Rechazada ❌<br>  * Descripción: "El pagador (NIT receptor) no está creado en la base de datos de Axces"<br>  * No se procesa<br>- Si existe:<br>  * Continúa con la siguiente validación | ☐ | ☐ | ☐ |
| 10 | Validación 7 - NIT emisor coincide con usuario | El pagador existe en BD | El sistema valida el emisor | El sistema verifica que:<br>- El NIT del emisor de la factura (en el XML) coincida con el NIT relacionado al usuario que está subiendo la factura<br>- Esto asegura que solo el emisor puede cargar sus propias facturas<br>- Si NO coincide:<br>  * Estado: Rechazada ❌<br>  * Descripción: "El NIT del emisor del documento no corresponde con el NIT de su sesión"<br>  * No se procesa<br>- Si coincide:<br>  * Continúa con validaciones de DIAN | ☐ | ☐ | ☐ |
| 11 | Validación DIAN 1 - CUFE existe en DIAN | Las validaciones básicas son exitosas | El sistema consulta API de validación de DIAN | El sistema:<br>1. Extrae el CUFE (Código Único de Factura Electrónica) del AttachedDocument<br>2. Consume el servicio de validación de CUFE de la DIAN<br>3. Verifica que:<br>   - El CUFE sea real y exista en la base de datos de la DIAN<br>   - La factura esté aprobada en la DIAN<br>4. Si la validación FALLA:<br>   - Se descarga el PDF de la DIAN (si está disponible)<br>   - Estado: Rechazada ❌<br>   - Descripción: "CUFE no encontrado en la DIAN o información inconsistente"<br>   - Se genera notificación al equipo de riesgos con:<br>     * XML subido por el cliente<br>     * Detalle del error<br>   - Se muestra notificación al usuario<br>   - No se procesa la factura<br>5. Si la validación es EXITOSA:<br>   - Continúa con validación de campos | ☐ | ☐ | ☐ |
| 12 | Validación DIAN 2 - Campos obligatorios coinciden | CUFE es válido en DIAN | El sistema compara campos entre AD del cliente y DIAN | El sistema valida que los siguientes campos coincidan entre el AD subido por el cliente y la información almacenada en la DIAN:<br><br>**Campos a validar**:<br>1. Forma de Pago<br>2. Fecha de Emisión<br>3. Fecha de Vencimiento<br>4. Nit del Receptor<br>5. Valor Total de la Factura<br>6. Valor Total Anticipos<br>7. Número de Factura<br>8. Nit del Emisor<br>9. Moneda de la Factura<br><br>Si algún campo NO coincide:<br>- Se genera notificación al equipo de riesgos conteniendo:<br>  * XML subido por el cliente<br>  * XML con información de la DIAN<br>  * Lista de coincidencias/diferencias encontradas<br>- Estado: Rechazada ❌<br>- Descripción: "CUFE no encontrado en la DIAN o información inconsistente"<br>- Se muestra notificación al usuario<br>- No se procesa<br><br>Si todos los campos coinciden:<br>- Todas las validaciones son exitosas<br>- Procede al guardado | ☐ | ☐ | ☐ |
| 13 | Mostrar resultados de validación en pantalla | Las validaciones se están ejecutando o finalizaron | El sistema procesa las validaciones | El sistema muestra en pantalla en tiempo real:<br><br>**Si hay errores**:<br>- Tabla con columnas: "NOMBRE ARCHIVO", "ESTADO", "DESCRIPCIÓN"<br>- Estado: Rechazada ❌ (círculo rojo)<br>- Descripción del error específico<br>- Ejemplos:<br>  * "Este archivo no es un xml"<br>  * "CUFE no encontrado en la DIAN o información inconsistente"<br>  * "El NIT del emisor del documento no corresponde con el NIT de su sesión"<br><br>**Si todas las validaciones son correctas**:<br>- Tabla con columnas: "NOMBRE ARCHIVO", "ESTADO", "DESCRIPCIÓN"<br>- Estado: En validación ⏳ (mientras procesa)<br>- Una vez completado: El botón "FINALIZAR" se habilita<br>- El usuario debe presionar "FINALIZAR" para confirmar el almacenamiento | ☐ | ☐ | ☐ |
| 14 | Guardar factura exitosamente | Todas las validaciones son correctas y el usuario presiona "FINALIZAR" | Usuario hace clic en el botón "FINALIZAR" | El sistema:<br>1. Almacena la información de la factura en la base de datos<br>2. Marca la factura con campo Origin = "EXT" (indica que es factura externa)<br>3. Registra el usuario que cargó el documento<br>4. Almacena metadata:<br>   - Fecha y hora de carga<br>   - Usuario que cargó<br>   - Validaciones ejecutadas<br>5. Envía la factura al buzón del factor en producción<br>   - Propósito: Para que posteriormente se puedan emitir los eventos de RADIAN<br>6. Muestra mensaje de éxito al usuario<br>7. Redirige al usuario al módulo "Inicio" de la plataforma | ☐ | ☐ | ☐ |
| 15 | Notificación al equipo de riesgos por error CUFE | La validación de CUFE en DIAN falla | El sistema detecta que el CUFE no existe o no está aprobado | El sistema envía notificación al equipo de riesgos con:<br><br>**Contenido del correo/notificación**:<br>- Título: "Alerta de Factura Externa no encontra en DIAN"<br>- Información:<br>  * Nombre del archivo<br>  * CUFE consultado<br>  * Usuario que intentó cargar<br>  * Fecha y hora del intento<br>  * Detalle: "Esta entidad debe proveer un cunil interno el servidor provee un cunil externo, sin dejar objetivos ni códices de cunis mensaje de cunis clave orgzados que el remedicho del servicio de evlazador de la misión."<br>- Archivos adjuntos:<br>  * XML subido por el cliente<br>  * Respuesta de la API de la DIAN<br>- Equipo destinatario: program@axcescorp@Axces.com o correo configurado | ☐ | ☐ | ☐ |
| 16 | Notificación al equipo de riesgos por campos inconsistentes | La validación de campos obligatorios entre AD y DIAN falla | El sistema detecta diferencias en los campos | El sistema envía notificación al equipo de riesgos con:<br><br>**Contenido del correo/notificación**:<br>- Título: "Alerta Posible Intento de Fraude - Factura Modificada"<br>- Información:<br>  * Nombre del archivo<br>  * CUFE de la factura<br>  * Usuario que intentó cargar<br>  * Fecha y hora del intento<br>  * NIT del emisor<br>  * NIT del receptor<br>- Tabla comparativa con:<br>  * Campo<br>  * Valor en DIAN<br>  * Valor en XML subido<br>  * Coincide (✓/✗)<br>- Lista de campos con discrepancias:<br>  * No Coincidencia 1: [descripción]<br>  * No Coincidencia 2: [descripción]<br>  * etc.<br>- Archivos adjuntos:<br>  * XML subido por el cliente<br>  * XML con información de la DIAN<br>- Acción requerida: Revisar manualmente | ☐ | ☐ | ☐ |
| 17 | Notificación visual al usuario por error de validación | Una o más validaciones fallan | El sistema completa el proceso de validación | El sistema muestra al usuario:<br>- Tabla con el resultado de la validación<br>- Archivo: [nombre del archivo]<br>- Estado: Rechazada ❌<br>- Descripción: [mensaje de error específico]<br>- El botón "FINALIZAR" permanece deshabilitado<br>- Opciones del usuario:<br>  * Intentar con otro archivo<br>  * Corregir el archivo y volver a cargarlo<br>  * Cancelar operación | ☐ | ☐ | ☐ |
| 18 | Cargar múltiples facturas simultáneamente | Usuario está en la interfaz de carga | Usuario carga varios archivos XML a la vez | El sistema:<br>- Acepta múltiples archivos XML<br>- Procesa cada archivo individualmente<br>- Ejecuta validaciones para cada uno<br>- Muestra resultados en tabla con múltiples filas<br>- Cada fila muestra: Nombre archivo, Estado, Descripción<br>- Permite que algunos archivos sean rechazados y otros aprobados<br>- Solo los archivos con validaciones exitosas se pueden finalizar<br>- El usuario puede decidir finalizar solo los archivos válidos | ☐ | ☐ | ☐ |
| 19 | Restricción de acceso por perfil | Usuario sin perfil Client o Client Administrator intenta acceder | Usuario intenta acceder al módulo "Cargar Facturas" | El sistema:<br>- Verifica el perfil del usuario<br>- Si NO tiene perfil Client o Client Administrator:<br>  * No muestra la opción "Cargar Facturas" en el menú<br>  * O muestra la opción deshabilitada<br>  * Si intenta acceder por URL directa:<br>    - Redirige al inicio<br>    - Muestra mensaje: "No tiene permisos para acceder a esta funcionalidad"<br>- Solo usuarios con perfiles autorizados pueden ver y usar el módulo | ☐ | ☐ | ☐ |
| 20 | Manejo de errores técnicos durante validación DIAN | El sistema intenta consumir API de DIAN | Ocurre error de conexión, timeout o error del servicio de DIAN | El sistema:<br>- Detecta el error técnico (no error de validación)<br>- Implementa reintentos (ej: 3 intentos con backoff exponencial)<br>- Si después de reintentos sigue fallando:<br>  * Registra el error en logs<br>  * Estado: En validación ⏳ o Error técnico ⚠️<br>  * Descripción: "Error al validar con DIAN. Por favor, intente nuevamente más tarde"<br>  * No se procesa la factura<br>  * Se notifica al equipo técnico del error<br>- El usuario puede intentar cargar nuevamente más tarde | ☐ | ☐ | ☐ |
| 21 | Auditoría completa de cargas | Cualquier intento de carga (exitoso o fallido) | El usuario carga un archivo | El sistema registra en auditoría:<br>- ID de la operación<br>- Usuario que realizó la carga<br>- Fecha y hora<br>- Nombre del archivo<br>- Tamaño del archivo<br>- CUFE extraído (si aplica)<br>- NIT emisor extraído<br>- NIT receptor extraído<br>- Resultado de cada validación<br>- Resultado final (Exitoso/Rechazado)<br>- Motivo de rechazo (si aplica)<br>- Tiempo de procesamiento<br>- IP del usuario<br>- Si fue almacenado en BD<br>- Si fue enviado al buzón del factor | ☐ | ☐ | ☐ |

---

## Interacción con el usuario y prototipo

### Flujo completo de carga de factura:

#### 1. **Pantalla de Carga de Facturas**
```
┌──────────────────────────────────────────────┐
│  AXCES                          [Usuario] ▾ │
├──────────────────────────────────────────────┤
│  Menú:                                       │
│  ├─ Inicio                                   │
│  ├─ Cargar Facturas ◄── SELECCIONADO        │
│  ├─ Perfil Empresa                           │
│  └─ Administrar Usuarios                     │
└──────────────────────────────────────────────┘

┌──────────────────────────────────────────────┐
│  Subir Facturas                              │
├──────────────────────────────────────────────┤
│                                              │
│  Ten en cuenta lo siguiente antes de cargar │
│  las facturas:                               │
│                                              │
│  • Es obligatorio subir un XML              │
│    (AttachedDocument) aprobado por la DIAN  │
│                                              │
│  • Solo se permiten archivos en formato XML │
│                                              │
│  • No se reciben facturas vencidas          │
│                                              │
│  • Las facturas deben tener fechas de       │
│    vencimiento                               │
│                                              │
│  Recuerda que el AttachedDocument es el     │
│  documento que genera la DIAN después de    │
│  validar la factura, si tienes un DIAN      │
│  dispones de válido la factura, si tienes   │
│  deuda con el respecto podrías comunicarte  │
│  con tu Proveedor Tecnológico de            │
│  Facturación Electrónica                     │
│                                              │
│  ┌────────────────────────────────────────┐ │
│  │                                        │ │
│  │      📤                                │ │
│  │  Arrastra o suelta tus archivos XML    │ │
│  │  aquí                                  │ │
│  │                                        │ │
│  │  Seleccionar un archivo                │ │
│  │                                        │ │
│  └────────────────────────────────────────┘ │
│                                              │
│                    [FINALIZAR]               │
│                   (deshabilitado)            │
│                                              │
└──────────────────────────────────────────────┘
```

#### 2. **Archivo cargado - Validando**
```
┌──────────────────────────────────────────────┐
│  Subir Facturas                              │
├──────────────────────────────────────────────┤
│                                              │
│  ┌────────────────────────────────────────┐ │
│  │  Archivo cargado - Validando...        │ │
│  │                                        │ │
│  │  📄 ad0902035668963212SETTD01121.xml   │ │
│  │                                        │ │
│  │  ⏳ Validando reglas...                │ │
│  │                                        │ │
│  │  ✓ Formato XML                         │ │
│  │  ✓ No duplicado                        │ │
│  │  ✓ Codificación UTF-8                  │ │
│  │  ✓ AttachedDocument válido             │ │
│  │  ✓ Plazo de vencimiento                │ │
│  │  ✓ Pagador existe                      │ │
│  │  ✓ NIT emisor válido                   │ │
│  │  🔄 Validando CUFE en DIAN...          │ │
│  │  ⏸️ Validando campos vs DIAN...        │ │
│  │                                        │ │
│  └────────────────────────────────────────┘ │
│                                              │
│  NOMBRE ARCHIVO          ESTADO  DESCRIPCIÓN│
│  ─────────────────────────────────────────── │
│  ad090203566...xml       ⏳      Validando  │
│                                              │
│                    [FINALIZAR]               │
│                   (deshabilitado)            │
│                                              │
└──────────────────────────────────────────────┘
```

#### 3. **Validación Exitosa**
```
┌──────────────────────────────────────────────┐
│  Subir Facturas                              │
├──────────────────────────────────────────────┤
│                                              │
│  NOMBRE ARCHIVO          ESTADO  DESCRIPCIÓN│
│  ─────────────────────────────────────────── │
│  ad090203566...xml       ✅      Validado   │
│                                  correctamente│
│                                              │
│                                              │
│                    [FINALIZAR]               │
│                    (habilitado)              │
│                                              │
└──────────────────────────────────────────────┘
```

#### 4. **Error de Validación**
```
┌──────────────────────────────────────────────┐
│  Subir Facturas                              │
├──────────────────────────────────────────────┤
│                                              │
│  NOMBRE ARCHIVO          ESTADO  DESCRIPCIÓN│
│  ─────────────────────────────────────────── │
│  ad090203566...xml       ❌      CUFE no    │
│                         Rechazada encontrado│
│                                  en la DIAN │
│                                  o información│
│                                  inconsistente│
│                                              │
│  archivo_002.xml         ❌      El NIT del │
│                         Rechazada emisor no │
│                                  corresponde │
│                                  con su sesión│
│                                              │
│  SETP915511.xml          ✅      Validado   │
│                                  correctamente│
│                                              │
│                    [FINALIZAR]               │
│                    (habilitado)              │
│          (solo procesa archivos válidos)     │
│                                              │
└──────────────────────────────────────────────┘
```

#### 5. **Confirmación de Guardado (después de presionar FINALIZAR)**
```
┌──────────────────────────────────────────────┐
│  Subir Facturas                              │
├──────────────────────────────────────────────┤
│                                              │
│  ✅ ¡Facturas cargadas exitosamente!        │
│                                              │
│  Las siguientes facturas se han almacenado  │
│  correctamente:                              │
│                                              │
│  • SETP915511.xml                           │
│    - CUFE: abc123...                         │
│    - Monto: $1,500,000.00 COP               │
│    - Vencimiento: 2026-02-15                 │
│                                              │
│  Las facturas han sido enviadas al buzón    │
│  del factor.                                 │
│                                              │
│  Redirigiendo a Inicio...                    │
│                                              │
└──────────────────────────────────────────────┘
```

#### 6. **Notificación al Equipo de Riesgos (Email)**
```
De: sistema@axces.com.co
Para: riesgos@axcescorp.com
Asunto: ⚠️ Alerta Posible Intento de Fraude - Factura Modificada

─────────────────────────────────────────────
ALERTA: Intento de carga de factura con
información inconsistente con DIAN
─────────────────────────────────────────────

Usuario: usuario@empresa.com (NIT: 890930534)
Fecha: 2025-12-09 14:30:00
Archivo: ad090203566896321SETTD011211.xml
CUFE: abc123def456...

CAMPOS CON DISCREPANCIAS:
─────────────────────────────────────────────
No Coincidencia 1:
- Campo: Valor Total de la Factura
- Valor en DIAN: $1,500,000.00
- Valor en XML subido: $1,800,000.00
- Coincide: ✗

No Coincidencia 2:
- Campo: Fecha de Vencimiento
- Valor en DIAN: 2026-01-08
- Valor en XML subido: 2026-02-15
- Coincide: ✗

No Coincidencia 3:
- Campo: Nit del Emisor
- Valor en DIAN: 890930534
- Valor en XML subido: 890930999
- Coincide: ✗

ARCHIVOS ADJUNTOS:
- XML_subido_cliente.xml
- XML_informacion_DIAN.xml

ACCIÓN REQUERIDA:
Por favor revisar manualmente esta operación
y contactar al cliente si es necesario.
```

### Diagrama de flujo técnico:

```
┌─────────────────┐
│   Cliente       │
│ (Perfil Client/ │
│ Client Admin)   │
└────────┬────────┘
         │
         │ 1. Accede a "Cargar Facturas"
         ↓
┌─────────────────────────┐
│ Interfaz de Carga       │
│ - Instrucciones         │
│ - Área drag & drop      │
│ - Botón FINALIZAR (off) │
└────────┬────────────────┘
         │
         │ 2. Arrastra/Selecciona XML
         ↓
┌─────────────────────────┐
│ Validaciones Automáticas│
└────────┬────────────────┘
         │
         ├─► 1. ¿Es XML válido?
         │        │
         │        ├─[NO]─► ❌ "Este archivo no es un xml"
         │        │
         │        └─[SÍ]─► Continúa
         │
         ├─► 2. ¿Factura duplicada?
         │        │
         │        ├─[SÍ]─► ❌ "Ya existe en BD"
         │        │
         │        └─[NO]─► Continúa
         │
         ├─► 3. ¿Codificación UTF-8?
         │        │
         │        ├─[NO]─► ❌ "No es UTF-8"
         │        │
         │        └─[SÍ]─► Continúa
         │
         ├─► 4. ¿Es AttachedDocument?
         │        │
         │        ├─[NO]─► ❌ "No es AD válido"
         │        │
         │        └─[SÍ]─► Continúa
         │
         ├─► 5. ¿Vencimiento > 10 días?
         │        │
         │        ├─[NO]─► ❌ "No cumple plazo"
         │        │
         │        └─[SÍ]─► Continúa
         │
         ├─► 6. ¿Pagador existe en BD?
         │        │
         │        ├─[NO]─► ❌ "Pagador no existe"
         │        │
         │        └─[SÍ]─► Continúa
         │
         ├─► 7. ¿NIT emisor = NIT usuario?
         │        │
         │        ├─[NO]─► ❌ "NIT no corresponde"
         │        │
         │        └─[SÍ]─► Validar DIAN
         │
         ├─► 8. Validar CUFE en DIAN
         │        │
         │        ├──[ERROR]──┐
         │        │           │
         │        │           ↓
         │        │    ┌──────────────────┐
         │        │    │ ❌ Rechazada     │
         │        │    │ Notifica Riesgos │
         │        │    │ Notifica Usuario │
         │        │    └──────────────────┘
         │        │
         │        └─[OK]─► Validar campos
         │
         └─► 9. Validar campos vs DIAN
                  │     (9 campos)
                  │
                  ├──[ERROR]──┐
                  │           │
                  │           ↓
                  │    ┌──────────────────┐
                  │    │ ❌ Rechazada     │
                  │    │ Notifica Riesgos │
                  │    │ Notifica Usuario │
                  │    └──────────────────┘
                  │
                  └─[OK]─► ✅ Habilita FINALIZAR
                           │
                           │ Usuario presiona FINALIZAR
                           ↓
                    ┌──────────────────┐
                    │ Guardar en BD    │
                    │ Origin = "EXT"   │
                    └──────┬───────────┘
                           │
                           ↓
                    ┌──────────────────┐
                    │ Enviar al buzón  │
                    │ del Factor       │
                    │ (eventos RADIAN) │
                    └──────┬───────────┘
                           │
                           ↓
                    ┌──────────────────┐
                    │ ✅ Éxito         │
                    │ Redirigir Inicio │
                    └──────────────────┘
```

### Consideraciones de UX:

- **Validación automática**: El proceso inicia apenas se carga el archivo, sin necesidad de botones adicionales
- **Feedback visual continuo**: Mostrar progreso de cada validación con iconos (⏳, ✓, ✗)
- **Tabla de resultados**: Mostrar claramente estado y descripción de cada archivo
- **Múltiples archivos**: Permitir carga masiva y mostrar resultado individual
- **Botón FINALIZAR condicional**: Solo habilitarlo cuando hay archivos válidos
- **Mensajes descriptivos**: Errores específicos que ayuden al usuario a entender qué corregir
- **Instrucciones claras**: Explicar requisitos antes de la carga
- **Confirmación de éxito**: Mostrar resumen antes de redirigir
- **Notificaciones críticas**: Alertar inmediatamente al equipo de riesgos en casos sospechosos

---

## Definición de Terminado (DoD)

- [ ] El código cumple con los estándares de desarrollo del proyecto
- [ ] Se han implementado todos los escenarios de aceptación (21 escenarios)
- [ ] La interfaz muestra correctamente las instrucciones de carga
- [ ] Las 7 validaciones de reglas de negocio funcionan correctamente
- [ ] La integración con API de validación de CUFE de DIAN funciona
- [ ] La validación de 9 campos obligatorios vs DIAN funciona
- [ ] El sistema de notificaciones al equipo de riesgos está implementado
- [ ] Las notificaciones incluyen los XMLs adjuntos (cliente y DIAN)
- [ ] El guardado en BD con Origin = "EXT" funciona correctamente
- [ ] El envío al buzón del factor está implementado
- [ ] La restricción de perfiles (Client/Client Administrator) funciona
- [ ] La carga múltiple de archivos funciona correctamente
- [ ] La auditoría completa de operaciones está implementada
- [ ] El manejo de errores técnicos (timeout DIAN) es robusto
- [ ] Las pruebas unitarias tienen una cobertura mínima del 80%
- [ ] Las pruebas de integración con DIAN funcionan
- [ ] La funcionalidad ha sido probada con archivos XML reales
- [ ] La documentación técnica está actualizada
- [ ] El Product Owner ha validado la funcionalidad
- [ ] No existen bugs críticos pendientes

---

## Notas Técnicas

### Reglas de Validación (Resumen):

| # | Validación | Campo/Condición | Error si falla |
|---|------------|-----------------|----------------|
| 1 | Formato XML | Archivo debe ser XML válido | "Este archivo no es un xml" |
| 2 | No duplicado | CUFE no existe en BD Axces | "La factura ya existe en BD" |
| 3 | Codificación | UTF-8 | "No está codificado en UTF-8" |
| 4 | Tipo documento | Invoice + Application Response | "No es un AD válido" |
| 5 | Plazo | Payment Due Date > Actual Date + 10, ID = 2 | "No cumple plazo mínimo" |
| 6 | Pagador | NIT receptor existe en BD Axces | "Pagador no existe en BD" |
| 7 | Emisor | NIT emisor = NIT usuario sesión | "NIT emisor no corresponde" |
| 8 | CUFE DIAN | CUFE existe y aprobado en DIAN | "CUFE no encontrado en DIAN" |
| 9 | Campos DIAN | 9 campos coinciden con DIAN | "Información inconsistente" |

### Campos a validar contra DIAN:

1. Forma de Pago
2. Fecha de Emisión
3. Fecha de Vencimiento
4. Nit del Receptor
5. Valor Total de la Factura
6. Valor Total Anticipos
7. Número de Factura
8. Nit del Emisor
9. Moneda de la Factura

### API de DIAN (Validación de CUFE):

**Documentación**: Según el documento, existe un servicio de validación de DIAN. El equipo debe consultar la documentación oficial de la API de la DIAN para:
- Endpoint de validación de CUFE
- Autenticación requerida
- Formato de request/response
- Endpoint de consulta de información detallada de factura

**Ejemplo conceptual:**
```
POST /api/dian/validar-cufe
{
  "cufe": "abc123def456..."
}

Response:
{
  "valido": true,
  "estado": "Aprobado",
  "fechaAprobacion": "2025-12-09",
  "datosFactura": {
    "formaPago": "2",
    "fechaEmision": "2025-12-09",
    "fechaVencimiento": "2026-01-08",
    ...
  }
}
```

### Endpoint API Sugerido (Backend Axces):

#### Validar y cargar factura
```
POST /api/facturas/externas/validar
Content-Type: multipart/form-data

Headers:
- Authorization: Bearer {token}
- X-User-NIT: {nit_usuario_sesion}
```

**Request**:
```
files: [archivo1.xml, archivo2.xml, ...]
```

**Response (Procesando)**:
```json
{
  "archivos": [
    {
      "nombreArchivo": "ad090203566896321.xml",
      "estado": "validando",
      "validaciones": {
        "formatoXML": "ok",
        "noDuplicado": "ok",
        "codificacionUTF8": "ok",
        "attachedDocument": "ok",
        "plazoVencimiento": "ok",
        "pagadorExiste": "ok",
        "nitEmisor": "ok",
        "cuFEDIAN": "procesando",
        "camposDIAN": "pendiente"
      }
    }
  ]
}
```

**Response (Completado con errores)**:
```json
{
  "archivos": [
    {
      "nombreArchivo": "ad090203566896321.xml",
      "estado": "rechazada",
      "descripcion": "CUFE no encontrado en la DIAN o información inconsistente",
      "validaciones": {
        "formatoXML": "ok",
        "noDuplicado": "ok",
        "codificacionUTF8": "ok",
        "attachedDocument": "ok",
        "plazoVencimiento": "ok",
        "pagadorExiste": "ok",
        "nitEmisor": "ok",
        "cuFEDIAN": "error",
        "camposDIAN": "no_ejecutado"
      },
      "detalleError": {
        "paso": "validacion_cufe_dian",
        "mensaje": "El CUFE no existe en la base de datos de la DIAN"
      },
      "notificacionEnviada": true
    }
  ]
}
```

**Response (Completado exitoso)**:
```json
{
  "archivos": [
    {
      "nombreArchivo": "SETP915511.xml",
      "estado": "validado",
      "descripcion": "Validado correctamente",
      "validaciones": {
        "formatoXML": "ok",
        "noDuplicado": "ok",
        "codificacionUTF8": "ok",
        "attachedDocument": "ok",
        "plazoVencimiento": "ok",
        "pagadorExiste": "ok",
        "nitEmisor": "ok",
        "cuFEDIAN": "ok",
        "camposDIAN": "ok"
      },
      "datosExtraidos": {
        "cufe": "abc123...",
        "numeroFactura": "SETP915511",
        "nitEmisor": "890930534",
        "nitReceptor": "901453011",
        "valorTotal": 1500000.00,
        "moneda": "COP",
        "fechaEmision": "2025-12-09",
        "fechaVencimiento": "2026-02-15"
      },
      "listoParaGuardar": true
    }
  ]
}
```

#### Finalizar y guardar facturas validadas
```
POST /api/facturas/externas/finalizar
```

**Request**:
```json
{
  "archivos": ["SETP915511.xml"],
  "usuarioCarga": "usuario@empresa.com"
}
```

**Response**:
```json
{
  "success": true,
  "message": "Facturas guardadas exitosamente",
  "facturasGuardadas": [
    {
      "id": "uuid",
      "cufe": "abc123...",
      "numeroFactura": "SETP915511",
      "origin": "EXT",
      "enviadoBuzonFactor": true,
      "fechaCarga": "2025-12-09T14:30:00Z"
    }
  ]
}
```

### Estructura Base de Datos:

```sql
-- Tabla de facturas externas
CREATE TABLE facturas_externas (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  cufe VARCHAR(255) NOT NULL UNIQUE,
  numero_factura VARCHAR(50) NOT NULL,
  prefijo VARCHAR(10),
  nit_emisor VARCHAR(20) NOT NULL,
  nit_receptor VARCHAR(20) NOT NULL,
  valor_total DECIMAL(18,2) NOT NULL,
  valor_anticipos DECIMAL(18,2),
  moneda VARCHAR(3) DEFAULT 'COP',
  forma_pago VARCHAR(10),
  fecha_emision DATE NOT NULL,
  fecha_vencimiento DATE NOT NULL,
  origin VARCHAR(10) DEFAULT 'EXT',
  xml_path TEXT,
  usuario_carga VARCHAR(100),
  fecha_carga TIMESTAMP DEFAULT NOW(),
  enviado_buzon_factor BOOLEAN DEFAULT FALSE,
  fecha_envio_buzon TIMESTAMP,
  estado VARCHAR(50) DEFAULT 'Cargada',

  -- Índices
  INDEX idx_cufe (cufe),
  INDEX idx_nit_emisor (nit_emisor),
  INDEX idx_nit_receptor (nit_receptor),
  INDEX idx_fecha_carga (fecha_carga DESC),
  INDEX idx_origin (origin),

  -- Foreign Keys
  FOREIGN KEY (nit_receptor) REFERENCES pagadores(nit)
);

-- Tabla de auditoría de cargas
CREATE TABLE auditoria_cargas_externas (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  factura_id UUID REFERENCES facturas_externas(id),
  cufe VARCHAR(255),
  nombre_archivo VARCHAR(255) NOT NULL,
  tamano_archivo INT,
  usuario VARCHAR(100) NOT NULL,
  nit_usuario VARCHAR(20),
  ip_usuario VARCHAR(45),
  fecha_intento TIMESTAMP DEFAULT NOW(),
  resultado VARCHAR(20) NOT NULL, -- 'EXITOSO', 'RECHAZADO', 'ERROR_TECNICO'
  motivo_rechazo TEXT,
  validaciones_ejecutadas JSONB,
  tiempo_procesamiento_ms INT,
  notificacion_riesgos_enviada BOOLEAN DEFAULT FALSE,

  INDEX idx_usuario (usuario),
  INDEX idx_resultado (resultado),
  INDEX idx_fecha_intento (fecha_intento DESC)
);

-- Tabla de notificaciones al equipo de riesgos
CREATE TABLE notificaciones_riesgos (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tipo VARCHAR(50) NOT NULL, -- 'CUFE_INVALIDO', 'CAMPOS_INCONSISTENTES'
  factura_cufe VARCHAR(255),
  usuario VARCHAR(100) NOT NULL,
  nit_usuario VARCHAR(20),
  nombre_archivo VARCHAR(255),
  fecha_notificacion TIMESTAMP DEFAULT NOW(),
  detalles JSONB,
  xml_cliente TEXT,
  xml_dian TEXT,
  estado VARCHAR(20) DEFAULT 'PENDIENTE', -- 'PENDIENTE', 'REVISADO', 'RESUELTO'
  revisado_por VARCHAR(100),
  fecha_revision TIMESTAMP,
  notas_revision TEXT,

  INDEX idx_tipo (tipo),
  INDEX idx_estado (estado),
  INDEX idx_fecha_notificacion (fecha_notificacion DESC)
);
```

### Lógica de Procesamiento (Pseudocódigo):

```javascript
async function procesarFacturaExterna(xmlFile, usuarioNIT) {
  const resultado = {
    nombreArchivo: xmlFile.name,
    estado: 'procesando',
    validaciones: {},
    errores: []
  };

  try {
    // Validación 1: Formato XML
    const xml = await parseXML(xmlFile);
    resultado.validaciones.formatoXML = 'ok';

    // Validación 2: No duplicado
    const cufe = extraerCUFE(xml);
    const existe = await buscarPorCUFE(cufe);
    if (existe) {
      throw new ValidationError('La factura ya existe en BD');
    }
    resultado.validaciones.noDuplicado = 'ok';

    // Validación 3: Codificación UTF-8
    if (!esUTF8(xmlFile)) {
      throw new ValidationError('No está codificado en UTF-8');
    }
    resultado.validaciones.codificacionUTF8 = 'ok';

    // Validación 4: AttachedDocument
    if (!esAttachedDocument(xml)) {
      throw new ValidationError('No es un AD válido');
    }
    resultado.validaciones.attachedDocument = 'ok';

    // Validación 5: Plazo de vencimiento
    const formaPago = xml.querySelector('PaymentMeansID').textContent;
    const fechaVencimiento = new Date(xml.querySelector('PaymentDueDate').textContent);
    const hoy = new Date();
    const diasDiferencia = (fechaVencimiento - hoy) / (1000 * 60 * 60 * 24);

    if (formaPago !== '2' || diasDiferencia <= 10) {
      throw new ValidationError('No cumple plazo mínimo');
    }
    resultado.validaciones.plazoVencimiento = 'ok';

    // Validación 6: Pagador existe
    const nitReceptor = xml.querySelector('ReceiverNIT').textContent;
    const pagadorExiste = await buscarPagador(nitReceptor);
    if (!pagadorExiste) {
      throw new ValidationError('Pagador no existe en BD');
    }
    resultado.validaciones.pagadorExiste = 'ok';

    // Validación 7: NIT emisor
    const nitEmisor = xml.querySelector('SupplierNIT').textContent;
    if (nitEmisor !== usuarioNIT) {
      throw new ValidationError('NIT emisor no corresponde');
    }
    resultado.validaciones.nitEmisor = 'ok';

    // Validación 8: CUFE en DIAN
    const respuestaDIAN = await validarCUFEEnDIAN(cufe);
    if (!respuestaDIAN.valido) {
      await notificarEquipoRiesgos({
        tipo: 'CUFE_INVALIDO',
        cufe: cufe,
        usuario: usuarioNIT,
        nombreArchivo: xmlFile.name,
        xmlCliente: xml.toString()
      });
      throw new ValidationError('CUFE no encontrado en DIAN');
    }
    resultado.validaciones.cuFEDIAN = 'ok';

    // Validación 9: Campos vs DIAN
    const camposComparar = [
      'formaPago',
      'fechaEmision',
      'fechaVencimiento',
      'nitReceptor',
      'valorTotal',
      'valorAnticipos',
      'numeroFactura',
      'nitEmisor',
      'moneda'
    ];

    const inconsistencias = [];
    for (const campo of camposComparar) {
      const valorCliente = extraerCampo(xml, campo);
      const valorDIAN = respuestaDIAN.datosFactura[campo];

      if (valorCliente !== valorDIAN) {
        inconsistencias.push({
          campo: campo,
          valorCliente: valorCliente,
          valorDIAN: valorDIAN
        });
      }
    }

    if (inconsistencias.length > 0) {
      await notificarEquipoRiesgos({
        tipo: 'CAMPOS_INCONSISTENTES',
        cufe: cufe,
        usuario: usuarioNIT,
        nombreArchivo: xmlFile.name,
        inconsistencias: inconsistencias,
        xmlCliente: xml.toString(),
        xmlDIAN: respuestaDIAN.xmlOriginal
      });
      throw new ValidationError('Información inconsistente con DIAN');
    }
    resultado.validaciones.camposDIAN = 'ok';

    // Todo validado exitosamente
    resultado.estado = 'validado';
    resultado.descripcion = 'Validado correctamente';
    resultado.listoParaGuardar = true;
    resultado.datosExtraidos = extraerTodosCampos(xml);

  } catch (error) {
    resultado.estado = 'rechazada';
    resultado.descripcion = error.message;
    resultado.errores.push(error);
  }

  // Auditoría
  await registrarAuditoria({
    nombreArchivo: xmlFile.name,
    usuario: usuarioNIT,
    resultado: resultado.estado,
    motivoRechazo: resultado.descripcion,
    validaciones: resultado.validaciones
  });

  return resultado;
}

async function finalizarYGuardar(archivosValidados, usuario) {
  const facturasGuardadas = [];

  for (const archivo of archivosValidados) {
    // Guardar en BD
    const facturaId = await guardarFacturaExterna({
      ...archivo.datosExtraidos,
      origin: 'EXT',
      usuarioCarga: usuario,
      fechaCarga: new Date()
    });

    // Enviar al buzón del factor
    await enviarABuzonFactor(facturaId, archivo.datosExtraidos);

    facturasGuardadas.push({
      id: facturaId,
      cufe: archivo.datosExtraidos.cufe,
      numeroFactura: archivo.datosExtraidos.numeroFactura
    });
  }

  return facturasGuardadas;
}
```

### Sistema de Notificaciones al Equipo de Riesgos:

```javascript
async function notificarEquipoRiesgos(datos) {
  const { tipo, cufe, usuario, nombreArchivo, xmlCliente, xmlDIAN, inconsistencias } = datos;

  let asunto, cuerpo;

  if (tipo === 'CUFE_INVALIDO') {
    asunto = '⚠️ Alerta de Factura Externa no encontrada en DIAN';
    cuerpo = `
      ALERTA: Intento de carga de factura con CUFE inválido

      Usuario: ${usuario}
      Archivo: ${nombreArchivo}
      CUFE: ${cufe}
      Fecha: ${new Date().toISOString()}

      El CUFE consultado no existe o no está aprobado en la base de datos de la DIAN.

      Por favor revisar el archivo adjunto y contactar al cliente si es necesario.
    `;
  } else if (tipo === 'CAMPOS_INCONSISTENTES') {
    const listaInconsistencias = inconsistencias.map((inc, index) =>
      `No Coincidencia ${index + 1}:
       - Campo: ${inc.campo}
       - Valor en DIAN: ${inc.valorDIAN}
       - Valor en XML subido: ${inc.valorCliente}
       - Coincide: ✗`
    ).join('\n\n');

    asunto = '⚠️ Alerta Posible Intento de Fraude - Factura Modificada';
    cuerpo = `
      ALERTA: Intento de carga de factura con información inconsistente con DIAN

      Usuario: ${usuario}
      Archivo: ${nombreArchivo}
      CUFE: ${cufe}
      Fecha: ${new Date().toISOString()}

      CAMPOS CON DISCREPANCIAS:
      ${listaInconsistencias}

      ARCHIVOS ADJUNTOS:
      - XML subido por el cliente
      - XML con información de la DIAN

      ACCIÓN REQUERIDA:
      Por favor revisar manualmente esta operación y contactar al cliente si es necesario.
    `;
  }

  // Enviar email
  await enviarEmail({
    para: 'riesgos@axcescorp.com',
    asunto: asunto,
    cuerpo: cuerpo,
    adjuntos: [
      { nombre: 'XML_cliente.xml', contenido: xmlCliente },
      { nombre: 'XML_DIAN.xml', contenido: xmlDIAN || '' }
    ]
  });

  // Registrar notificación en BD
  await guardarNotificacionRiesgos({
    tipo: tipo,
    facturaCufe: cufe,
    usuario: usuario,
    nombreArchivo: nombreArchivo,
    detalles: { inconsistencias },
    xmlCliente: xmlCliente,
    xmlDian: xmlDIAN
  });
}
```

---

## Dependencias

- **Servicios Externos**:
  - API de validación de CUFE de la DIAN
  - API de consulta de información de facturas de la DIAN

- **Infraestructura**:
  - Base de datos relacional (PostgreSQL/MySQL) para facturas y auditoría
  - Sistema de almacenamiento de archivos XML
  - Servidor de correo electrónico para notificaciones
  - Buzón del Factor (sistema de mensajería/eventos)

- **Seguridad**:
  - Sistema de autenticación y autorización por roles (perfiles)
  - Parser XML seguro (protección contra XXE)

- **Componentes del sistema**:
  - Componente de carga de archivos (drag & drop)
  - Sistema de validación en tiempo real
  - Sistema de notificaciones
  - Integración con RADIAN (eventos)

---

## Riesgos y Mitigaciones

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|--------------|---------|------------|
| API de DIAN no disponible o lenta | Alta | Alto | Implementar reintentos con backoff; timeout configurables; cache de validaciones; alertar al equipo técnico |
| Usuario intenta cargar facturas modificadas (fraude) | Media | Crítico | Validación estricta de 9 campos vs DIAN; notificación inmediata a riesgos; auditoría completa |
| XML malformado causa error en parser | Media | Medio | Validación de formato antes de parsear; usar parser seguro; manejo de excepciones robusto |
| Facturas duplicadas por timing | Baja | Medio | Constraint UNIQUE en CUFE en BD; validación atómica antes de insert |
| Carga de archivos muy grandes causa timeout | Media | Medio | Límite de tamaño de archivo; procesamiento asíncrono si es necesario |
| Notificaciones al equipo de riesgos no llegan | Media | Alto | Múltiples canales (email, BD, dashboard); logs detallados; reintentos en envío |
| Usuario sin permisos accede por URL directa | Baja | Alto | Validación de permisos en backend además de frontend; middleware de autorización |
| Almacenamiento de XMLs consume mucho espacio | Alta | Medio | Compresión de archivos; limpieza periódica de archivos antiguos; almacenamiento en la nube |
| Ataques XXE (XML External Entity) | Baja | Crítico | Deshabilitar entidades externas en parser XML; validar estructura antes de procesar |
| Inconsistencias entre validaciones frontend y backend | Media | Alto | Validaciones críticas solo en backend; frontend solo para UX |

---

## Casos de Prueba Sugeridos

### Pruebas Funcionales:

1. **Carga exitosa básica**: Cargar un XML válido con todas las validaciones correctas
2. **Archivo no XML**: Intentar cargar un archivo .pdf o .txt
3. **XML mal formado**: Cargar XML con sintaxis incorrecta
4. **Factura duplicada**: Cargar dos veces el mismo CUFE
5. **Codificación incorrecta**: XML en ISO-8859-1 en lugar de UTF-8
6. **No es AttachedDocument**: Cargar solo Invoice sin Application Response
7. **Plazo menor a 10 días**: Factura con vencimiento en 5 días
8. **Factura de contado**: Forma de pago diferente a crédito (ID ≠ 2)
9. **Pagador no existe**: NIT receptor no está en BD de Axces
10. **NIT emisor no corresponde**: Usuario con NIT diferente al emisor
11. **CUFE inválido en DIAN**: CUFE que no existe en DIAN
12. **Campos inconsistentes**: Monto diferente entre XML y DIAN
13. **Carga múltiple mixta**: 3 archivos (1 válido, 2 con errores)
14. **Usuario sin perfil correcto**: Usuario sin rol Client/Client Administrator
15. **Finalizar solo archivos válidos**: De 5 archivos, finalizar solo los 2 válidos

### Pruebas de Integración:

1. Verificar que las notificaciones al equipo de riesgos se envían correctamente
2. Verificar que los XMLs se adjuntan en las notificaciones
3. Verificar que las facturas se guardan con Origin = "EXT"
4. Verificar que las facturas se envían al buzón del factor
5. Verificar que la auditoría registra todos los intentos
6. Verificar comportamiento cuando API DIAN está caída
7. Verificar reintentos automáticos en errores de red

### Pruebas de Seguridad:

1. Intento de XXE attack mediante XML malicioso
2. Intento de cargar archivo XML extremadamente grande (>100MB)
3. Intento de acceso sin autenticación
4. Intento de acceso con rol incorrecto
5. Intento de cargar factura de otro emisor (fraude)
6. Validar que las contraseñas/tokens no se loguean en auditoría

### Pruebas de Rendimiento:

1. Carga de 50 archivos simultáneos
2. Carga de archivo XML de 10MB
3. Tiempo de respuesta de validaciones (< 10 segundos)
4. Concurrencia: 10 usuarios cargando al mismo tiempo

---

**Fecha de creación**: 2026-02-02
**Última actualización**: 2026-02-02
**Versión del documento de referencia**: DO-Funcional Fact. Externas-020226-145745.pdf (v.12, Dec 18, 2023)
