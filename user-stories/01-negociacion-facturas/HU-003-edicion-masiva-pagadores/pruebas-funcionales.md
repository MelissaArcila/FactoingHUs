# Pruebas Funcionales - HU-003: Edición Masiva de Pagadores

**Historia de Usuario**: [HU-003-edicion-masiva-pagadores.md](./HU-003-edicion-masiva-pagadores.md)

**Fecha de creación**: 2025-12-09

**Responsable**: QA Team

---

## Objetivo

Validar que la funcionalidad de carga masiva de pagadores mediante plantilla Excel permita crear nuevos pagadores y actualizar existentes de forma correcta, con validaciones apropiadas, feedback claro y auditoría completa.

---

## Pre-requisitos

- Usuario con permisos de carga masiva autenticado en el sistema
- Base de datos con pagadores existentes para pruebas de actualización
- Plantilla Excel oficial disponible para descargar
- Acceso a la pantalla de gestión de pagadores

---

## Casos de Prueba

### PF-003-01: Descargar plantilla Excel

**Descripción**: Verificar que se puede descargar la plantilla oficial de Excel con la estructura correcta.

**Datos de Entrada**:
- Usuario: `admin@factoring.com` (rol: Administrador)
- Ruta: `/pagadores`

**Pasos**:
1. Iniciar sesión con usuario con permisos
2. Navegar a la pantalla de gestión de pagadores
3. Hacer clic en el botón "Descargar Plantilla" o similar

**Resultado Esperado**:
- ✅ Se descarga un archivo Excel
- ✅ Nombre del archivo: `Plantilla_Pagadores.xlsx` o similar
- ✅ El archivo se puede abrir en Excel/LibreOffice
- ✅ Contiene encabezado con logo e instrucciones:
  - Logo de CDN Factoring
  - Título: "Crear Pagadores"
  - Instrucciones de formato
- ✅ Contiene instrucciones claras:
  - "Ingrese los datos de este formato y guardelo como Libro de Excel (.xlsx)"
  - "Por favor, ingrese las tasas con 4 decimales (ej. 4,0000) y los montos monetarios con 2 decimales (ej. 100000,00)"
- ✅ Contiene fila de encabezados con las columnas (en orden):
  1. NIT Pagador*
  2. Razón Social*
  3. Dirección oficina línea 1
  4. Dirección oficina línea 2
  5. Ciudad de la oficina
  6. Departamento / Estado de la oficina
  7. Código postal oficina
  8. País de la oficina
  9. Email de la oficina
  10. Tasa de descuento por defecto*
  11. Tasa de desembolso por defecto*
- ✅ Los campos obligatorios están marcados con asterisco (*)

---

### PF-003-02: Verificar estructura de plantilla descargada

**Descripción**: Verificar que la plantilla descargada tiene el formato y estructura correctos.

**Datos de Entrada**:
- Plantilla descargada en PF-003-01

**Pasos**:
1. Abrir la plantilla en Excel
2. Verificar formato y estructura

**Resultado Esperado**:
- ✅ El archivo es formato .xlsx válido
- ✅ Contiene al menos una hoja de datos
- ✅ Los encabezados están en negrita o resaltados
- ✅ Las instrucciones son claramente visibles
- ✅ No contiene datos de ejemplo (filas vacías listas para llenar)
- ✅ Opcionalmente: validaciones de celda configuradas (listas desplegables, formato numérico)

---

### PF-003-03: Cargar plantilla con solo creaciones (todos NITs nuevos)

**Descripción**: Verificar que se pueden crear múltiples pagadores nuevos mediante la plantilla.

**Datos de Entrada**:
- Archivo: `Plantilla_Creaciones.xlsx`
- Contenido: 5 pagadores con NITs que no existen en BD

| NIT Pagador | Razón Social | Tasa Descuento | Tasa Desembolso | Email |
|-------------|--------------|----------------|-----------------|-------|
| 999888777 | EMPRESA NUEVA 1 | 2,5000 | 90,0000 | empresa1@test.com |
| 888777666 | EMPRESA NUEVA 2 | 3,0000 | 95,0000 | empresa2@test.com |
| 777666555 | EMPRESA NUEVA 3 | 2,2000 | 92,0000 | empresa3@test.com |
| 666555444 | EMPRESA NUEVA 4 | 4,0000 | 88,0000 | |
| 555444333 | EMPRESA NUEVA 5 | 1,5000 | 100,0000 | empresa5@test.com |

**Pasos**:
1. Llenar la plantilla con los datos anteriores
2. Guardar como archivo Excel
3. En la aplicación, hacer clic en "Carga Masiva"
4. Seleccionar el archivo
5. Hacer clic en "Procesar"
6. Revisar vista previa
7. Confirmar la carga

