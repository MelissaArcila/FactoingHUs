# Generación Automática de Evento RADIAN - Informe para Pago

**By:** [Nombre del Product Owner]

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
| **Arquitectura relacionada** | RADIAN, Sistema de Eventos, Notificaciones |
| **Documento de referencia** | Especificación Eventos RADIAN |

---

## Contexto

### Enunciado general de la historia

Como **Sistema Axces**, quiero **generar automáticamente el evento RADIAN de Informe para Pago** 3 días hábiles antes del vencimiento de cada factura en estado "Desembolsado", para que el Factor pueda cumplir con los requerimientos de la DIAN de notificar oportunamente el pago programado de las facturas, garantizando la trazabilidad completa del proceso y notificando al equipo de Backoffice cuando ocurran errores que no puedan resolverse automáticamente.

### Roles
- **Sistema Axces**: Sistema que ejecuta el proceso automático diario
- **Factor**: Empresa de factoring que desembolsa las facturas
- **DIAN**: Entidad que recibe los eventos RADIAN
- **Backoffice**: Equipo que recibe notificaciones de errores críticos
- **Usuarios visualizadores**: Personas que consultan el estado de las facturas en la pestaña "Desembolsada"

### Característica / Funcionalidad
Generación automática y programada de eventos RADIAN tipo "Informe para Pago" (código 046) con sistema de reintentos, notificaciones de error y trazabilidad completa visible desde la interfaz de usuario.

### Razón / Resultado
Cumplir con las obligaciones regulatorias de la DIAN mediante la notificación automática y oportuna del informe para pago de facturas desembolsadas, reduciendo la intervención manual y garantizando que se generen los eventos dentro del plazo establecido (3 días hábiles antes del vencimiento).

---

## Escenarios

