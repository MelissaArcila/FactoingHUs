# Edición Masiva de Pagadores mediante Plantilla Excel

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
| **Arquitectura relacionada** | |

---

## Contexto

### Enunciado general de la historia

Como usuario administrador del sistema de factoring, quiero poder **crear y/o actualizar múltiples pagadores de forma masiva** mediante una plantilla de Excel, para agilizar el proceso de gestión de pagadores sin tener que crear o editar cada uno individualmente, permitiendo que si un pagador ya existe se actualice y si no existe se cree.

### Roles
- **Administrador**: Usuario con permisos para realizar carga masiva de pagadores
- **Operaciones**: Usuario con permisos para realizar carga masiva de pagadores

### Característica / Funcionalidad
Carga masiva de pagadores (creación y actualización) mediante plantilla Excel con validación de datos y auditoría completa

### Razón / Resultado
Optimizar el proceso de gestión de pagadores permitiendo la creación y actualización simultánea de múltiples registros, eliminando el error actual que rechaza pagadores existentes y permitiendo mantener la información actualizada de forma eficiente y trazable.

---

## Escenarios

| Número | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|--------|--------------------------------|----------|--------|-------------------------------------|------------|----|--------------|
| 1 | Descargar plantilla de Excel para carga masiva | El usuario con permisos accede a la funcionalidad de gestión de pagadores | El usuario hace clic en el botón "Descargar Plantilla" o "Exportar Plantilla" | Se descarga un archivo Excel (.xlsx) con el nombre "Plantilla_Pagadores.xlsx" que contiene:<br>- Encabezado con logo e instrucciones<br>- Instrucciones de formato: "Por favor, ingrese las tasas con 4 decimales (ej. 4,0000) y los montos monetarios con 2 decimales (ej. 100000,00)"<br>- Fila de encabezados con las columnas:<br>  * NIT Pagador*<br>  * Razón Social*<br>  * Dirección oficina línea 1<br>  * Dirección oficina línea 2<br>  * Ciudad de la oficina<br>  * Departamento / Estado de la oficina<br>  * Código postal oficina<br>  * País de la oficina<br>  * Email de la oficina<br>  * Tasa de descuento por defecto*<br>  * Tasa de desembolso por defecto*<br>- Campos marcados con * son obligatorios | ☐ | ☐ | ☐ |
| 2 | Cargar archivo Excel con pagadores | El usuario ha descargado y llenado la plantilla con datos de pagadores | El usuario selecciona el archivo Excel y hace clic en "Cargar" o "Subir archivo" | El sistema:<br>1. Valida que el archivo sea formato Excel (.xlsx o .xls)<br>2. Lee el contenido del archivo<br>3. Muestra indicador de carga/procesamiento<br>4. Valida la estructura de la plantilla (columnas esperadas)<br>5. Procesa cada fila del archivo<br>6. Muestra vista previa o resumen de la carga antes de confirmar | ☐ | ☐ | ☐ |
| 3 | Validación de campos obligatorios en la plantilla | El archivo Excel contiene filas con campos obligatorios vacíos | El sistema procesa el archivo | El sistema:<br>- Identifica las filas con errores de campos obligatorios<br>- Genera reporte de errores indicando:<br>  * Número de fila<br>  * Campo faltante<br>  * Mensaje: "El campo [nombre campo] es obligatorio"<br>- Campos obligatorios:<br>  * NIT Pagador<br>  * Razón Social<br>  * Tasa de descuento por defecto<br>  * Tasa de desembolso por defecto<br>- Los campos opcionales pueden estar vacíos<br>- No se procesa la carga hasta corregir los errores | ☐ | ☐ | ☐ |
| 4 | Validación de formato de datos | El archivo contiene datos con formato incorrecto | El sistema valida cada fila | El sistema valida:<br><br>**NIT Pagador**:<br>- Solo números<br>- Longitud según esquema de BD<br><br>**Tasas (descuento y desembolso)**:<br>- Formato numérico decimal<br>- 4 decimales (ej: 4,0000 o 4.0000)<br>- Tasa descuento >= 0<br>- Tasa desembolso >= 0 y <= 100<br><br>**Montos monetarios** (si aplica):<br>- 2 decimales (ej: 100000,00)<br><br>**Email**:<br>- Formato válido de email (si no está vacío)<br><br>Si hay errores, genera reporte con:<br>- Fila con error<br>- Campo inválido<br>- Valor ingresado<br>- Mensaje de error específico | ☐ | ☐ | ☐ |
| 5 | Crear nuevo pagador que no existe en BD | La plantilla contiene un pagador con NIT que no existe en la base de datos | El sistema procesa la fila | El sistema:<br>1. Verifica que el NIT no existe en BD<br>2. Crea un nuevo registro de pagador con todos los datos de la plantilla<br>3. Registra en auditoría:<br>   - Acción: "CREACION_MASIVA"<br>   - Usuario que realizó la carga<br>   - Fecha y hora<br>   - Todos los datos del nuevo pagador<br>4. Incluye el registro en el resumen de resultados como "Creado" | ☐ | ☐ | ☐ |
| 6 | Actualizar pagador existente en BD | La plantilla contiene un pagador con NIT que ya existe en la base de datos | El sistema procesa la fila | El sistema:<br>1. Verifica que el NIT existe en BD<br>2. Compara los datos actuales con los datos de la plantilla<br>3. Actualiza TODOS los campos del pagador con los valores de la plantilla<br>4. Registra en auditoría:<br>   - Acción: "ACTUALIZACION_MASIVA"<br>   - Usuario que realizó la carga<br>   - Fecha y hora<br>   - Valores anteriores (before)<br>   - Valores nuevos (after)<br>   - Campos modificados<br>5. Incluye el registro en el resumen como "Actualizado" | ☐ | ☐ | ☐ |
| 7 | Procesar archivo con mezcla de creaciones y actualizaciones | La plantilla contiene tanto pagadores nuevos como existentes | El usuario carga el archivo y confirma | El sistema:<br>- Procesa todas las filas válidas<br>- Crea los pagadores nuevos (NITs no existentes)<br>- Actualiza los pagadores existentes (NITs ya registrados)<br>- Genera un solo registro de auditoría por cada operación<br>- NO falla ni se detiene si encuentra pagadores existentes<br>- Procesa de forma transaccional o con manejo de errores por fila | ☐ | ☐ | ☐ |
| 8 | Mostrar resumen de resultados después de la carga | El archivo ha sido procesado completamente | El sistema finaliza el procesamiento | Se muestra una pantalla/modal de resumen con:<br><br>**Estadísticas**:<br>- Total de filas procesadas<br>- Pagadores creados: X<br>- Pagadores actualizados: Y<br>- Errores: Z<br><br>**Detalle por tipo**:<br>- Lista de pagadores creados (Razón Social, NIT)<br>- Lista de pagadores actualizados (Razón Social, NIT, campos modificados)<br>- Lista de errores (Fila, NIT, Razón Social, Error)<br><br>**Opciones**:<br>- Descargar reporte detallado (Excel o PDF)<br>- Botón "Cerrar" o "Aceptar" | ☐ | ☐ | ☐ |
| 9 | Manejo de errores parciales en el archivo | El archivo contiene algunas filas válidas y otras con errores | El sistema procesa el archivo | El sistema debe:<br><br>**Opción A (Recomendada)**: Validación previa<br>- Validar TODAS las filas antes de procesar<br>- Si hay errores, mostrar reporte completo de errores<br>- NO procesar ninguna fila hasta que se corrijan todos los errores<br>- Permitir descargar archivo con marcas de error<br><br>**Opción B**: Procesamiento parcial<br>- Procesar solo las filas válidas<br>- Omitir filas con errores<br>- Mostrar resumen de éxitos y errores<br>- Permitir corregir y recargar solo las filas fallidas<br><br>(Definir cuál opción implementar según requerimiento de negocio) | ☐ | ☐ | ☐ |
| 10 | Validación de duplicados dentro del mismo archivo | La plantilla contiene múltiples filas con el mismo NIT | El sistema valida el archivo | El sistema debe:<br>- Detectar NITs duplicados dentro del archivo<br>- Mostrar error: "El NIT [número] aparece en las filas [X, Y, Z]"<br>- Indicar al usuario que debe dejar solo una fila por NIT<br>- No procesar el archivo hasta corregir duplicados<br>- Opcionalmente: Procesar solo la primera ocurrencia y advertir sobre las demás | ☐ | ☐ | ☐ |
| 11 | Validación de permisos de usuario | Un usuario sin permisos intenta acceder a la carga masiva | El usuario intenta acceder a la funcionalidad | El sistema debe:<br>- Verificar que el usuario tiene rol de Administrador u Operaciones<br>- Si no tiene permisos:<br>  * Ocultar o deshabilitar el botón de carga masiva<br>  * Mostrar mensaje: "No tiene permisos para realizar carga masiva"<br>- No permitir el acceso a la funcionalidad | ☐ | ☐ | ☐ |
| 12 | Manejo de archivo muy grande | El usuario intenta cargar un archivo con más de 1000 filas (límite a definir) | El sistema recibe el archivo | El sistema debe:<br>- Validar el tamaño/número de filas del archivo<br>- Si excede el límite:<br>  * Mostrar advertencia: "El archivo excede el límite de [X] registros. Por favor, divida la carga en múltiples archivos"<br>  * No procesar el archivo<br>- Procesar el archivo por lotes/chunks para evitar timeout<br>- Mostrar barra de progreso durante el procesamiento | ☐ | ☐ | ☐ |
| 13 | Cancelar carga antes de confirmar | El usuario ha cargado el archivo y ve la vista previa | El usuario hace clic en "Cancelar" antes de confirmar | El sistema:<br>- Descarta todos los cambios propuestos<br>- No crea ni actualiza ningún pagador<br>- No registra nada en auditoría<br>- Cierra la vista previa y regresa a la pantalla de gestión de pagadores | ☐ | ☐ | ☐ |
| 14 | Confirmar carga masiva | El usuario ha revisado la vista previa/resumen y está conforme | El usuario hace clic en "Confirmar" o "Procesar" | El sistema:<br>- Ejecuta todas las operaciones (creaciones y actualizaciones)<br>- Muestra indicador de progreso<br>- Registra todas las auditorías<br>- Actualiza la tabla de pagadores con los nuevos/modificados datos<br>- Muestra resumen final de resultados<br>- Envía notificación de éxito | ☐ | ☐ | ☐ |
| 15 | Validación de estructura de plantilla modificada | El usuario modifica la estructura de la plantilla (agrega/elimina columnas, cambia nombres) | El sistema valida el archivo cargado | El sistema debe:<br>- Verificar que todas las columnas obligatorias existen<br>- Verificar que los nombres de columnas coinciden exactamente<br>- Si la estructura no coincide:<br>  * Mostrar error: "La plantilla ha sido modificada. Por favor, descargue la plantilla oficial"<br>  * Listar las columnas faltantes o incorrectas<br>  * No procesar el archivo<br>- Ser flexible con columnas adicionales (ignorarlas o alertar) | ☐ | ☐ | ☐ |
| 16 | Actualización de tabla después de carga exitosa | La carga masiva se completó exitosamente | El usuario cierra el resumen de resultados | El sistema:<br>- Actualiza automáticamente la tabla de pagadores<br>- Muestra los pagadores creados/actualizados con sus nuevos valores<br>- No requiere recarga manual de la página<br>- Los cambios son visibles inmediatamente para todos los usuarios | ☐ | ☐ | ☐ |