**Resultado Esperado**:
- ✅ El archivo se carga sin errores de validación
- ✅ La vista previa muestra:
  - Total: 5 registros
  - A crear: 5
  - A actualizar: 0
  - Errores: 0
- ✅ Se muestra lista de los 5 pagadores a crear
- ✅ Al confirmar, se procesan los 5 registros
- ✅ Se muestra resumen final:
  - Creados: 5
  - Actualizados: 0
  - Errores: 0
- ✅ En base de datos:
  - Los 5 pagadores existen con los datos correctos
  - Se crearon 5 registros de auditoría con acción "CREACION_MASIVA"
- ✅ Los pagadores aparecen en la tabla de gestión

---

### PF-003-04: Cargar plantilla con solo actualizaciones (todos NITs existentes)

**Descripción**: Verificar que se pueden actualizar múltiples pagadores existentes mediante la plantilla.

**Datos de Entrada**:
- Archivo: `Plantilla_Actualizaciones.xlsx`
- Contenido: 3 pagadores existentes en BD con datos modificados

| NIT Pagador | Razón Social | Tasa Descuento | Tasa Desembolso | Email |
|-------------|--------------|----------------|-----------------|-------|
| 890930534 | CADENA S.A. | 2,5000 | 95,0000 | nuevo@cadena.com |
| 901453011 | PRESIZA S.A.S | 3,0000 | 92,0000 | contacto@presiza.com |
| 800114437 | ENTREGA DE CARGA S.A. | 2,8000 | 90,0000 | info@entregacarga.com |

**Valores actuales en BD**:
- CADENA S.A.: tasa descuento = 2.0, email = test@gmail.com
- PRESIZA S.A.S: tasa descuento = 2.0, email = test@gmail.com
- ENTREGA DE CARGA S.A.: tasa descuento = 2.0, email = email@oficina.com

**Pasos**:
1. Llenar la plantilla con los datos de pagadores existentes (con cambios)
2. Cargar el archivo
3. Revisar vista previa
4. Confirmar la actualización

**Resultado Esperado**:
- ✅ La vista previa muestra:
  - Total: 3 registros
  - A crear: 0
  - A actualizar: 3
  - Errores: 0
- ✅ Se muestra detalle de cambios para cada pagador:
  - CADENA S.A. (890930534):
    * Tasa descuento: 2.0 → 2.5
    * Tasa desembolso: 90.0 → 95.0
    * Email: test@gmail.com → nuevo@cadena.com
- ✅ Al confirmar, se actualizan los 3 pagadores
- ✅ Resumen final:
  - Creados: 0
  - Actualizados: 3
  - Errores: 0
- ✅ En base de datos:
  - Los datos se actualizaron correctamente
  - Se crearon 3 registros de auditoría con:
    * Acción: "ACTUALIZACION_MASIVA"
    * Valores anteriores (before)
    * Valores nuevos (after)
    * Campos modificados
- ✅ La tabla de pagadores muestra los valores actualizados

---

### PF-003-05: Cargar plantilla mixta (creaciones y actualizaciones)

**Descripción**: Verificar que se pueden procesar simultáneamente creaciones y actualizaciones en un solo archivo.

**Datos de Entrada**:
- Archivo: `Plantilla_Mixta.xlsx`
- Contenido: 2 pagadores nuevos + 2 existentes

**NITs nuevos**:
- 444333222: NUEVA COMPAÑIA S.A.
- 333222111: OTRO PAGADOR LTDA

**NITs existentes**:
- 890930534: CADENA S.A. (con cambios)
- 901453011: PRESIZA S.A.S (con cambios)

**Pasos**:
1. Llenar plantilla con mezcla de pagadores
2. Cargar archivo
3. Revisar vista previa
4. Confirmar

**Resultado Esperado**:
- ✅ Vista previa muestra:
  - Total: 4
  - A crear: 2
  - A actualizar: 2
  - Errores: 0
- ✅ Detalla claramente cuáles se crearán y cuáles se actualizarán
- ✅ Al confirmar:
  - Se crean los 2 nuevos pagadores
  - Se actualizan los 2 existentes
  - Se crean 4 registros de auditoría (2 de creación + 2 de actualización)
- ✅ Resumen final:
  - Creados: 2
  - Actualizados: 2
  - Errores: 0
- ✅ NO hay errores de "pagador ya existe"

---

### PF-003-06: Validar campo obligatorio vacío - NIT Pagador

**Descripción**: Verificar que se detecta error cuando el NIT está vacío.

**Datos de Entrada**:
- Archivo con una fila con NIT vacío:

| NIT Pagador | Razón Social | Tasa Descuento | Tasa Desembolso |
|-------------|--------------|----------------|-----------------|
| [vacío] | EMPRESA SIN NIT | 2,0000 | 90,0000 |

**Pasos**:
1. Llenar plantilla con NIT vacío
2. Cargar archivo