| Número | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|--------|--------------------------------|----------|--------|-------------------------------------|------------|----|--------------|
| 1 | Proceso diario automático se ejecuta | El sistema tiene configurado un proceso automático | Todos los días a una hora definida (ej: 6:00 AM) | El sistema:<br>1. Inicia automáticamente el proceso de revisión de facturas<br>2. Busca todas las facturas que cumplan las condiciones para generar el evento<br>3. Registra en log el inicio del proceso<br>4. Procede a evaluar cada factura<br>5. El proceso se ejecuta sin intervención manual<br>6. Se ejecuta incluso en fines de semana (evalúa solo días hábiles) | ☐ | ☐ | ☐ |
| 2 | Identificar facturas elegibles | El proceso diario se está ejecutando | El sistema busca facturas que necesitan el evento | El sistema identifica facturas que cumplan TODAS estas condiciones:<br><br>**Condiciones obligatorias**:<br>1. Estado = "Desembolsado"<br>2. Fecha de vencimiento - 3 días hábiles = Hoy<br>3. El evento de Informe para Pago NO ha sido generado exitosamente antes<br>4. La factura NO tiene errores bloqueantes de generación<br><br>**Ejemplo**:<br>- Si hoy es lunes 5 de febrero<br>- Y la factura vence el jueves 8 de febrero<br>- Contando 3 días hábiles antes: lunes 5<br>- ✅ La factura es elegible HOY<br><br>**Días hábiles**:<br>- No se cuentan sábados, domingos ni festivos<br>- Se debe considerar el calendario de festivos colombiano | ☐ | ☐ | ☐ |
| 3 | Preparar datos del evento | Se identificó una factura elegible | El sistema prepara la información para enviar | El sistema extrae de la factura la siguiente información:<br><br>**Datos requeridos**:<br>- supplierId: NIT del emisor de la factura<br>- receiverId: NIT del receptor de la factura<br>- documentTypeCode: Valor fijo "01" (Factura)<br>- documentId: Número de la factura (ej: "SETT4890161")<br>- factorId: Valor fijo configurado en el sistema<br>- username: Campo vacío ""<br>- statusCode: Valor fijo "046" (Informe para Pago)<br>- statusDate: Fecha y hora actual en formato timestamp<br>- statusReason: Campo vacío ""<br>- statusNote: Campo vacío ""<br>- CustomerEmail: Email del cliente/emisor de la factura<br><br>El sistema valida que todos los datos requeridos estén disponibles antes de enviar | ☐ | ☐ | ☐ |
| 4 | Enviar evento al servicio RADIAN | Los datos están preparados | El sistema envía el evento al servicio de generación de eventos | El sistema:<br>1. Construye la petición con el formato JSON requerido<br>2. Envía la petición al servicio de generación de eventos RADIAN<br>3. Espera la respuesta del servicio<br>4. Si la respuesta es exitosa:<br>   - Marca el evento como generado exitosamente<br>   - Registra fecha y hora de generación<br>   - Registra respuesta del servicio<br>   - Continúa con la siguiente factura<br>5. Si la respuesta es un error:<br>   - Pasa al flujo de reintentos (ver siguiente escenario) | ☐ | ☐ | ☐ |
| 5 | Reintentar en caso de error | El envío del evento falló | El sistema detecta un error en la generación | El sistema implementa reintentos con las siguientes reglas:<br><br>**Estrategia de reintentos**:<br>1. Primer intento: Inmediato<br>2. Si falla: Espera 5 minutos → Segundo intento<br>3. Si falla: Espera 15 minutos → Tercer intento<br>4. Si falla: Espera 30 minutos → Cuarto intento (último del día)<br><br>**Después de 4 intentos fallidos**:<br>- Marca la factura con estado: "Error temporal - Reintentar mañana"<br>- Registra el error y todos los intentos<br>- NO notifica a Backoffice todavía<br>- Al día siguiente, vuelve a intentar desde cero<br><br>**Tipos de error que permiten reintento**:<br>- Timeout de conexión<br>- Error 500 del servicio<br>- Error de red temporal<br>- Servicio no disponible | ☐ | ☐ | ☐ |
| 6 | Notificar error crítico a Backoffice | Una factura falló 2 días consecutivos | El sistema detecta falla persistente | El sistema:<br>1. Identifica que la factura falló ayer Y falló hoy<br>2. Marca la factura con estado: "Error crítico - Requiere intervención"<br>3. Envía notificación al equipo de Backoffice con:<br><br>**Contenido de la notificación**:<br>- Asunto: "⚠️ Error crítico: Evento RADIAN no generado - Factura [ID]"<br>- Información de la factura:<br>  * Número de factura<br>  * NIT emisor<br>  * NIT receptor<br>  * Fecha de vencimiento<br>  * Días restantes hasta vencimiento<br>- Detalles del error:<br>  * Descripción del error<br>  * Fecha y hora de los intentos<br>  * Respuestas recibidas del servicio<br>- Acción requerida: "Por favor revisar y generar manualmente"<br><br>4. NO vuelve a intentar automáticamente hasta que Backoffice resuelva el error<br>5. La factura queda marcada como pendiente de intervención manual | ☐ | ☐ | ☐ |
| 7 | Registrar trazabilidad completa | Cualquier intento de generación (exitoso o fallido) | El sistema procesa un evento | El sistema registra la siguiente información para cada factura:<br><br>**Información de trazabilidad**:<br>- ID de la factura<br>- Fecha y hora de cada intento<br>- Resultado de cada intento (Éxito/Error)<br>- Descripción del error (si aplica)<br>- Código de respuesta del servicio<br>- Mensaje de respuesta del servicio<br>- Datos enviados (JSON completo)<br>- Usuario del sistema que ejecutó (automático)<br>- Número de intentos realizados<br>- Fecha de último intento<br>- Estado final del evento<br><br>Esta información debe ser:<br>- Almacenada permanentemente<br>- Consultable desde la interfaz<br>- Auditable<br>- Exportable | ☐ | ☐ | ☐ |
| 8 | Visualizar en pestaña Desembolsada | Usuario accede a la pestaña "Desembolsada" | Usuario consulta las facturas desembolsadas | El sistema muestra en la pestaña "Desembolsada" una columna adicional llamada "Evento Informe Pago" que muestra:<br><br>**Para cada factura**:<br><br>**Si el evento se generó exitosamente**:<br>- ✅ Icono verde (check)<br>- Texto: "Generado"<br>- Al hacer clic o pasar el mouse:<br>  * Fecha y hora de generación<br>  * "Generado automáticamente"<br><br>**Si está pendiente de generar**:<br>- ⏳ Icono de reloj<br>- Texto: "Pendiente"<br>- Al hacer clic o pasar el mouse:<br>  * "Se generará 3 días hábiles antes del vencimiento"<br>  * Fecha estimada de generación<br><br>**Si tuvo error temporal**:<br>- ⚠️ Icono amarillo de advertencia<br>- Texto: "Reintentando"<br>- Al hacer clic o pasar el mouse:<br>  * "Error temporal - Se reintentará mañana"<br>  * Número de intentos realizados<br>  * Último error<br><br>**Si tuvo error crítico**:<br>- ❌ Icono rojo (X)<br>- Texto: "Error - Requiere intervención"<br>- Al hacer clic o pasar el mouse:<br>  * Descripción del error<br>  * "Backoffice notificado"<br>  * Botón: "Ver detalles" | ☐ | ☐ | ☐ |
| 9 | Ver detalle completo del evento | Usuario hace clic en "Ver detalles" de un evento | Usuario quiere ver la trazabilidad completa | El sistema muestra un modal o panel con:<br><br>**Información general**:<br>- Número de factura<br>- Estado del evento<br>- Fecha de vencimiento de la factura<br>- Días hábiles restantes<br><br>**Historial de intentos**:<br>Tabla con columnas:<br>- Fecha/Hora del intento<br>- Resultado (Éxito/Error)<br>- Código de respuesta<br>- Mensaje<br>- Datos enviados (expandible)<br><br>**Si está en error crítico**:<br>- Estado de notificación a Backoffice<br>- Fecha de notificación<br>- Opción: "Reintentar manualmente" (solo para Backoffice)<br><br>**Acciones disponibles** (según permisos):<br>- Descargar detalles (JSON/PDF)<br>- Copiar datos enviados<br>- Reintentar generación (solo Backoffice) | ☐ | ☐ | ☐ |
| 10 | No procesar facturas ya procesadas | El proceso diario encuentra una factura con evento exitoso | El sistema evalúa una factura | El sistema:<br>1. Verifica si la factura ya tiene el evento "Informe para Pago" generado exitosamente<br>2. Si YA tiene el evento:<br>   - La omite del procesamiento<br>   - No intenta generar nuevamente<br>   - Continúa con la siguiente factura<br>3. Si NO tiene el evento:<br>   - Procede con la generación<br><br>Esto previene duplicados y optimiza el proceso | ☐ | ☐ | ☐ |
| 11 | Calcular correctamente días hábiles | El sistema debe determinar cuándo generar el evento | El sistema calcula 3 días hábiles antes del vencimiento | El sistema:<br>1. Toma la fecha de vencimiento de la factura<br>2. Retrocede 3 días HÁBILES (no naturales)<br>3. Considera:<br>   - Lunes a viernes como días hábiles<br>   - Sábados y domingos NO son hábiles<br>   - Festivos colombianos NO son hábiles<br>4. Genera el evento en el día calculado<br><br>**Ejemplos**:<br><br>Ejemplo 1:<br>- Vencimiento: Viernes 9 Feb<br>- 3 días hábiles antes: Martes 6 Feb<br>- ✅ Se genera el martes 6<br><br>Ejemplo 2:<br>- Vencimiento: Miércoles 7 Feb<br>- 3 días hábiles antes: Viernes 2 Feb<br>- ✅ Se genera el viernes 2<br><br>Ejemplo 3 (con fin de semana):<br>- Vencimiento: Martes 6 Feb<br>- Retrocediendo: Lunes 5, Viernes 2, Jueves 1<br>- ✅ Se genera el jueves 1<br><br>Ejemplo 4 (con festivo):<br>- Vencimiento: Jueves 8 Feb<br>- Lunes 5 es festivo<br>- Retrocediendo: Miércoles 7, Viernes 2, Jueves 1<br>- ✅ Se genera el jueves 1 | ☐ | ☐ | ☐ |
| 12 | Reportar resumen diario | El proceso diario finaliza | Todas las facturas fueron procesadas | El sistema genera un reporte/log diario con:<br><br>**Resumen del proceso**:<br>- Fecha y hora de ejecución<br>- Total de facturas evaluadas<br>- Total de facturas elegibles<br>- Total de eventos generados exitosamente<br>- Total de eventos con error temporal<br>- Total de eventos con error crítico<br>- Total de notificaciones enviadas a Backoffice<br>- Tiempo total de ejecución<br><br>Este reporte debe:<br>- Guardarse en el sistema<br>- Ser consultable por Backoffice<br>- Permitir análisis histórico<br>- Opcionalmente: Enviarse por email a un grupo definido | ☐ | ☐ | ☐ |
| 13 | Reintento manual desde Backoffice | Backoffice recibió notificación de error crítico | Usuario de Backoffice necesita reintentar manualmente | El sistema permite a usuarios con perfil Backoffice:<br>1. Acceder al detalle de la factura con error<br>2. Ver el historial completo de intentos<br>3. Revisar el error específico<br>4. Hacer clic en "Reintentar generación"<br>5. El sistema:<br>   - Valida que el usuario tenga permisos<br>   - Vuelve a intentar generar el evento<br>   - Muestra en tiempo real si fue exitoso o no<br>   - Si es exitoso:<br>     * Cambia estado a "Generado"<br>     * Registra que fue generación manual<br>     * Registra usuario que realizó el reintento<br>   - Si falla:<br>     * Muestra el error al usuario<br>     * Permite editar datos si es necesario<br>     * Permite intentar nuevamente | ☐ | ☐ | ☐ |
| 14 | No generar para facturas sin email | El sistema prepara datos de una factura sin email del cliente | La factura no tiene CustomerEmail | El sistema:<br>1. Detecta que la factura NO tiene email del cliente<br>2. Marca la factura con error: "Falta email del cliente"<br>3. NO intenta enviar el evento<br>4. Notifica a Backoffice inmediatamente con:<br>   - "Factura [ID] no puede generar evento: Falta email del cliente"<br>   - Solicita que se complete la información<br>5. Una vez se agregue el email:<br>   - La factura vuelve a ser elegible<br>   - Se intenta generar en el siguiente proceso diario | ☐ | ☐ | ☐ |
| 15 | Manejar cambios de estado de factura | Una factura cambia de estado después de que se generó el evento | La factura pasa de "Desembolsado" a otro estado | El sistema:<br>1. Mantiene el registro del evento generado<br>2. NO invalida el evento enviado a DIAN<br>3. Muestra en la interfaz:<br>   - Que el evento fue generado<br>   - El estado actual de la factura<br>   - Advertencia si aplica: "El evento fue generado cuando la factura estaba desembolsada"<br>4. NO genera nuevos eventos automáticamente para esta factura<br>5. Si es necesario generar otro evento (por cambio de estado), debe ser proceso manual | ☐ | ☐ | ☐ |
| 16 | Filtrar facturas en pestaña Desembolsada | Usuario quiere ver solo facturas con errores | Usuario aplica filtros en la pestaña | El sistema permite filtrar las facturas por estado del evento:<br><br>**Filtros disponibles**:<br>- Todos<br>- Evento generado ✅<br>- Evento pendiente ⏳<br>- Con errores temporales ⚠️<br>- Con errores críticos ❌<br><br>Al aplicar un filtro:<br>- La tabla muestra solo las facturas que cumplen el criterio<br>- Se muestra contador: "Mostrando X de Y facturas"<br>- Los filtros son acumulables con otros filtros de la tabla | ☐ | ☐ | ☐ |
| 17 | Exportar reporte de eventos | Backoffice necesita un reporte de eventos generados | Usuario solicita exportar información | El sistema permite a usuarios autorizados:<br>1. Exportar lista de eventos en formato Excel/CSV<br>2. El reporte incluye:<br>   - Número de factura<br>   - NIT emisor<br>   - NIT receptor<br>   - Fecha de vencimiento<br>   - Estado del evento<br>   - Fecha de generación<br>   - Número de intentos<br>   - Último error (si aplica)<br>   - Usuario que generó (Automático/Manual)<br>3. Permite filtrar por:<br>   - Rango de fechas<br>   - Estado del evento<br>   - Emisor<br>   - Receptor<br>4. El archivo se descarga inmediatamente | ☐ | ☐ | ☐ |
| 18 | Validar datos antes de enviar | El sistema prepara datos para enviar | Antes de enviar al servicio RADIAN | El sistema valida que:<br><br>**Validaciones obligatorias**:<br>1. supplierId (NIT emisor) existe y es válido<br>2. receiverId (NIT receptor) existe y es válido<br>3. documentId (número factura) no está vacío<br>4. CustomerEmail es un email válido<br>5. statusDate es una fecha válida<br><br>Si alguna validación falla:<br>- NO envía el evento<br>- Marca error: "Datos incompletos o inválidos"<br>- Especifica qué dato falta o es inválido<br>- Notifica a Backoffice para corrección<br>- Permite edición manual del dato<br><br>Esto previene errores evitables y rechazos del servicio | ☐ | ☐ | ☐ |
| 19 | Configuración del proceso automático | Administrador necesita configurar parámetros | Se requiere flexibilidad en la configuración | El sistema permite a administradores configurar:<br><br>**Parámetros configurables**:<br>- Hora de ejecución del proceso diario (ej: 06:00 AM)<br>- Tiempo de espera entre reintentos (actualmente 5, 15, 30 min)<br>- Número máximo de reintentos por día (actualmente 4)<br>- Email para notificaciones a Backoffice<br>- Días hábiles antes del vencimiento (actualmente 3)<br>- Lista de festivos colombianos (actualizable anualmente)<br><br>**Interfaz de configuración**:<br>- Panel de administración<br>- Cambios requieren confirmación<br>- Se registra quién y cuándo modificó<br>- Los cambios aplican desde el siguiente proceso | ☐ | ☐ | ☐ |
| 20 | Dashboard de monitoreo | Backoffice necesita vista general del proceso | Usuario accede a dashboard de eventos RADIAN | El sistema muestra un dashboard con:<br><br>**Indicadores clave**:<br>- Total de eventos generados hoy<br>- Total de eventos pendientes<br>- Total de eventos con error<br>- Facturas críticas (vencen en menos de 3 días sin evento)<br><br>**Gráficos**:<br>- Tendencia de generación últimos 30 días<br>- Tasa de éxito vs errores<br>- Tipos de errores más frecuentes<br><br>**Alertas activas**:<br>- Lista de facturas con error crítico<br>- Lista de facturas próximas a vencer sin evento<br><br>**Acciones rápidas**:<br>- Ver todas las facturas con error<br>- Ejecutar proceso manualmente (forzar)<br>- Descargar reporte del día | ☐ | ☐ | ☐ |