---

## Interacción con el usuario y prototipo

### Flujo de interacción completo:

#### 1. **Pantalla de Gestión de Pagadores**
```
┌────────────────────────────────────────────────────────┐
│ Gestión de Pagadores                      [Descargar  │
│                                            Plantilla]  │
│                                           [Carga       │
│                                            Masiva]     │
├────────────────────────────────────────────────────────┤
│ [Tabla de pagadores existentes]                       │
└────────────────────────────────────────────────────────┘
```

#### 2. **Descargar Plantilla**
Al hacer clic en "Descargar Plantilla":
- Se descarga: `Plantilla_Pagadores.xlsx`
- Estructura de la plantilla:

```
┌─────────────────────────────────────────────────────────────┐
│ [Logo CDN Factoring]          Crear Pagadores               │
├─────────────────────────────────────────────────────────────┤
│ Ingrese los datos de este formato y guardelo como Libro    │
│ de Excel (.xlsx)                                            │
│                                                             │
│ Por favor, ingrese las tasas con 4 decimales (ej. 4,0000) │
│ y los montos monetarios con 2 decimales (ej. 100000,00).  │
├─────────────────────────────────────────────────────────────┤
│ NIT     │ Razón    │ Dirección │ Dirección │ Ciudad │ ... │
│ Pagador*│ Social*  │ oficina   │ oficina   │ de la  │     │
│         │          │ línea 1   │ línea 2   │ oficina│     │
├─────────────────────────────────────────────────────────────┤
│         │          │           │           │        │     │
│         │          │           │           │        │     │
└─────────────────────────────────────────────────────────────┘
```