**Resultado Esperado**:
- ✅ El sistema detecta el error
- ✅ Muestra modal/mensaje de errores de validación
- ✅ Indica:
  - Fila: 2 (o el número de fila correspondiente)
  - Campo: "NIT Pagador"
  - Error: "El NIT es obligatorio" o "Este campo es requerido"
- ✅ NO procesa el archivo
- ✅ NO se abre la vista previa de confirmación
- ✅ Permite descargar reporte de errores
- ✅ NO se crea nada en BD

---

### PF-003-07: Validar campo obligatorio vacío - Razón Social

**Descripción**: Verificar que se detecta error cuando la Razón Social está vacía.

**Datos de Entrada**:
- Archivo con Razón Social vacía:

| NIT Pagador | Razón Social | Tasa Descuento | Tasa Desembolso |
|-------------|--------------|----------------|-----------------|
| 123456789 | [vacío] | 2,0000 | 90,0000 |

**Pasos**:
1. Llenar plantilla con Razón Social vacía
2. Cargar archivo

**Resultado Esperado**:
- ✅ Error detectado: "La Razón Social es obligatoria"
- ✅ Fila identificada correctamente
- ✅ No se procesa el archivo

---

### PF-003-08: Validar campo obligatorio vacío - Tasa de descuento

**Descripción**: Verificar que se detecta error cuando la Tasa de descuento está vacía.

**Datos de Entrada**:
- Archivo con Tasa descuento vacía:

| NIT Pagador | Razón Social | Tasa Descuento | Tasa Desembolso |
|-------------|--------------|----------------|-----------------|
| 123456789 | EMPRESA TEST | [vacío] | 90,0000 |

**Pasos**:
1. Cargar archivo con tasa vacía

**Resultado Esperado**:
- ✅ Error: "La Tasa de descuento es obligatoria"
- ✅ No se procesa

---

### PF-003-09: Validar campo obligatorio vacío - Tasa de desembolso

**Descripción**: Verificar que se detecta error cuando la Tasa de desembolso está vacía.

**Datos de Entrada**:
- Archivo con Tasa desembolso vacía:

| NIT Pagador | Razón Social | Tasa Descuento | Tasa Desembolso |
|-------------|--------------|----------------|-----------------|
| 123456789 | EMPRESA TEST | 2,0000 | [vacío] |

**Pasos**:
1. Cargar archivo

**Resultado Esperado**:
- ✅ Error: "La Tasa de desembolso es obligatoria"
- ✅ No se procesa

---

### PF-003-10: Validar múltiples campos obligatorios vacíos

**Descripción**: Verificar que se detectan todos los errores de campos obligatorios en una misma fila.

**Datos de Entrada**:
- Archivo con múltiples campos vacíos:

| NIT Pagador | Razón Social | Tasa Descuento | Tasa Desembolso |
|-------------|--------------|----------------|-----------------|
| [vacío] | [vacío] | [vacío] | [vacío] |

**Pasos**:
1. Cargar archivo

**Resultado Esperado**:
- ✅ Se muestran 4 errores para la misma fila:
  - "El NIT es obligatorio"
  - "La Razón Social es obligatoria"
  - "La Tasa de descuento es obligatoria"
  - "La Tasa de desembolso es obligatoria"
- ✅ No se procesa

---

### PF-003-11: Validar campos opcionales vacíos son aceptados

**Descripción**: Verificar que los campos opcionales pueden estar vacíos sin generar error.

**Datos de Entrada**:
- Archivo con campos opcionales vacíos:

| NIT | Razón Social | Dirección 1 | Dirección 2 | Ciudad | Depto | CP | País | Email | Tasa Desc | Tasa Desemb |
|-----|--------------|-------------|-------------|--------|-------|-------|------|-------|-----------|-------------|
| 123456 | EMPRESA | [vacío] | [vacío] | [vacío] | [vacío] | [vacío] | [vacío] | [vacío] | 2,0000 | 90,0000 |

**Pasos**:
1. Llenar solo campos obligatorios
2. Dejar campos opcionales vacíos
3. Cargar archivo

**Resultado Esperado**:
- ✅ NO se muestran errores
- ✅ El archivo se procesa correctamente
- ✅ El pagador se crea/actualiza con campos opcionales en null/vacío

---

### PF-003-12: Validar formato de NIT (debe ser numérico)

**Descripción**: Verificar que el NIT debe ser un valor numérico.

**Datos de Entrada**:
- NITs con formato inválido:
  - `ABC123` (letras)
  - `123-456-789` (con guiones)
  - `12.34.56` (con puntos)

**Pasos**:
1. Cargar archivo con NITs no numéricos

**Resultado Esperado**:
- ✅ Error: "El NIT debe ser numérico" o "Formato de NIT inválido"
- ✅ Se identifican todas las filas con error
- ✅ No se procesa

---