---

## Interacción con el usuario y prototipo

### Flujo del proceso automático:

```
┌──────────────────────────────────────┐
│   PROCESO DIARIO AUTOMÁTICO          │
│   Ejecuta: Todos los días 6:00 AM   │
└──────────────┬───────────────────────┘
               │
               ↓
┌──────────────────────────────────────┐
│ 1. Buscar facturas elegibles         │
│    - Estado: Desembolsado            │
│    - Vence en 3 días hábiles         │
│    - Sin evento generado             │
└──────────────┬───────────────────────┘
               │
               ↓
        ¿Hay facturas?
               │
        ┌──────┴──────┐
        NO            SÍ
        │              │
        ↓              ↓
  Finalizar    ┌──────────────────┐
   proceso     │ 2. Por cada      │
               │    factura       │
               └────┬─────────────┘
                    │
                    ↓
        ┌───────────────────────┐
        │ 3. Preparar datos     │
        │    - supplierId       │
        │    - receiverId       │
        │    - documentId       │
        │    - statusCode: 046  │
        │    - CustomerEmail    │
        └────┬──────────────────┘
             │
             ↓
        ¿Datos válidos?
             │
      ┌──────┴──────┐
      NO           SÍ
      │             │
      ↓             ↓
  Error:      ┌──────────────────┐
  Datos       │ 4. Enviar evento │
  inválidos   │    al servicio   │
      │       │    RADIAN        │
      │       └────┬─────────────┘
      │            │
      │            ↓
      │      ¿Exitoso?
      │            │
      │     ┌──────┴──────┐
      │     SÍ           NO
      │     │             │
      │     ↓             ↓
      │  ✅ Éxito    ⚠️ Error
      │     │             │
      │     ↓             ↓
      │  Registrar   ┌─────────────┐
      │  evento OK   │ 5. Reintentar│
      │     │        │    (máx 4    │
      │     │        │    intentos) │
      │     │        └─────┬────────┘
      │     │              │
      │     │              ↓
      │     │        ¿Se resolvió?
      │     │              │
      │     │       ┌──────┴──────┐
      │     │       SÍ           NO
      │     │       │             │
      │     │       ↓             ↓
      │     └────→ OK      Error temporal
      │                    "Reintentar mañana"
      │                           │
      │                           ↓
      │                    ¿Es 2do día
      │                    consecutivo?
      │                           │
      │                    ┌──────┴──────┐
      │                    NO           SÍ
      │                    │             │
      │                    ↓             ↓
      │              Reintentar    ❌ Error crítico
      │              mañana        Notificar Backoffice
      │                                   │
      └───────────────────────────────────┘
                    │
                    ↓
        ┌───────────────────────┐
        │ 6. Siguiente factura  │
        └───────────────────────┘
```