**Columnas completas**:
1. NIT Pagador* (obligatorio)
2. Razón Social* (obligatorio)
3. Dirección oficina línea 1
4. Dirección oficina línea 2
5. Ciudad de la oficina
6. Departamento / Estado de la oficina
7. Código postal oficina
8. País de la oficina
9. Email de la oficina
10. Tasa de descuento por defecto* (obligatorio, 4 decimales)
11. Tasa de desembolso por defecto* (obligatorio, 4 decimales)

#### 3. **Modal de Carga Masiva**
```
┌──────────────────────────────────────────────┐
│  Carga Masiva de Pagadores              [X] │
├──────────────────────────────────────────────┤
│                                              │
│  1. Descargue la plantilla                   │
│     [Descargar Plantilla]                    │
│                                              │
│  2. Complete los datos en la plantilla       │
│     • Campos con * son obligatorios          │
│     • Tasas con 4 decimales (ej: 4,0000)    │
│     • Montos con 2 decimales (ej: 100000,00)│
│                                              │
│  3. Cargue el archivo                        │
│     ┌────────────────────────────┐           │
│     │ Arrastre el archivo aquí   │           │
│     │ o haga clic para seleccionar│          │
│     └────────────────────────────┘           │
│     [Seleccionar archivo]                    │
│                                              │
│     Archivo seleccionado:                    │
│     📄 Plantilla_Pagadores.xlsx              │
│     Tamaño: 15 KB | 10 registros             │
│                                              │
│           [Cancelar]    [Procesar]           │
└──────────────────────────────────────────────┘
```