### PF-003-13: Validar formato de tasa - Valor negativo

**Descripción**: Verificar que no se aceptan tasas negativas.

**Datos de Entrada**:
- Tasa de descuento: `-2,0000`
- Tasa de desembolso: `-90,0000`

**Pasos**:
1. Cargar archivo con tasas negativas

**Resultado Esperado**:
- ✅ Error: "La tasa debe ser mayor o igual a 0"
- ✅ Se identifican ambos campos con error
- ✅ No se procesa

---

### PF-003-14: Validar formato de tasa - Desembolso mayor a 100

**Descripción**: Verificar que la tasa de desembolso no puede ser mayor a 100.

**Datos de Entrada**:
- Tasa de desembolso: `150,0000`

**Pasos**:
1. Cargar archivo con tasa > 100

**Resultado Esperado**:
- ✅ Error: "La tasa de desembolso debe estar entre 0 y 100"
- ✅ No se procesa

---

### PF-003-15: Validar formato de tasas - 4 decimales

**Descripción**: Verificar que las tasas deben tener 4 decimales según las instrucciones.

**Datos de Entrada**:
- Tasas con diferentes decimales:
  - `2,5` (1 decimal)
  - `2,50` (2 decimales)
  - `2,500` (3 decimales)
  - `2,5000` (4 decimales - correcto)
  - `2,50000` (5 decimales - excede)

**Pasos**:
1. Cargar archivo con tasas con distinta cantidad de decimales

**Resultado Esperado**:
- ✅ Opción A: El sistema acepta cualquier cantidad de decimales y los ajusta a 4
- ✅ Opción B: El sistema muestra advertencia si no tiene exactamente 4 decimales
- ✅ Los valores se guardan con 4 decimales en BD
- ✅ No se pierden datos por redondeo

---

### PF-003-16: Validar formato de email inválido

**Descripción**: Verificar que el formato de email se valida correctamente.

**Datos de Entrada**:
- Emails inválidos:
  - `correo-sin-arroba.com`
  - `@dominio.com`
  - `usuario@`
  - `usuario dominio.com` (con espacio)

**Pasos**:
1. Cargar archivo con emails inválidos

**Resultado Esperado**:
- ✅ Error: "Formato de email inválido" para cada caso
- ✅ Se identifican todas las filas con error
- ✅ No se procesa

---

### PF-003-17: Validar formato de email válido

**Descripción**: Verificar que se aceptan emails con formato válido.

**Datos de Entrada**:
- Emails válidos:
  - `usuario@dominio.com`
  - `nombre.apellido@empresa.com.co`
  - `usuario+tag@dominio.org`

**Pasos**:
1. Cargar archivo con emails válidos

**Resultado Esperado**:
- ✅ No se muestran errores de email
- ✅ El archivo se procesa correctamente

---

### PF-003-18: Detectar NITs duplicados dentro del archivo

**Descripción**: Verificar que se detectan NITs duplicados en el mismo archivo.

**Datos de Entrada**:
- Archivo con NITs duplicados:
  - Fila 2: NIT 123456789
  - Fila 5: NIT 123456789
  - Fila 8: NIT 123456789

**Pasos**:
1. Cargar archivo con NITs repetidos

**Resultado Esperado**:
- ✅ Error: "El NIT 123456789 aparece en las filas 2, 5, 8" o similar
- ✅ Se identifican todos los duplicados
- ✅ Opción A: No se procesa hasta corregir
- ✅ Opción B: Se procesa solo la primera ocurrencia con advertencia
- ✅ (Definir comportamiento esperado según requisito de negocio)

---

### PF-003-19: Validar estructura de plantilla modificada

**Descripción**: Verificar que se rechaza un archivo con estructura diferente a la plantilla oficial.

**Datos de Entrada**:
- Archivo con:
  - Columnas con nombres diferentes
  - Columnas faltantes
  - Columnas en orden diferente

**Pasos**:
1. Modificar encabezados de la plantilla
2. Cargar archivo modificado

**Resultado Esperado**:
- ✅ Error: "La plantilla ha sido modificada. Por favor, descargue la plantilla oficial"
- ✅ Se listan las columnas faltantes o incorrectas
- ✅ No se procesa el archivo
- ✅ Se sugiere descargar nueva plantilla

---

### PF-003-20: Cargar archivo con formato incorrecto (no Excel)

**Descripción**: Verificar que solo se aceptan archivos Excel.

**Datos de Entrada**:
- Archivos de prueba:
  - `datos.csv`
  - `datos.txt`
  - `datos.pdf`
  - `imagen.jpg`

**Pasos**:
1. Intentar cargar archivo no Excel

**Resultado Esperado**:
- ✅ Error: "El archivo debe ser formato Excel (.xlsx o .xls)"
- ✅ No se procesa el archivo
- ✅ El selector de archivos solo permite .xlsx/.xls (opcionalmente)