### Pantalla - Pestaña Desembolsada:

```
┌────────────────────────────────────────────────────────────────────┐
│  Facturas Desembolsadas                               [Exportar ▾] │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Filtros: [Emisor ▾] [Receptor ▾] [Fechas] [Evento Informe Pago ▾]│
│                                                                    │
│  Mostrando 125 facturas                                            │
│                                                                    │
├────────────────────────────────────────────────────────────────────┤
│ No.      │ Emisor    │ Receptor  │ Monto     │ Vence    │ Evento  │
│ Factura  │           │           │           │          │ Inf.Pago│
├──────────┼───────────┼───────────┼───────────┼──────────┼─────────┤
│ SETT4890 │ CADENA    │ PRESIZA   │ $1.5M     │ 08/02/26 │ ✅      │
│ 161      │ 830507412 │ 890930534 │           │          │ Generado│
│          │           │           │           │          │ 05/02/26│
├──────────┼───────────┼───────────┼───────────┼──────────┼─────────┤
│ PR-456   │ ENTREGA   │ DINDELCO  │ $2.3M     │ 10/02/26 │ ⏳      │
│          │ 800114437 │ 900439177 │           │          │Pendiente│
│          │           │           │           │          │ Gen:07/02│
├──────────┼───────────┼───────────┼───────────┼──────────┼─────────┤
│ FC-789   │ DISTRIB   │ ALMACENES │ $890K     │ 06/02/26 │ ⚠️      │
│          │ 890123456 │ 901234567 │           │          │Reintent.│
│          │           │           │           │          │ 2 intent│
├──────────┼───────────┼───────────┼───────────┼──────────┼─────────┤
│ INV-1001 │ PROVEEDOR │ COMPRADOR │ $1.2M     │ 05/02/26 │ ❌ Error│
│          │ 800999888 │ 890777666 │           │          │ Crítico │
│          │           │           │           │          │[Detalles│
└────────────────────────────────────────────────────────────────────┘

Leyenda:
✅ = Evento generado exitosamente
⏳ = Pendiente de generar (se generará automáticamente)
⚠️ = Error temporal, se reintentará
❌ = Error crítico, requiere intervención de Backoffice
```