#### 4. **Vista Previa / Validación**
```
┌──────────────────────────────────────────────┐
│  Validando archivo...                        │
├──────────────────────────────────────────────┤
│  ⏳ Procesando 10 registros...               │
│  ████████████████████░░░░░░░  80%            │
└──────────────────────────────────────────────┘
```

#### 5. **Resumen de Validación (con errores)**
```
┌──────────────────────────────────────────────┐
│  Errores de Validación                  [X] │
├──────────────────────────────────────────────┤
│  ⚠️ Se encontraron 3 errores en el archivo  │
│                                              │
│  Fila 5: NIT Pagador es obligatorio          │
│  Fila 7: Tasa de descuento debe ser >= 0    │
│  Fila 9: Email inválido: "correo.com"       │
│                                              │
│  Por favor corrija los errores y vuelva a   │
│  cargar el archivo.                          │
│                                              │
│  [Descargar reporte de errores]              │
│                                              │
│              [Cerrar]                        │
└──────────────────────────────────────────────┘
```

#### 6. **Resumen de Vista Previa (sin errores)**
```
┌──────────────────────────────────────────────┐
│  Confirmar Carga Masiva                 [X] │
├──────────────────────────────────────────────┤
│  Resumen de cambios:                         │
│                                              │
│  📊 Total de registros: 10                   │
│  ✅ Pagadores a crear: 6                     │
│  🔄 Pagadores a actualizar: 4                │
│                                              │
│  ──────────────────────────────────────      │
│  Pagadores a crear (6):                      │
│  • NUEVA EMPRESA S.A. (NIT: 900123456)      │
│  • COMERCIAL XYZ (NIT: 800987654)           │
│  • ... (ver más)                            │
│                                              │
│  Pagadores a actualizar (4):                 │
│  • CADENA S.A. (NIT: 890930534)             │
│    - Tasa descuento: 2.0 → 2.5              │
│    - Email: test@gmail.com → nuevo@email.com│
│  • ... (ver más)                            │
│                                              │
│  ⚠️ Esta acción no se puede deshacer        │
│                                              │
│  [Descargar vista previa]                    │
│                                              │
│         [Cancelar]    [Confirmar]            │
└──────────────────────────────────────────────┘
```

#### 7. **Procesamiento**
```
┌──────────────────────────────────────────────┐
│  Procesando carga masiva...                  │
├──────────────────────────────────────────────┤
│  ⏳ Guardando cambios en base de datos...    │
│  ████████████████████████████  100%          │
│                                              │
│  Creados: 6/6                                │
│  Actualizados: 4/4                           │
└──────────────────────────────────────────────┘
```