---

### PF-003-21: Cargar archivo vacío (sin datos)

**Descripción**: Verificar el comportamiento con un archivo sin datos.

**Datos de Entrada**:
- Archivo Excel con:
  - Encabezados correctos
  - Cero filas de datos

**Pasos**:
1. Cargar archivo sin datos

**Resultado Esperado**:
- ✅ Mensaje: "El archivo no contiene datos" o similar
- ✅ No se procesa
- ✅ No se abre vista previa

---

### PF-003-22: Cargar archivo muy grande (límite de registros)

**Descripción**: Verificar el manejo de archivos que exceden el límite de registros.

**Datos de Entrada**:
- Archivo con 1500 registros (asumiendo límite de 1000)

**Pasos**:
1. Crear archivo con muchos registros
2. Intentar cargar

**Resultado Esperado**:
- ✅ Advertencia: "El archivo excede el límite de 1000 registros"
- ✅ Sugerencia: "Por favor, divida la carga en múltiples archivos"
- ✅ No se procesa
- ✅ O se procesa con advertencia de que puede tardar mucho tiempo

---

### PF-003-23: Procesar archivo con barra de progreso

**Descripción**: Verificar que se muestra progreso durante el procesamiento.

**Datos de Entrada**:
- Archivo con 100 registros

**Pasos**:
1. Cargar archivo
2. Confirmar procesamiento
3. Observar indicadores de progreso

**Resultado Esperado**:
- ✅ Se muestra barra de progreso o spinner
- ✅ Se muestra porcentaje completado (0% - 100%)
- ✅ Se muestra texto: "Procesando X de Y registros..."
- ✅ La interfaz no se congela
- ✅ Se puede ver el progreso en tiempo real

---

### PF-003-24: Cancelar carga antes de confirmar

**Descripción**: Verificar que se puede cancelar antes de procesar.

**Datos de Entrada**:
- Archivo válido con 10 registros

**Pasos**:
1. Cargar archivo
2. Ver vista previa
3. Hacer clic en "Cancelar"

**Resultado Esperado**:
- ✅ La modal/vista previa se cierra
- ✅ NO se procesan los registros
- ✅ NO se crean/actualizan pagadores en BD
- ✅ NO se registra nada en auditoría
- ✅ Se regresa a la pantalla de gestión de pagadores

---

### PF-003-25: Confirmar y verificar resumen final

**Descripción**: Verificar que el resumen final muestra información correcta y completa.

**Datos de Entrada**:
- Archivo con:
  - 3 creaciones
  - 2 actualizaciones
  - 0 errores

**Pasos**:
1. Cargar archivo
2. Confirmar procesamiento
3. Esperar a que termine
4. Revisar resumen final

**Resultado Esperado**:
- ✅ Modal de resumen con título: "Carga Masiva Completada" o similar
- ✅ Ícono de éxito (✓)
- ✅ Estadísticas:
  - Total procesados: 5
  - Creados: 3
  - Actualizados: 2
  - Errores: 0
- ✅ Detalle de creados (lista con NIT y Razón Social)
- ✅ Detalle de actualizados (lista con NIT, Razón Social y campos modificados)
- ✅ Opción para descargar reporte completo
- ✅ Botón "Cerrar" o "Aceptar"

---

### PF-003-26: Verificar registros de auditoría para creaciones

**Descripción**: Verificar que se crean registros de auditoría correctos para pagadores nuevos.

**Datos de Entrada**:
- 1 pagador nuevo creado vía carga masiva:
  - NIT: 999888777
  - Razón Social: EMPRESA NUEVA
  - Usuario: admin@factoring.com

**Pasos**:
1. Crear pagador vía carga masiva
2. Consultar tabla de auditoría en BD

**Resultado Esperado**:
- ✅ Existe registro en tabla `auditoria_pagadores`
- ✅ Campos del registro:
  - pagador_id: ID del nuevo pagador
  - usuario_id: ID del usuario que cargó
  - accion: "CREACION_MASIVA"
  - tipo_operacion: "INSERT"
  - valores_anteriores: null
  - valores_nuevos: JSON con todos los datos del pagador
  - fecha_modificacion: timestamp de la operación
  - ip_usuario: IP del usuario (opcional)

---

### PF-003-27: Verificar registros de auditoría para actualizaciones

**Descripción**: Verificar que se crean registros de auditoría correctos para pagadores actualizados.

**Datos de Entrada**:
- 1 pagador actualizado vía carga masiva:
  - NIT: 890930534 (CADENA S.A.)
  - Cambios: Tasa descuento (2.0 → 2.5), Email

**Pasos**:
1. Actualizar pagador vía carga masiva
2. Consultar auditoría