### Modal - Detalle del Evento:

```
┌──────────────────────────────────────────────────────┐
│  Detalle Evento RADIAN - Informe para Pago      [X] │
├──────────────────────────────────────────────────────┤
│                                                      │
│  📄 Factura: SETT4890161                            │
│  Estado: ❌ Error crítico - Requiere intervención   │
│  Vencimiento: 05/02/2026                             │
│  Días hábiles restantes: 1 día                       │
│                                                      │
├──────────────────────────────────────────────────────┤
│  HISTORIAL DE INTENTOS                               │
├──────────────────────────────────────────────────────┤
│                                                      │
│  📅 03/02/2026 - 06:00 AM                           │
│  └─ ❌ Error: Timeout de conexión                   │
│      Reintento en 5 min...                           │
│                                                      │
│  📅 03/02/2026 - 06:05 AM                           │
│  └─ ❌ Error: Servicio no disponible (HTTP 503)     │
│      Reintento en 15 min...                          │
│                                                      │
│  📅 03/02/2026 - 06:20 AM                           │
│  └─ ❌ Error: Timeout de conexión                   │
│      Reintento en 30 min...                          │
│                                                      │
│  📅 03/02/2026 - 06:50 AM                           │
│  └─ ❌ Error: Servicio no disponible (HTTP 503)     │
│      Se reintentará mañana                           │
│                                                      │
│  ─────────────────────────────────────────────────   │
│                                                      │
│  📅 04/02/2026 - 06:00 AM                           │
│  └─ ❌ Error: Servicio no disponible (HTTP 503)     │
│      ERROR CRÍTICO - 2do día consecutivo             │
│                                                      │
│  📧 Backoffice notificado: 04/02/2026 06:01 AM      │
│                                                      │
├──────────────────────────────────────────────────────┤
│  DATOS ENVIADOS (último intento)                     │
├──────────────────────────────────────────────────────┤
│  {                                                   │
│    "supplierId": "830507412",                        │
│    "receiverId": "890930534",                        │
│    "documentTypeCode": "01",                         │
│    "documentId": "SETT4890161",                      │
│    "factorId": "89093534",                           │
│    "username": "",                                   │
│    "documentStatus": {                               │
│      "statusCode": "046",                            │
│      "statusDate": 1738659600,                       │
│      "statusReason": "",                             │
│      "statusNote": "",                               │
│      "CustomerEmail": "cliente@empresa.com"          │
│    }                                                 │
│  }                                                   │
│                                                      │
├──────────────────────────────────────────────────────┤
│                                                      │
│  [Copiar datos]  [Descargar PDF]  [Reintentar] ⚠️   │
│                                                      │
└──────────────────────────────────────────────────────┘
```