#### 8. **Resumen Final de Resultados**
```
┌──────────────────────────────────────────────┐
│  ✅ Carga Masiva Completada             [X] │
├──────────────────────────────────────────────┤
│  La carga se completó exitosamente           │
│                                              │
│  📊 Resultados:                              │
│  ────────────────────────                    │
│  Total procesados:        10                 │
│  ✅ Creados:              6                  │
│  🔄 Actualizados:         4                  │
│  ❌ Errores:              0                  │
│                                              │
│  Detalle de creados (6):                     │
│  • NUEVA EMPRESA S.A. (900123456)           │
│  • COMERCIAL XYZ (800987654)                │
│  • ...                                      │
│                                              │
│  Detalle de actualizados (4):                │
│  • CADENA S.A. (890930534)                  │
│    Campos: Tasa descuento, Email            │
│  • ...                                      │
│                                              │
│  [Descargar reporte completo]                │
│                                              │
│                [Cerrar]                      │
└──────────────────────────────────────────────┘
```

### Consideraciones de UX:

- **Validación progresiva**: Validar estructura → validar datos → mostrar vista previa → procesar
- **Feedback claro**: Mostrar exactamente qué se va a crear/actualizar antes de confirmar
- **Manejo de errores detallado**: Indicar fila, campo y error específico
- **Progreso visible**: Barra de progreso durante validación y procesamiento
- **Reversibilidad**: Advertir que la acción no se puede deshacer
- **Descargables**: Permitir descargar reportes de errores y resultados
- **Auditoría transparente**: Registrar todo para trazabilidad

---

## Definición de Terminado (DoD)

- [ ] El código cumple con los estándares de desarrollo del proyecto
- [ ] Se han implementado todos los escenarios de aceptación
- [ ] La plantilla Excel se genera correctamente con formato e instrucciones
- [ ] El sistema maneja correctamente creaciones y actualizaciones (upsert)
- [ ] Las validaciones de datos funcionan correctamente
- [ ] Los registros de auditoría se crean para todas las operaciones
- [ ] El manejo de errores es robusto (errores parciales, archivo grande, etc.)
- [ ] Las pruebas unitarias tienen una cobertura mínima del 80%
- [ ] Las pruebas de integración funcionan correctamente
- [ ] Se probó con archivos grandes (1000+ registros)
- [ ] La documentación técnica está actualizada
- [ ] El Product Owner ha validado la funcionalidad
- [ ] No existen bugs críticos pendientes

---

## Notas Técnicas

### Endpoint API (sugerido):

#### Descargar plantilla
```
GET /api/pagadores/plantilla/descargar
```

**Response**: Archivo Excel

#### Validar archivo
```
POST /api/pagadores/carga-masiva/validar
Content-Type: multipart/form-data
```

**Request**:
```
file: [archivo Excel]
```

**Response**:
```json
{
  "valid": true,
  "summary": {
    "totalRows": 10,
    "toCreate": 6,
    "toUpdate": 4,
    "errors": 0
  },
  "preview": {
    "creates": [
      {
        "nit": "900123456",
        "razonSocial": "NUEVA EMPRESA S.A.",
        "tasaDescuento": 4.0000,
        "tasaDesembolso": 90.0000
      }
    ],
    "updates": [
      {
        "nit": "890930534",
        "razonSocial": "CADENA S.A.",
        "changes": {
          "tasaDescuento": { "from": 2.0, "to": 2.5 },
          "email": { "from": "test@gmail.com", "to": "nuevo@email.com" }
        }
      }
    ]
  },
  "errors": []
}
```

#### Procesar carga masiva
```
POST /api/pagadores/carga-masiva/procesar
```

**Request**:
```json
{
  "file": "base64_encoded_file",
  "usuarioId": "user_id",
  "confirmarProcesamiento": true
}
```

**Response**:
```json
{
  "success": true,
  "results": {
    "totalProcessed": 10,
    "created": 6,
    "updated": 4,
    "failed": 0
  },
  "details": {
    "created": [
      { "nit": "900123456", "razonSocial": "NUEVA EMPRESA S.A." }
    ],
    "updated": [
      { "nit": "890930534", "razonSocial": "CADENA S.A.", "changedFields": ["tasaDescuento", "email"] }
    ],
    "errors": []
  },
  "auditIds": ["uuid1", "uuid2", ...]
}
```