**Resultado Esperado**:
- ✅ Existe registro de auditoría
- ✅ Campos:
  - accion: "ACTUALIZACION_MASIVA"
  - tipo_operacion: "UPDATE"
  - valores_anteriores: JSON con valores antes del cambio
  - valores_nuevos: JSON con valores después del cambio
  - campos_modificados: Array con ["tasaDescuento", "email"]
  - usuario_id, fecha, etc.

---

### PF-003-28: Actualización de tabla después de carga exitosa

**Descripción**: Verificar que la tabla de pagadores se actualiza automáticamente.

**Datos de Entrada**:
- Carga masiva completada con:
  - 2 pagadores creados
  - 1 pagador actualizado

**Pasos**:
1. Completar carga masiva exitosa
2. Cerrar modal de resumen
3. Observar tabla de pagadores

**Resultado Esperado**:
- ✅ La tabla se actualiza automáticamente (sin F5)
- ✅ Los 2 pagadores nuevos aparecen en la tabla
- ✅ El pagador actualizado muestra los nuevos valores
- ✅ Los cambios son visibles inmediatamente
- ✅ No hay delay o inconsistencia en los datos

---

### PF-003-29: Usuario sin permisos no puede acceder

**Descripción**: Verificar que usuarios sin permisos no pueden realizar carga masiva.

**Datos de Entrada**:
- Usuario: `consultor@factoring.com` (sin permisos de carga masiva)

**Pasos**:
1. Iniciar sesión con usuario sin permisos
2. Navegar a gestión de pagadores
3. Buscar opción de carga masiva

**Resultado Esperado**:
- ✅ El botón "Carga Masiva" no está visible, O
- ✅ El botón está deshabilitado (grayed out), O
- ✅ Al intentar acceder muestra: "No tiene permisos para realizar carga masiva"
- ✅ No puede acceder a la funcionalidad

---

### PF-003-30: Manejo de errores parciales - Opción A (Validación previa)

**Descripción**: Si se implementa validación previa total, verificar que NADA se procesa si hay errores.

**Datos de Entrada**:
- Archivo con:
  - 5 filas válidas
  - 2 filas con errores

**Pasos**:
1. Cargar archivo con errores parciales

**Resultado Esperado**:
- ✅ El sistema valida TODAS las filas primero
- ✅ Muestra reporte con las 2 filas con error
- ✅ NO procesa NINGUNA fila (ni las 5 válidas)
- ✅ Mensaje: "Corrija los errores antes de procesar"
- ✅ Permite descargar reporte de errores
- ✅ NO se crea/actualiza nada en BD

---

### PF-003-31: Manejo de errores parciales - Opción B (Procesamiento parcial)

**Descripción**: Si se implementa procesamiento parcial, verificar que solo se procesan las filas válidas.

**Datos de Entrada**:
- Archivo con:
  - 5 filas válidas
  - 2 filas con errores

**Pasos**:
1. Cargar archivo

**Resultado Esperado**:
- ✅ Vista previa muestra:
  - Válidas: 5
  - Con errores: 2
- ✅ Detalla cuáles filas tienen errores
- ✅ Opción para procesar solo las válidas
- ✅ Al confirmar:
  - Se procesan las 5 válidas
  - Se omiten las 2 con errores
- ✅ Resumen final:
  - Procesados: 5
  - Errores: 2
- ✅ Lista de errores con filas y detalles
- ✅ Permite descargar reporte de errores para corregir

---

### PF-003-32: Concurrencia - Dos usuarios cargan simultáneamente

**Descripción**: Verificar el comportamiento cuando dos usuarios cargan archivos al mismo tiempo.

**Datos de Entrada**:
- Usuario A: Carga archivo con 10 registros
- Usuario B: Carga archivo con 5 registros (algunos NITs coinciden con A)
- Tiempo: Simultáneo o casi simultáneo

**Pasos**:
1. Usuario A inicia carga
2. Usuario B inicia carga (antes de que A termine)
3. Ambos confirman

**Resultado Esperado**:
- ✅ Ambas cargas se procesan sin conflictos
- ✅ Si hay NITs coincidentes, el último en procesar gana (o se implementa lock)
- ✅ Los registros de auditoría muestran correctamente quién hizo cada cambio
- ✅ No hay corrupción de datos
- ✅ No hay errores de transacción

---

### PF-003-33: Caracteres especiales en nombres

**Descripción**: Verificar que se manejan correctamente caracteres especiales.

**Datos de Entrada**:
- Razón Social con caracteres especiales:
  - `EMPRESA & CIA S.A.S.`
  - `CORPORACIÓN ÑOÑO LTDA`
  - `IMPORTADORA D'ACOSTA`
  - `SOCIEDAD (EN LIQUIDACIÓN)`

**Pasos**:
1. Cargar archivo con caracteres especiales

**Resultado Esperado**:
- ✅ Los caracteres especiales se aceptan
- ✅ Se guardan correctamente en BD
- ✅ Se muestran correctamente en la tabla
- ✅ No hay caracteres corruptos o mal codificados