### Notificación Email a Backoffice:

```
De: sistema@axces.com.co
Para: backoffice@axces.com.co
Asunto: ⚠️ Error crítico: Evento RADIAN no generado - Factura SETT4890161

──────────────────────────────────────────────────
ALERTA: Evento RADIAN Informe para Pago no generado
Error persistente - Requiere intervención manual
──────────────────────────────────────────────────

INFORMACIÓN DE LA FACTURA:
──────────────────────────────────────────────────
Número: SETT4890161
NIT Emisor: CADENA S.A. (830507412)
NIT Receptor: PRESIZA S.A.S (890930534)
Monto: $1,500,000.00 COP
Fecha de vencimiento: 05/02/2026
Días hábiles restantes: 1 día

⚠️ URGENTE: Esta factura vence muy pronto

HISTORIAL DE ERRORES:
──────────────────────────────────────────────────
Día 1 (03/02/2026):
- 4 intentos realizados
- Todos fallaron por: Servicio no disponible (HTTP 503)

Día 2 (04/02/2026):
- 4 intentos realizados
- Todos fallaron por: Servicio no disponible (HTTP 503)

ÚLTIMO ERROR:
Servicio no disponible (HTTP 503)
Código de respuesta: SERVICE_UNAVAILABLE
Mensaje: El servicio RADIAN está temporalmente no disponible

ACCIÓN REQUERIDA:
──────────────────────────────────────────────────
1. Verificar disponibilidad del servicio RADIAN
2. Revisar los datos de la factura
3. Intentar generación manual desde la plataforma
4. Si persiste el error, contactar con soporte RADIAN

ACCESO RÁPIDO:
Ver detalles completos: https://axces.com/facturas/SETT4890161/eventos

──────────────────────────────────────────────────
Este es un mensaje automático del sistema Axces
```

### Dashboard de Monitoreo:

```
┌────────────────────────────────────────────────────────────────┐
│  Dashboard - Eventos RADIAN Informe para Pago            [⟳] │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  📊 RESUMEN HOY (04/02/2026)                                  │
│  ┌──────────────┬──────────────┬──────────────┬─────────────┐│
│  │ ✅ Generados │ ⏳ Pendientes│ ⚠️ Con Error │ ❌ Críticos ││
│  │      45      │      12      │       3      │      2      ││
│  └──────────────┴──────────────┴──────────────┴─────────────┘│
│                                                                │
│  📈 TENDENCIA ÚLTIMOS 30 DÍAS                                 │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  50│                                      ✓             │  │
│  │  40│              ✓    ✓         ✓      ✓              │  │
│  │  30│        ✓   ✓  ✓ ✓  ✓     ✓  ✓   ✓                │  │
│  │  20│    ✓ ✓  ✓           ✓   ✓                         │  │
│  │  10│  ✓                                                 │  │
│  │   0└────────────────────────────────────────────────────│  │
│  │     05  10  15  20  25  30  01  (Febrero)              │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                                │
│  ⚠️ FACTURAS CRÍTICAS (Requieren Atención)                   │
│  ┌──────────────────────────────────────────────────────┐    │
│  │ SETT4890161  │ Vence: 05/02 │ Error 2 días │ [Ver] │    │
│  │ INV-1001     │ Vence: 05/02 │ Error 2 días │ [Ver] │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                                │
│  📋 ACCIONES RÁPIDAS                                          │
│  [Ver todas con error] [Forzar proceso] [Descargar reporte]  │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Consideraciones de UX:

- **Visibilidad clara**: Estados del evento fácilmente identificables con iconos y colores
- **Información contextual**: Tooltips que explican cada estado
- **Acceso a detalles**: Un clic para ver historial completo
- **Filtros útiles**: Encontrar rápidamente facturas con problemas
- **Notificaciones proactivas**: Alertar a Backoffice antes de que sea tarde
- **Dashboard centralizado**: Vista general para toma de decisiones
- **Trazabilidad completa**: Registro de todos los intentos y resultados
- **Acciones rápidas**: Reintentos manuales desde la misma interfaz

---

## Definición de Terminado (DoD)

- [ ] El código cumple con los estándares de desarrollo del proyecto
- [ ] Se han implementado todos los escenarios de aceptación (20 escenarios)
- [ ] El proceso automático se ejecuta diariamente a la hora configurada
- [ ] El cálculo de días hábiles funciona correctamente
- [ ] El calendario de festivos colombianos está implementado
- [ ] La identificación de facturas elegibles es precisa
- [ ] La preparación de datos genera el JSON correcto
- [ ] El envío al servicio RADIAN funciona correctamente
- [ ] El sistema de reintentos funciona con los tiempos configurados
- [ ] Las notificaciones a Backoffice se envían correctamente
- [ ] La trazabilidad completa se registra en el sistema
- [ ] La columna "Evento Informe Pago" se muestra en la pestaña Desembolsada
- [ ] Los estados visuales (iconos, colores) son correctos
- [ ] El modal de detalle muestra toda la información
- [ ] Los filtros por estado del evento funcionan
- [ ] La exportación de reportes funciona
- [ ] El reintento manual desde Backoffice funciona
- [ ] El dashboard de monitoreo muestra datos en tiempo real
- [ ] Las validaciones de datos previas al envío funcionan
- [ ] El panel de configuración permite ajustar parámetros
- [ ] Las pruebas unitarias tienen una cobertura mínima del 80%
- [ ] Las pruebas de integración con el servicio RADIAN funcionan
- [ ] Se han realizado pruebas con diferentes escenarios de error
- [ ] El proceso maneja correctamente timeouts y errores de red
- [ ] La documentación técnica está actualizada
- [ ] El Product Owner ha validado la funcionalidad
- [ ] No existen bugs críticos pendientes

---

## Formato de Datos para el Servicio RADIAN

### Estructura del JSON a enviar:

```json
{
  "supplierId": "830507412",
  "receiverId": "890930534",
  "documentTypeCode": "01",
  "documentId": "SETT4890161",
  "factorId": "89093534",
  "username": "",
  "documentStatus": {
    "statusCode": "046",
    "statusDate": 1738659600,
    "statusReason": "",
    "statusNote": "",
    "CustomerEmail": "cliente@empresa.com"
  }
}
```

### Descripción de campos:

| Campo | Descripción | Origen | Tipo | Requerido |
|-------|-------------|--------|------|-----------|
| supplierId | NIT del emisor de la factura | Base de datos de la factura | String | Sí |
| receiverId | NIT del receptor de la factura | Base de datos de la factura | String | Sí |
| documentTypeCode | Tipo de documento | **Valor fijo: "01"** (Factura) | String | Sí |
| documentId | Número de la factura | Base de datos de la factura | String | Sí |
| factorId | NIT del factor | **Valor fijo configurado** | String | Sí |
| username | Usuario (no usado actualmente) | **Valor fijo: ""** | String | Sí |
| statusCode | Código del evento RADIAN | **Valor fijo: "046"** (Informe Pago) | String | Sí |
| statusDate | Fecha y hora del evento | Timestamp actual al generar | Integer | Sí |
| statusReason | Razón del estado (no usado) | **Valor fijo: ""** | String | Sí |
| statusNote | Nota adicional (no usado) | **Valor fijo: ""** | String | Sí |
| CustomerEmail | Email del cliente/emisor | Base de datos del cliente | String | Sí |

### Valores fijos configurados:

- **documentTypeCode**: "01"
- **statusCode**: "046"
- **factorId**: Valor a configurar según el factor (el equipo de desarrollo debe parametrizarlo)
- **username**: ""
- **statusReason**: ""
- **statusNote**: ""

---

## Notas Funcionales

### Días hábiles:

- **Lunes a Viernes**: Días hábiles
- **Sábados y Domingos**: NO son hábiles
- **Festivos colombianos**: NO son hábiles (debe mantenerse calendario actualizado)
- El sistema debe contar hacia atrás desde la fecha de vencimiento excluyendo fines de semana y festivos

### Estados de la factura:

- Solo se procesan facturas en estado **"Desembolsado"**
- Si cambia de estado después de generar el evento, el evento ya enviado NO se invalida

### Sistema de reintentos:

- **4 intentos máximo por día**
- Tiempos de espera: 5 min, 15 min, 30 min
- Si falla el día 1: Se reintenta al día siguiente
- Si falla día 1 Y día 2: Se notifica a Backoffice y se marca como crítico

### Notificaciones:

- **A Backoffice**: Solo cuando hay error crítico (2 días consecutivos) o falta información
- **Formato**: Email con detalles completos
- **Urgencia**: Si la factura vence en menos de 2 días, marcar como urgente

### Datos obligatorios:

- Todas las facturas deben tener **CustomerEmail** para poder generar el evento
- Si falta este dato, se notifica inmediatamente para que se complete

---

## Dependencias

- **Servicio de Generación de Eventos RADIAN**: Servicio externo que recibe los eventos
- **Calendario de festivos colombianos**: Fuente actualizada de días festivos
- **Sistema de notificaciones**: Para enviar emails a Backoffice
- **Sistema de logs/auditoría**: Para registrar trazabilidad completa
- **Proceso programado (Cron/Scheduler)**: Para ejecutar el proceso diario automático

---

## Riesgos y Mitigaciones

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|--------------|---------|------------|
| Servicio RADIAN no disponible | Media | Alto | Sistema de reintentos con backoff; notificación a Backoffice; reintento al día siguiente |
| Factura sin email del cliente | Media | Medio | Validación previa; notificación inmediata para completar datos; no bloquear otras facturas |
| Cálculo incorrecto de días hábiles | Baja | Alto | Validación exhaustiva del algoritmo; calendario de festivos actualizado; casos de prueba completos |
| Proceso diario no se ejecuta | Baja | Crítico | Monitoreo del proceso; alertas si no se ejecuta; logs de ejecución; posibilidad de ejecutar manualmente |
| Reintentos consumen muchos recursos | Media | Medio | Límite de reintentos; tiempos de espera exponenciales; procesamiento asíncrono |
| Notificaciones no llegan a Backoffice | Baja | Alto | Múltiples canales; confirmación de envío; dashboard visible; logs de notificaciones |
| Datos de factura incompletos | Media | Medio | Validación previa al envío; especificar qué datos faltan; permitir completar y reintentar |
| Festivos desactualizados | Baja | Medio | Recordatorio anual para actualizar; interfaz de administración; cálculo conservador |
| Duplicación de eventos | Baja | Alto | Validar que no exista evento previo exitoso; constraint en BD; verificación antes de enviar |

---

## Casos de Prueba Sugeridos

### Pruebas Funcionales:

1. **Generación exitosa básica**: Factura desembolsada que vence en 3 días hábiles exactos
2. **Cálculo de días hábiles con fin de semana**: Factura que vence un martes (debe generarse jueves anterior)
3. **Cálculo con festivo**: Factura que vence después de un festivo
4. **Factura sin email**: Intentar generar evento para factura sin CustomerEmail
5. **Reintento exitoso en segundo intento**: Primer intento falla, segundo es exitoso
6. **Error crítico 2 días**: Falla día 1 y día 2, debe notificar a Backoffice
7. **Factura ya con evento**: No debe generar duplicado
8. **Factura cambia de estado**: Evento ya generado debe permanecer
9. **Validación de datos inválidos**: NIT inválido o email mal formado
10. **Filtros en interfaz**: Filtrar por cada estado del evento
11. **Reintento manual**: Backoffice reintenta factura con error crítico
12. **Exportación de reporte**: Exportar lista de eventos en Excel
13. **Dashboard actualizado**: Verificar que métricas son correctas
14. **Proceso no encuentra facturas**: Día sin facturas elegibles
15. **Múltiples facturas en un día**: 50+ facturas elegibles en el mismo día

### Pruebas de Integración:

1. Verificar que el JSON enviado tiene el formato correcto
2. Verificar respuesta del servicio RADIAN cuando es exitosa
3. Verificar manejo de diferentes códigos de error del servicio
4. Verificar que notificaciones a Backoffice se envían correctamente
5. Verificar que el proceso programado se ejecuta a la hora correcta
6. Verificar trazabilidad completa en BD después de cada intento
7. Verificar que cambios de configuración se aplican correctamente

### Pruebas de Regresión:

1. Facturas desembolsadas antes de implementar esta funcionalidad
2. Eventos generados manualmente antes vs automáticos después
3. Interfaz de pestaña Desembolsada con nueva columna
4. Rendimiento con gran volumen de facturas

### Pruebas de Rendimiento:

1. Procesar 100+ facturas en un solo día
2. Tiempo de respuesta del servicio RADIAN
3. Tiempo total de ejecución del proceso diario
4. Carga en BD por registro de trazabilidad

---

**Fecha de creación**: 2026-02-03
**Última actualización**: 2026-02-03
**Versión**: 1.0