### Lógica de Procesamiento:

```javascript
// Pseudocódigo del flujo de procesamiento

async function procesarCargaMasiva(archivo, usuarioId) {
  // 1. Leer y parsear Excel
  const filas = await leerExcel(archivo);

  // 2. Validar estructura de plantilla
  validarEstructuraPlantilla(filas.headers);

  // 3. Validar cada fila
  const validaciones = [];
  for (const fila of filas.data) {
    const errores = validarFila(fila);
    validaciones.push({ fila, errores });
  }

  // 4. Si hay errores, retornar reporte
  if (validaciones.some(v => v.errores.length > 0)) {
    return generarReporteErrores(validaciones);
  }

  // 5. Procesar cada pagador (upsert)
  const resultados = {
    creados: [],
    actualizados: [],
    errores: []
  };

  for (const fila of filas.data) {
    try {
      const nit = fila.nitPagador;
      const pagadorExistente = await buscarPorNIT(nit);

      if (pagadorExistente) {
        // ACTUALIZAR
        const cambios = detectarCambios(pagadorExistente, fila);
        await actualizarPagador(nit, fila);
        await registrarAuditoria({
          accion: 'ACTUALIZACION_MASIVA',
          pagadorId: pagadorExistente.id,
          usuarioId,
          valoresAnteriores: pagadorExistente,
          valoresNuevos: fila,
          camposCambiados: cambios
        });
        resultados.actualizados.push({ nit, cambios });
      } else {
        // CREAR
        const nuevoPagador = await crearPagador(fila);
        await registrarAuditoria({
          accion: 'CREACION_MASIVA',
          pagadorId: nuevoPagador.id,
          usuarioId,
          valoresNuevos: fila
        });
        resultados.creados.push({ nit });
      }
    } catch (error) {
      resultados.errores.push({
        nit: fila.nitPagador,
        error: error.message
      });
    }
  }

  return resultados;
}
```

### Validaciones:

```javascript
function validarFila(fila) {
  const errores = [];

  // Campos obligatorios
  if (!fila.nitPagador) {
    errores.push({ campo: 'nitPagador', mensaje: 'El NIT es obligatorio' });
  }
  if (!fila.razonSocial) {
    errores.push({ campo: 'razonSocial', mensaje: 'La Razón Social es obligatoria' });
  }
  if (fila.tasaDescuento === null || fila.tasaDescuento === undefined) {
    errores.push({ campo: 'tasaDescuento', mensaje: 'La Tasa de descuento es obligatoria' });
  }
  if (fila.tasaDesembolso === null || fila.tasaDesembolso === undefined) {
    errores.push({ campo: 'tasaDesembolso', mensaje: 'La Tasa de desembolso es obligatoria' });
  }

  // Validaciones de formato
  if (fila.nitPagador && !esNumerico(fila.nitPagador)) {
    errores.push({ campo: 'nitPagador', mensaje: 'El NIT debe ser numérico' });
  }

  // Validaciones de rango
  if (fila.tasaDescuento < 0) {
    errores.push({ campo: 'tasaDescuento', mensaje: 'La tasa debe ser >= 0' });
  }
  if (fila.tasaDesembolso < 0 || fila.tasaDesembolso > 100) {
    errores.push({ campo: 'tasaDesembolso', mensaje: 'La tasa debe estar entre 0 y 100' });
  }

  // Validación de email (si no está vacío)
  if (fila.email && !esEmailValido(fila.email)) {
    errores.push({ campo: 'email', mensaje: 'Formato de email inválido' });
  }

  // Validación de decimales
  if (fila.tasaDescuento && !tieneMaximoDecimales(fila.tasaDescuento, 4)) {
    errores.push({ campo: 'tasaDescuento', mensaje: 'Máximo 4 decimales permitidos' });
  }

  return errores;
}
```

### Registro de Auditoría:

```sql
-- Para creación
INSERT INTO auditoria_pagadores (
  pagador_id,
  usuario_id,
  accion,
  tipo_operacion,
  valores_nuevos,
  fecha_modificacion,
  ip_usuario
) VALUES (
  'uuid-pagador',
  'uuid-usuario',
  'CREACION_MASIVA',
  'INSERT',
  '{"nit": "900123456", "razonSocial": "NUEVA EMPRESA", ...}',
  NOW(),
  '192.168.1.1'
);

-- Para actualización
INSERT INTO auditoria_pagadores (
  pagador_id,
  usuario_id,
  accion,
  tipo_operacion,
  valores_anteriores,
  valores_nuevos,
  campos_modificados,
  fecha_modificacion,
  ip_usuario
) VALUES (
  'uuid-pagador',
  'uuid-usuario',
  'ACTUALIZACION_MASIVA',
  'UPDATE',
  '{"tasaDescuento": 2.0, "email": "test@gmail.com"}',
  '{"tasaDescuento": 2.5, "email": "nuevo@email.com"}',
  '["tasaDescuento", "email"]',
  NOW(),
  '192.168.1.1'
);
```

### Generación de Plantilla Excel:

Librerías sugeridas:
- **Node.js**: `exceljs`, `xlsx`
- **Python**: `openpyxl`, `xlsxwriter`
- **Java**: `Apache POI`
- **C#**: `EPPlus`, `ClosedXML`

Estructura:
- Hoja 1: "Instrucciones" con logo y guía de uso
- Hoja 2: "Datos" con encabezados y formato
- Validaciones de celda (opcional): Listas desplegables, formato numérico

---

## Dependencias

- Sistema de autenticación y autorización por roles
- Base de datos con modelo de pagadores
- Librería para procesamiento de archivos Excel
- Sistema de auditoría implementado
- Componente de carga de archivos (drag & drop)
- Sistema de notificaciones/mensajes al usuario
- Manejo de procesamiento asíncrono para archivos grandes

---

## Riesgos y Mitigaciones

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|--------------|---------|------------|
| Archivo Excel con formato corrupto | Media | Medio | Validar estructura y formato antes de procesar; mensajes de error claros |
| Timeout al procesar archivos muy grandes | Media | Alto | Implementar procesamiento por lotes/chunks; establecer límite de registros; procesamiento asíncrono |
| Errores parciales dejan BD en estado inconsistente | Media | Alto | Usar transacciones de BD; validar TODO antes de procesar NADA; o permitir rollback |
| Usuario carga plantilla antigua o modificada | Alta | Medio | Validar estructura y versión de plantilla; incluir campo de versión en plantilla |
| NITs duplicados dentro del mismo archivo | Media | Medio | Validar duplicados antes de procesar; permitir solo una entrada por NIT |
| Concurrencia: dos usuarios cargan archivos simultáneamente | Baja | Medio | Implementar locks o procesamiento en cola; validar al momento de guardar |
| Pérdida de auditoría por fallo en registro | Baja | Alto | Hacer registro de auditoría parte de la transacción principal |
| Usuario no entiende formatos de tasas (4 decimales) | Alta | Bajo | Instrucciones claras en plantilla; ejemplos; validación y sugerencias |

---

## Casos de Prueba Sugeridos

### Casos de Prueba Funcionales:

1. **Descargar plantilla y verificar estructura**
2. **Cargar archivo con solo creaciones (todos NITs nuevos)**
3. **Cargar archivo con solo actualizaciones (todos NITs existentes)**
4. **Cargar archivo mixto (creaciones y actualizaciones)**
5. **Validar campos obligatorios vacíos**
6. **Validar formatos de datos incorrectos**
7. **Validar rangos de tasas (negativas, >100)**
8. **Validar formato de email**
9. **Procesar archivo con errores parciales**
10. **Detectar NITs duplicados en el archivo**
11. **Cargar archivo muy grande (1000+ registros)**
12. **Validar estructura de plantilla modificada**
13. **Cancelar antes de confirmar**
14. **Confirmar y verificar creaciones/actualizaciones**
15. **Verificar registros de auditoría**
16. **Usuario sin permisos intenta cargar**
17. **Cargar archivo con formato incorrecto (no Excel)**
18. **Verificar actualización de tabla después de carga**

### Casos Límite:

1. Archivo vacío (sin datos, solo encabezados)
2. Archivo con una sola fila
3. Archivo con caracteres especiales en nombres
4. Tasas con exactamente 4 decimales vs más de 4
5. NITs muy largos
6. Emails muy largos
7. Campos de texto con límite de caracteres

---

**Fecha de creación**: 2025-12-09
**Última actualización**: 2025-12-09