---

### PF-003-34: Valores límite - Tasas en 0

**Descripción**: Verificar que se aceptan tasas en 0 (valor mínimo válido).

**Datos de Entrada**:
- Tasa descuento: `0,0000`
- Tasa desembolso: `0,0000`

**Pasos**:
1. Cargar archivo con tasas en 0

**Resultado Esperado**:
- ✅ Las tasas en 0 se aceptan
- ✅ Se procesan sin error

---

### PF-003-35: Valores límite - Tasa desembolso en 100

**Descripción**: Verificar que se acepta 100 como valor máximo para tasa de desembolso.

**Datos de Entrada**:
- Tasa desembolso: `100,0000`

**Pasos**:
1. Cargar archivo

**Resultado Esperado**:
- ✅ Tasa 100 se acepta
- ✅ Se procesa correctamente

---

### PF-003-36: Descargar reporte de resultados

**Descripción**: Verificar que se puede descargar reporte detallado de los resultados.

**Datos de Entrada**:
- Carga completada con resultados mixtos

**Pasos**:
1. Completar carga masiva
2. En resumen final, hacer clic en "Descargar reporte"

**Resultado Esperado**:
- ✅ Se descarga archivo (Excel o PDF)
- ✅ El archivo contiene:
  - Fecha y hora de la carga
  - Usuario que realizó la carga
  - Estadísticas (total, creados, actualizados, errores)
  - Detalle de cada operación
  - Lista de errores (si los hubo)
- ✅ El archivo se puede abrir y leer correctamente

---

### PF-003-37: Descargar reporte de errores

**Descripción**: Verificar que se puede descargar reporte de errores para corregir.

**Datos de Entrada**:
- Archivo con 3 filas con errores

**Pasos**:
1. Cargar archivo con errores
2. En modal de errores, hacer clic en "Descargar reporte de errores"

**Resultado Esperado**:
- ✅ Se descarga archivo Excel o CSV
- ✅ Contiene:
  - Fila con error
  - NIT (si existe)
  - Razón Social (si existe)
  - Campo con error
  - Valor ingresado
  - Mensaje de error
  - Valor esperado/correcto
- ✅ Permite identificar rápidamente qué corregir

---

### PF-003-38: Timeout con archivo muy grande

**Descripción**: Verificar que archivos grandes no causan timeout.

**Datos de Entrada**:
- Archivo con 500 registros (grande pero dentro del límite)

**Pasos**:
1. Cargar archivo grande
2. Confirmar procesamiento
3. Monitorear tiempo de procesamiento

**Resultado Esperado**:
- ✅ El archivo se procesa completamente
- ✅ No hay timeout
- ✅ Se muestra progreso durante todo el procesamiento
- ✅ El tiempo de procesamiento es razonable (< 2 minutos)
- ✅ Opcionalmente: Se procesa por lotes/chunks

---

### PF-003-39: Responsive - Carga masiva en móvil

**Descripción**: Verificar funcionalidad en dispositivos móviles.

**Datos de Entrada**:
- Dispositivo: iPhone/Android
- Archivo pequeño de prueba

**Pasos**:
1. Acceder desde móvil
2. Intentar descargar plantilla
3. Intentar cargar archivo

**Resultado Esperado**:
- ✅ El botón de carga masiva es accesible
- ✅ Se puede descargar plantilla
- ✅ Se puede seleccionar archivo desde el dispositivo
- ✅ Las modales se adaptan al tamaño de pantalla
- ✅ La funcionalidad es usable en móvil
- ✅ Nota: Puede ser más práctico desde desktop, pero debe funcionar

---

### PF-003-40: Verificar que datos opcionales se guardan correctamente

**Descripción**: Verificar que los campos opcionales con datos se guardan en BD.

**Datos de Entrada**:
- Pagador con todos los campos opcionales llenos:
  - Dirección línea 1: "Calle 123 #45-67"
  - Dirección línea 2: "Edificio Torre Norte, Piso 5"
  - Ciudad: "Medellín"
  - Departamento: "Antioquia"
  - Código postal: "050001"
  - País: "Colombia"
  - Email: "contacto@empresa.com"

**Pasos**:
1. Cargar pagador con todos los datos
2. Procesar carga
3. Verificar en BD

**Resultado Esperado**:
- ✅ Todos los campos opcionales se guardan correctamente
- ✅ Los valores coinciden con lo ingresado en la plantilla
- ✅ No hay pérdida de datos

---

## Matriz de Cobertura

| Escenario de Aceptación | Casos de Prueba Relacionados | Estado |
|--------------------------|------------------------------|--------|
| 1. Descargar plantilla | PF-003-01, PF-003-02 | ☐ |
| 2. Cargar archivo | PF-003-03 a PF-003-05, PF-003-20, PF-003-21 | ☐ |
| 3. Validación campos obligatorios | PF-003-06 a PF-003-10 | ☐ |
| 4. Validación formato datos | PF-003-12 a PF-003-17 | ☐ |
| 5. Crear nuevo pagador | PF-003-03, PF-003-05, PF-003-26 | ☐ |
| 6. Actualizar pagador existente | PF-003-04, PF-003-05, PF-003-27 | ☐ |
| 7. Mezcla creaciones/actualizaciones | PF-003-05 | ☐ |
| 8. Resumen de resultados | PF-003-25, PF-003-36 | ☐ |
| 9. Manejo errores parciales | PF-003-30, PF-003-31 | ☐ |
| 10. Duplicados en archivo | PF-003-18 | ☐ |
| 11. Validación permisos | PF-003-29 | ☐ |
| 12. Archivo muy grande | PF-003-22, PF-003-38 | ☐ |
| 13. Cancelar carga | PF-003-24 | ☐ |
| 14. Confirmar carga | PF-003-25, PF-003-28 | ☐ |
| 15. Validación estructura plantilla | PF-003-19 | ☐ |
| 16. Actualización tabla | PF-003-28 | ☐ |
| Auditoría | PF-003-26, PF-003-27 | ☐ |
| Concurrencia | PF-003-32 | ☐ |
| Valores límite | PF-003-34, PF-003-35 | ☐ |
| Caracteres especiales | PF-003-33 | ☐ |

---

## Datos de Prueba Sugeridos

### Archivo de Prueba 1: Solo Creaciones
```
NIT      | Razón Social        | Tasa Desc | Tasa Desemb | Email
999888777| EMPRESA NUEVA 1     | 2,5000    | 90,0000     | empresa1@test.com
888777666| EMPRESA NUEVA 2     | 3,0000    | 95,0000     | empresa2@test.com
777666555| EMPRESA NUEVA 3     | 2,2000    | 92,0000     |
```

### Archivo de Prueba 2: Solo Actualizaciones
```
NIT      | Razón Social        | Tasa Desc | Tasa Desemb | Email
890930534| CADENA S.A.         | 2,5000    | 95,0000     | nuevo@cadena.com
901453011| PRESIZA S.A.S       | 3,0000    | 92,0000     | contacto@presiza.com
```

### Archivo de Prueba 3: Mixto
```
NIT      | Razón Social        | Tasa Desc | Tasa Desemb | Email
999888777| EMPRESA NUEVA       | 2,5000    | 90,0000     | nueva@test.com
890930534| CADENA S.A.         | 2,5000    | 95,0000     | nuevo@cadena.com
```

### Archivo de Prueba 4: Con Errores
```
NIT      | Razón Social        | Tasa Desc | Tasa Desemb | Email
[vacío]  | EMPRESA SIN NIT     | 2,0000    | 90,0000     | error1@test.com
123456   | [vacío]             | 2,0000    | 90,0000     | error2@test.com
789012   | TASA NEGATIVA       | -2,0000   | 90,0000     | error3@test.com
345678   | TASA ALTA           | 2,0000    | 150,0000    | error4@test.com
567890   | EMAIL INVALIDO      | 2,0000    | 90,0000     | correo-invalido
```

---

## Criterios de Aceptación de Pruebas

- ✅ Todos los casos de prueba deben pasar exitosamente
- ✅ La plantilla se descarga con formato correcto
- ✅ El sistema maneja correctamente creaciones y actualizaciones (upsert)
- ✅ Todas las validaciones funcionan correctamente
- ✅ Los registros de auditoría se crean para todas las operaciones
- ✅ El manejo de errores es robusto
- ✅ No hay errores en consola del navegador
- ✅ La cobertura de código debe ser >= 80%
- ✅ Las pruebas deben pasar en: Chrome, Firefox, Safari, Edge
- ✅ Se probó con archivos de diferentes tamaños (1, 10, 100, 1000 registros)
- ✅ No hay pérdida de datos durante el procesamiento
- ✅ El tiempo de procesamiento es aceptable

---

## Notas de Ejecución

**Ambiente de Pruebas**: [Especificar URL del ambiente QA]

**Base de Datos**: [Especificar instancia de BD de pruebas]

**Datos de Prueba Creados**:
- Pagadores existentes en BD para pruebas de actualización
- Pagadores nuevos para pruebas de creación
- Archivos Excel de prueba preparados

**Herramientas**:
- Excel / LibreOffice Calc para crear plantillas de prueba
- Herramientas de desarrollo del navegador para monitorear requests
- Acceso a BD para verificar datos y auditoría

**Fecha de Ejecución**: _______________________

**Ejecutado por**: _______________________

**Observaciones**:
_________________________________________________________________
_________________________________________________________________
_________________________________________________________________

---

**Última actualización**: 2025-12-09
