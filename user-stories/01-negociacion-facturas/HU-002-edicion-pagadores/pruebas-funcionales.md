# Pruebas Funcionales - HU-002: Edición de Información de Pagadores

**Historia de Usuario**: [HU-002-edicion-pagadores.md](./HU-002-edicion-pagadores.md)

**Fecha de creación**: 2025-12-09

**Responsable**: QA Team

---

## Objetivo

Validar que la funcionalidad de edición de pagadores permita actualizar correctamente la configuración de tasas y correo de contacto mediante una modal con validaciones, confirmación y feedback apropiado al usuario.

---

## Pre-requisitos

- Usuario con permisos de edición autenticado en el sistema
- Base de datos con pagadores registrados y datos existentes
- Acceso a la pantalla de gestión de pagadores

---

## Casos de Prueba

### PF-002-01: Visualización de columna "Acciones" con ícono de editar

**Descripción**: Verificar que se muestra la nueva columna "Acciones" en la tabla de pagadores con el ícono de editar para cada registro.

**Datos de Entrada**:
- Usuario: `admin@factoring.com` (rol: Administrador)
- Ruta: `/pagadores`

**Pasos**:
1. Iniciar sesión con usuario con permisos de edición
2. Navegar a la pantalla de gestión de pagadores

**Resultado Esperado**:
- ✅ Se muestra la tabla de pagadores con todas las columnas existentes
- ✅ Se muestra una nueva columna "Acciones"
- ✅ Cada fila tiene un ícono de editar (✏️ lápiz/pencil)
- ✅ El ícono es clickeable y tiene cursor pointer
- ✅ El ícono está visible para todos los pagadores en la tabla

---

### PF-002-02: Abrir modal de edición con datos pre-cargados

**Descripción**: Verificar que al hacer clic en el ícono de editar se abre la modal con todos los campos correctamente pre-cargados desde la base de datos.

**Datos de Entrada**:
- Pagador seleccionado:
  - Razón Social: `CADENA S.A.`
  - NIT: `890930534`
  - Tasa descuento actual: `2.0`
  - Tasa desembolso actual: `90`
  - Correo actual: `test@gmail.com`

**Pasos**:
1. Localizar el pagador "CADENA S.A." en la tabla
2. Hacer clic en el ícono de editar (✏️)

**Resultado Esperado**:
- ✅ Se abre una modal centrada con título "Editar Pagador"
- ✅ El fondo se oscurece (overlay)
- ✅ La modal tiene botón X de cerrar en la esquina superior derecha
- ✅ **Campos no editables** (solo lectura, griseados o deshabilitados):
  - Razón Social: "CADENA S.A."
  - NIT: "890930534"
- ✅ **Campos editables** con valores pre-cargados:
  - Tasa de descuento por defecto*: `2.0` (con sufijo %)
  - Tasa de desembolso por defecto*: `90` (con sufijo %)
  - Correo de contacto: `test@gmail.com`
- ✅ Los campos requeridos están marcados con asterisco (*)
- ✅ Se muestran dos botones: "Cancelar" y "Guardar"

---

### PF-002-03: Pre-carga de pagador con campo opcional vacío

**Descripción**: Verificar que si un pagador no tiene correo de contacto registrado, el campo se muestra vacío pero editable.

**Datos de Entrada**:
- Pagador seleccionado:
  - Razón Social: `testakshavw`
  - NIT: `098723451`
  - Tasa descuento: `0`
  - Tasa desembolso: `0`
  - Correo: `null` o vacío en BD

**Pasos**:
1. Hacer clic en editar para el pagador "testakshavw"

**Resultado Esperado**:
- ✅ La modal se abre correctamente
- ✅ Los campos obligatorios tienen sus valores: `0` y `0`
- ✅ El campo "Correo de contacto" está vacío (sin valor)
- ✅ El campo está habilitado y se puede escribir en él
- ✅ No hay mensaje de error por campo vacío (es opcional)

---

### PF-002-04: Validación de campo requerido vacío - Tasa de descuento

**Descripción**: Verificar que no se permite guardar si el campo "Tasa de descuento por defecto" está vacío.

**Datos de Entrada**:
- Pagador: `CADENA S.A.`
- Acción: Borrar el valor del campo "Tasa de descuento por defecto"
- Valores:
  - Tasa descuento: `[vacío]`
  - Tasa desembolso: `90`
  - Correo: `test@gmail.com`

**Pasos**:
1. Abrir modal de edición
2. Borrar completamente el valor del campo "Tasa de descuento por defecto"
3. Intentar hacer clic en "Guardar"

**Resultado Esperado**:
- ✅ El botón "Guardar" no procede con el guardado
- ✅ Se muestra mensaje de error debajo del campo: "Este campo es requerido"
- ✅ El campo se resalta visualmente (borde rojo u otro indicador)
- ✅ La modal permanece abierta
- ✅ No se abre la modal de confirmación
- ✅ No se realizan cambios en la base de datos

---

### PF-002-05: Validación de campo requerido vacío - Tasa de desembolso

**Descripción**: Verificar que no se permite guardar si el campo "Tasa de desembolso por defecto" está vacío.

**Datos de Entrada**:
- Pagador: `PRESIZA S.A.S`
- Acción: Borrar el valor del campo "Tasa de desembolso por defecto"
- Valores:
  - Tasa descuento: `2.2`
  - Tasa desembolso: `[vacío]`
  - Correo: `test@gmail.com`

**Pasos**:
1. Abrir modal de edición
2. Borrar completamente el valor del campo "Tasa de desembolso por defecto"
3. Intentar hacer clic en "Guardar"

**Resultado Esperado**:
- ✅ El botón "Guardar" no procede
- ✅ Se muestra mensaje de error: "Este campo es requerido"
- ✅ El campo se resalta con borde rojo
- ✅ No se abre modal de confirmación
- ✅ No se guardan cambios

---

### PF-002-06: Validación de múltiples campos requeridos vacíos

**Descripción**: Verificar que se muestran errores para todos los campos requeridos que estén vacíos simultáneamente.

**Datos de Entrada**:
- Pagador: `ENTREGA DE CARGA S.A.`
- Acción: Borrar ambos campos requeridos
- Valores:
  - Tasa descuento: `[vacío]`
  - Tasa desembolso: `[vacío]`
  - Correo: `email@oficina.com`

**Pasos**:
1. Abrir modal de edición
2. Borrar ambos campos obligatorios
3. Intentar guardar

**Resultado Esperado**:
- ✅ Se muestran mensajes de error en ambos campos:
  - "Este campo es requerido" bajo Tasa de descuento
  - "Este campo es requerido" bajo Tasa de desembolso
- ✅ Ambos campos se resaltan visualmente
- ✅ No se procede con el guardado

---

### PF-002-07: Validación de tipo de dato - Tasa de descuento con texto

**Descripción**: Verificar que el campo "Tasa de descuento" solo acepta valores numéricos y rechaza texto.

**Datos de Entrada**:
- Pagador: `CADENA S.A.`
- Valores ingresados:
  - Tasa descuento: `abc` (texto)
  - Tasa desembolso: `90`
  - Correo: `test@gmail.com`

**Pasos**:
1. Abrir modal de edición
2. Intentar escribir "abc" en el campo Tasa de descuento
3. Intentar guardar

**Resultado Esperado**:
- ✅ El campo no permite ingresar letras (solo números), O
- ✅ Se muestra error: "Debe ser un valor numérico"
- ✅ No se permite guardar con valor inválido

---

### PF-002-08: Validación de rango - Tasa de descuento negativa

**Descripción**: Verificar que no se permiten valores negativos en la tasa de descuento.

**Datos de Entrada**:
- Pagador: `PRESIZA S.A.S`
- Valores:
  - Tasa descuento: `-5`
  - Tasa desembolso: `90`
  - Correo: `test@gmail.com`

**Pasos**:
1. Abrir modal de edición
2. Ingresar "-5" en Tasa de descuento
3. Intentar guardar

**Resultado Esperado**:
- ✅ Se muestra mensaje de error: "La tasa de descuento debe ser mayor o igual a 0"
- ✅ El campo se marca como inválido
- ✅ No se permite guardar

---

### PF-002-09: Validación de rango - Tasa de desembolso mayor a 100

**Descripción**: Verificar que la tasa de desembolso no puede ser mayor a 100.

**Datos de Entrada**:
- Pagador: `IMPORTLOBEX S.A.S`
- Valores:
  - Tasa descuento: `2.2`
  - Tasa desembolso: `150`
  - Correo: `test@gmail.com`

**Pasos**:
1. Abrir modal de edición
2. Ingresar "150" en Tasa de desembolso
3. Intentar guardar

**Resultado Esperado**:
- ✅ Se muestra error: "La tasa de desembolso debe estar entre 0 y 100"
- ✅ El campo se marca como inválido
- ✅ No se permite guardar

---

### PF-002-10: Validación de valores decimales válidos

**Descripción**: Verificar que se aceptan valores decimales válidos en los campos de tasas.

**Datos de Entrada**:
- Pagador: `CADENA S.A.`
- Valores:
  - Tasa descuento: `2.5`
  - Tasa desembolso: `95.5`
  - Correo: `test@gmail.com`

**Pasos**:
1. Abrir modal de edición
2. Ingresar valores decimales
3. Guardar

**Resultado Esperado**:
- ✅ Los valores decimales se aceptan sin error
- ✅ Se permite proceder con el guardado
- ✅ Los decimales se mantienen correctamente (no se redondean incorrectamente)

---

### PF-002-11: Validación de formato de email inválido

**Descripción**: Verificar que el campo "Correo de contacto" valida el formato de email.

**Datos de Entrada**:
- Pagador: `DINDELCO S.A.S.`
- Valores:
  - Tasa descuento: `2.2`
  - Tasa desembolso: `90`
  - Correo: `correo-invalido` (sin @ ni dominio)

**Pasos**:
1. Abrir modal de edición
2. Ingresar "correo-invalido" en el campo Correo
3. Intentar guardar

**Resultado Esperado**:
- ✅ Se muestra error: "Debe ser un correo electrónico válido"
- ✅ El campo se marca como inválido
- ✅ No se permite guardar

---

### PF-002-12: Validación de email con formato válido

**Descripción**: Verificar que se aceptan emails con formato válido.

**Datos de Entrada**:
- Pagador: `DINDELCO S.A.S.`
- Emails válidos a probar:
  - `usuario@dominio.com`
  - `usuario.nombre@dominio.co`
  - `usuario+tag@dominio.com.co`

**Pasos**:
1. Abrir modal de edición
2. Ingresar cada email válido
3. Verificar validación

**Resultado Esperado**:
- ✅ Todos los formatos válidos son aceptados
- ✅ No se muestran errores
- ✅ Se permite proceder con el guardado

---

### PF-002-13: Campo email opcional puede dejarse vacío

**Descripción**: Verificar que el campo "Correo de contacto" puede guardarse vacío sin error.

**Datos de Entrada**:
- Pagador: `INVERSEGC SAS`
- Valores:
  - Tasa descuento: `0`
  - Tasa desembolso: `90`
  - Correo: `[vacío]`

**Pasos**:
1. Abrir modal de edición
2. Dejar el campo Correo vacío
3. Guardar

**Resultado Esperado**:
- ✅ No se muestra error en el campo Correo
- ✅ Se permite proceder con el guardado
- ✅ La modal de confirmación se abre normalmente

---

### PF-002-14: Cancelar edición sin guardar cambios

**Descripción**: Verificar que al hacer clic en "Cancelar", la modal se cierra sin guardar cambios.

**Datos de Entrada**:
- Pagador: `CADENA S.A.`
- Valores originales en BD:
  - Tasa descuento: `2.0`
  - Tasa desembolso: `90`
- Cambios realizados (no guardados):
  - Tasa descuento: `3.5`
  - Tasa desembolso: `95`

**Pasos**:
1. Abrir modal de edición
2. Modificar los valores
3. Hacer clic en "Cancelar"

**Resultado Esperado**:
- ✅ La modal se cierra inmediatamente
- ✅ No se guardan los cambios en la base de datos
- ✅ Los valores en BD permanecen como estaban: 2.0 y 90
- ✅ La tabla muestra los valores originales
- ✅ No se registra nada en auditoría

---

### PF-002-15: Cerrar modal con botón X sin guardar

**Descripción**: Verificar que al cerrar con la X, se comporta igual que "Cancelar".

**Datos de Entrada**:
- Pagador: `PRESIZA S.A.S`
- Cambios realizados pero no guardados

**Pasos**:
1. Abrir modal de edición
2. Modificar valores
3. Hacer clic en la X

**Resultado Esperado**:
- ✅ La modal se cierra
- ✅ No se guardan cambios
- ✅ Comportamiento idéntico a "Cancelar"

---

### PF-002-16: Modal de confirmación muestra datos correctos

**Descripción**: Verificar que al hacer clic en "Guardar" (con datos válidos), se abre la modal de confirmación mostrando correctamente los datos a actualizar.

**Datos de Entrada**:
- Pagador: `CADENA S.A.`
- NIT: `890930534`
- Cambios a realizar:
  - Tasa descuento: `2.0` → `2.5`
  - Tasa desembolso: `90` → `95`
  - Correo: `test@gmail.com` → `nuevo@email.com`

**Pasos**:
1. Abrir modal de edición
2. Cambiar los valores según los datos de entrada
3. Hacer clic en "Guardar"

**Resultado Esperado**:
- ✅ Se abre una segunda modal de confirmación
- ✅ Título: "Confirmar Cambios"
- ✅ Muestra pregunta: "¿Confirma la actualización de los datos del siguiente pagador?"
- ✅ Sección "Pagador":
  - Razón Social: CADENA S.A.
  - NIT: 890930534
- ✅ Sección "Nuevos valores":
  - Tasa de descuento: 2.5%
  - Tasa de desembolso: 95%
  - Correo: nuevo@email.com
- ✅ Botones: "Cancelar" y "Confirmar"
- ✅ La modal de edición queda en segundo plano o se oculta

---

### PF-002-17: Cancelar desde modal de confirmación

**Descripción**: Verificar que al cancelar desde la modal de confirmación, se regresa a la modal de edición sin guardar.

**Datos de Entrada**:
- Pagador: `PRESIZA S.A.S`
- Modal de confirmación abierta

**Pasos**:
1. Llegar hasta la modal de confirmación (seguir PF-002-16)
2. Hacer clic en "Cancelar" en la modal de confirmación

**Resultado Esperado**:
- ✅ La modal de confirmación se cierra
- ✅ Se regresa a la modal de edición
- ✅ Los datos ingresados se mantienen en la modal de edición
- ✅ El usuario puede seguir editando o cancelar completamente
- ✅ No se guardan cambios en BD

---

### PF-002-18: Confirmar guardado exitoso

**Descripción**: Verificar que al confirmar los cambios, se actualizan correctamente en la base de datos y se muestra feedback al usuario.

**Datos de Entrada**:
- Pagador: `ENTREGA DE CARGA S.A.`
- NIT: `800114437`
- Cambios:
  - Tasa descuento: `2.0` → `2.2`
  - Tasa desembolso: `90` → `92`
  - Correo: `email@oficina.com` → `nuevo@oficina.com`
- Usuario: `admin@factoring.com`
- Fecha/hora: `2025-12-09 14:30:00`

**Pasos**:
1. Realizar edición con los cambios especificados
2. Hacer clic en "Guardar"
3. En modal de confirmación, hacer clic en "Confirmar"

**Resultado Esperado**:
- ✅ Se muestra indicador de carga/procesamiento
- ✅ Ambas modales se cierran automáticamente
- ✅ Se muestra mensaje de éxito: "Datos del pagador actualizados correctamente"
- ✅ La tabla de pagadores se actualiza con los nuevos valores
- ✅ No es necesario recargar la página manualmente
- ✅ En base de datos:
  - Tasa descuento = 2.2
  - Tasa desembolso = 92
  - Correo = nuevo@oficina.com
  - Fecha actualización actualizada
- ✅ Se crea registro de auditoría con:
  - ID del pagador
  - Usuario: admin@factoring.com
  - Fecha: 2025-12-09 14:30:00
  - Valores anteriores: {descuento: 2.0, desembolso: 90, correo: email@oficina.com}
  - Valores nuevos: {descuento: 2.2, desembolso: 92, correo: nuevo@oficina.com}

---

### PF-002-19: Actualización de solo un campo

**Descripción**: Verificar que se puede actualizar solo un campo sin afectar los demás.

**Datos de Entrada**:
- Pagador: `DINDELCO S.A.S.`
- Cambio: Solo correo: `` → `nuevo@dindelco.com`
- Sin cambios en tasas

**Pasos**:
1. Abrir modal de edición
2. Modificar solo el campo Correo
3. Guardar y confirmar

**Resultado Esperado**:
- ✅ Solo el correo se actualiza en BD
- ✅ Las tasas permanecen sin cambios
- ✅ La auditoría solo registra el cambio del correo

---

### PF-002-20: Error al guardar - Error de servidor

**Descripción**: Verificar el manejo de errores cuando el servidor falla al guardar.

**Datos de Entrada**:
- Pagador: `IMPORTLOBEX S.A.S`
- Condición: Simular error 500 del servidor

**Pasos**:
1. Realizar cambios válidos
2. Confirmar en modal de confirmación
3. Simular error en el backend (500 Internal Server Error)

**Resultado Esperado**:
- ✅ Se muestra indicador de carga
- ✅ Se muestra mensaje de error: "No se pudieron guardar los cambios. Por favor, intente nuevamente."
- ✅ La modal de confirmación se cierra
- ✅ Se regresa a la modal de edición con los datos ingresados intactos
- ✅ El usuario puede:
  - Reintentar guardar
  - Modificar los datos
  - Cancelar la operación
- ✅ No se modifica nada en BD
- ✅ No se registra en auditoría

---

### PF-002-21: Error de validación en backend

**Descripción**: Verificar que los errores de validación del backend se muestran apropiadamente.

**Datos de Entrada**:
- Pagador: `Testdfx`
- Condición: Backend rechaza por validación (ej: tasa fuera de rango permitido en BD)

**Pasos**:
1. Ingresar datos que pasen validación de frontend
2. Confirmar cambios
3. Backend retorna error de validación

**Resultado Esperado**:
- ✅ Se muestra mensaje de error específico del backend
- ✅ Si el error es por campo específico, se muestra junto al campo
- ✅ La modal de edición se mantiene abierta
- ✅ Los datos ingresados no se pierden

---

### PF-002-22: Usuario sin permisos no ve icono de editar

**Descripción**: Verificar que usuarios sin permisos de edición no pueden acceder a la funcionalidad.

**Datos de Entrada**:
- Usuario: `consultor@factoring.com` (rol: Consultor - sin permisos de edición)

**Pasos**:
1. Iniciar sesión con usuario sin permisos
2. Navegar a pantalla de pagadores

**Resultado Esperado**:
- ✅ La tabla de pagadores se muestra normalmente
- ✅ La columna "Acciones" no está visible, O
- ✅ El ícono de editar no está visible, O
- ✅ El ícono está visible pero deshabilitado (grayed out)
- ✅ Si intenta acceder por URL, muestra error de permisos

---

### PF-002-23: Editar múltiples pagadores secuencialmente

**Descripción**: Verificar que se pueden editar múltiples pagadores uno tras otro sin problemas.

**Datos de Entrada**:
- Pagadores a editar:
  1. `CADENA S.A.` - cambiar tasa descuento a 2.5
  2. `PRESIZA S.A.S` - cambiar tasa desembolso a 95
  3. `ENTREGA DE CARGA S.A.` - cambiar correo

**Pasos**:
1. Editar y guardar CADENA S.A.
2. Inmediatamente editar y guardar PRESIZA S.A.S
3. Inmediatamente editar y guardar ENTREGA DE CARGA S.A.

**Resultado Esperado**:
- ✅ Cada edición se completa exitosamente
- ✅ No hay conflictos entre operaciones
- ✅ Se crean 3 registros de auditoría independientes
- ✅ Todos los cambios se reflejan correctamente en la tabla
- ✅ El rendimiento no se degrada

---

### PF-002-24: Validación de precisión de decimales según BD

**Descripción**: Verificar que la precisión de decimales respeta el esquema de la base de datos.

**Datos de Entrada**:
- Pagador: `INVERSEGC SAS`
- Valor a probar: `2.12345` (muchos decimales)
- Nota: Asumir que BD acepta 2 decimales

**Pasos**:
1. Abrir modal de edición
2. Ingresar valor con muchos decimales
3. Guardar

**Resultado Esperado**:
- ✅ El sistema redondea o trunca al número de decimales permitido por BD
- ✅ Se guarda como: `2.12` (si son 2 decimales)
- ✅ Se muestra el valor redondeado en la confirmación
- ✅ O se muestra error indicando el máximo de decimales permitidos

---

### PF-002-25: Concurrencia - Dos usuarios editan el mismo pagador

**Descripción**: Verificar el manejo de concurrencia cuando dos usuarios intentan editar el mismo pagador simultáneamente.

**Datos de Entrada**:
- Pagador: `CADENA S.A.`
- Usuario A: Cambia tasa descuento a 2.5
- Usuario B: Cambia tasa descuento a 3.0
- Usuario A confirma primero

**Pasos**:
1. Usuario A abre modal de edición
2. Usuario B abre modal de edición (mismo pagador)
3. Usuario A guarda cambios (tasa = 2.5)
4. Usuario B intenta guardar cambios (tasa = 3.0)

**Resultado Esperado**:
- ✅ Usuario A guarda exitosamente con tasa = 2.5
- ✅ Usuario B recibe advertencia:
  - "Los datos han sido modificados por otro usuario", O
  - Se muestran los datos actuales y se solicita confirmación
- ✅ Se previenen conflictos de escritura
- ✅ La última modificación no sobrescribe ciegamente la primera

---

### PF-002-26: Cerrar modal con ESC (Accesibilidad)

**Descripción**: Verificar que la modal puede cerrarse con la tecla ESC.

**Datos de Entrada**:
- Pagador: Cualquiera
- Acción: Presionar ESC

**Pasos**:
1. Abrir modal de edición
2. Presionar tecla ESC

**Resultado Esperado**:
- ✅ La modal se cierra
- ✅ Comportamiento igual que "Cancelar"

---

### PF-002-27: Navegación con teclado (Accesibilidad)

**Descripción**: Verificar que la modal es accesible mediante teclado.

**Datos de Entrada**:
- Método: Solo teclado (sin mouse)

**Pasos**:
1. Abrir modal de edición
2. Usar TAB para navegar entre campos
3. Usar ENTER/SPACE para activar botones

**Resultado Esperado**:
- ✅ Se puede navegar con TAB entre:
  - Tasa de descuento
  - Tasa de desembolso
  - Correo de contacto
  - Botón Cancelar
  - Botón Guardar
  - Botón X (cerrar)
- ✅ El foco es visible en cada elemento
- ✅ El orden de tabulación es lógico
- ✅ ENTER activa el botón enfocado

---

### PF-002-28: Responsive - Modal en dispositivos móviles

**Descripción**: Verificar que la modal funciona correctamente en móviles.

**Datos de Entrada**:
- Dispositivos: iPhone 12, Samsung Galaxy S21, iPad
- Orientaciones: Portrait y Landscape

**Pasos**:
1. Acceder desde dispositivo móvil
2. Abrir modal de edición
3. Editar campos
4. Confirmar y guardar

**Resultado Esperado**:
- ✅ La modal se adapta al tamaño de pantalla
- ✅ Todos los campos son accesibles
- ✅ Los botones son lo suficientemente grandes para tocar
- ✅ El teclado numérico aparece para campos numéricos
- ✅ El teclado email aparece para campo de correo
- ✅ La funcionalidad completa funciona en móvil

---

### PF-002-29: Validación de longitud máxima de correo

**Descripción**: Verificar que se respeta la longitud máxima del campo correo según BD.

**Datos de Entrada**:
- Pagador: Cualquiera
- Correo: Email muy largo (> 255 caracteres, si ese es el límite)

**Pasos**:
1. Abrir modal de edición
2. Ingresar email muy largo
3. Intentar guardar

**Resultado Esperado**:
- ✅ Se muestra error indicando longitud máxima permitida
- ✅ O el campo limita la entrada al máximo de caracteres
- ✅ No se permite guardar si excede el límite

---

### PF-002-30: Valores de borde - Tasa 0%

**Descripción**: Verificar que se aceptan valores de 0 para las tasas (valor mínimo válido).

**Datos de Entrada**:
- Pagador: `testakshavw`
- Valores:
  - Tasa descuento: `0`
  - Tasa desembolso: `0`

**Pasos**:
1. Abrir modal de edición
2. Ingresar 0 en ambas tasas
3. Guardar

**Resultado Esperado**:
- ✅ Los valores 0 son aceptados
- ✅ Se guarda correctamente en BD
- ✅ No se muestra error

---

### PF-002-31: Valores de borde - Tasa 100%

**Descripción**: Verificar que se acepta 100 como valor máximo para tasa de desembolso.

**Datos de Entrada**:
- Pagador: Cualquiera
- Tasa desembolso: `100`

**Pasos**:
1. Ingresar 100 en tasa de desembolso
2. Guardar

**Resultado Esperado**:
- ✅ El valor 100 es aceptado
- ✅ Se guarda correctamente

---

## Matriz de Cobertura

| Escenario de Aceptación | Casos de Prueba Relacionados | Estado |
|--------------------------|------------------------------|--------|
| 1. Visualización de columna acciones | PF-002-01 | ☐ |
| 2. Abrir modal de edición | PF-002-02, PF-002-03 | ☐ |
| 3. Validación campos requeridos | PF-002-04, PF-002-05, PF-002-06 | ☐ |
| 4. Validación tipos de datos | PF-002-07 a PF-002-13, PF-002-24, PF-002-29 a PF-002-31 | ☐ |
| 5. Cancelar edición | PF-002-14, PF-002-15 | ☐ |
| 6. Modal de confirmación | PF-002-16, PF-002-17 | ☐ |
| 7. Guardar exitosamente | PF-002-18, PF-002-19 | ☐ |
| 8. Manejo de errores | PF-002-20, PF-002-21 | ☐ |
| 9. Pre-carga de valores | PF-002-02, PF-002-03 | ☐ |
| 10. Validación de permisos | PF-002-22 | ☐ |
| 11. Responsive | PF-002-28 | ☐ |
| Concurrencia | PF-002-25 | ☐ |
| Accesibilidad | PF-002-26, PF-002-27 | ☐ |

---

## Datos de Prueba Sugeridos

### Pagadores de Prueba

| Razón Social | NIT | Tasa Descuento | Tasa Desembolso | Correo | Estado |
|--------------|-----|----------------|-----------------|--------|--------|
| CADENA S.A. | 890930534 | 2.0 | 90 | test@gmail.com | Activo |
| PRESIZA S.A.S | 901453011 | 2.0 | 90 | test@gmail.com | Inactivo |
| ENTREGA DE CARGA S.A. | 800114437 | 2.0 | 90 | email@oficina.com | Activo |
| testakshavw | 098723451 | 0 | 0 | null | Activo |
| Testdfx | 757575600 | 2.0 | 4 | akshav@factorfox.com | Activo |
| IMPORTLOBEX S.A.S | 901045489 | 2.2 | 90 | null | Activo |
| DINDELCO S.A.S. | 900439177 | 2.2 | 90 | null | Activo |
| INVERSEGC SAS | (sin datos) | 0 | 90 | null | Activo |

### Usuarios de Prueba

| Email | Rol | Permisos Edición |
|-------|-----|------------------|
| admin@factoring.com | Administrador | ✅ Sí |
| operador@factoring.com | Operaciones | ✅ Sí |
| consultor@factoring.com | Consultor | ❌ No |
| viewer@factoring.com | Visualizador | ❌ No |

### Valores de Prueba para Validaciones

**Tasas válidas**:
- `0`, `0.5`, `1`, `2`, `2.2`, `2.5`, `10`, `50`, `90`, `95.5`, `100`

**Tasas inválidas**:
- `-1`, `-5`, `101`, `150`, `abc`, `texto`, `null`

**Emails válidos**:
- `usuario@dominio.com`
- `nombre.apellido@empresa.com.co`
- `usuario+tag@dominio.com`
- `test123@factoring.io`

**Emails inválidos**:
- `correo-sin-arroba.com`
- `@dominio.com`
- `usuario@`
- `usuario dominio.com` (con espacio)
- `usuario@dominio` (sin TLD)

---

## Criterios de Aceptación de Pruebas

- ✅ Todos los casos de prueba deben pasar exitosamente
- ✅ No debe haber errores en consola del navegador
- ✅ El tiempo de respuesta de guardado debe ser < 2 segundos
- ✅ La cobertura de código debe ser >= 80%
- ✅ Las pruebas deben pasar en: Chrome, Firefox, Safari, Edge
- ✅ Las pruebas responsive deben pasar en al menos 3 dispositivos diferentes
- ✅ Los registros de auditoría deben crearse correctamente para cada cambio
- ✅ Las validaciones de frontend y backend deben estar sincronizadas

---

## Notas de Ejecución

**Ambiente de Pruebas**: [Especificar URL del ambiente QA]

**Base de Datos**: [Especificar instancia de BD de pruebas]

**Verificar esquema de BD antes de ejecutar**:
- Tipo de dato de `tasa_descuento_defecto`
- Tipo de dato de `tasa_desembolso_defecto`
- Precisión de decimales permitida
- Longitud máxima de `correo_contacto`

**Fecha de Ejecución**: _______________________

**Ejecutado por**: _______________________

**Observaciones**:
_________________________________________________________________
_________________________________________________________________
_________________________________________________________________

---

**Última actualización**: 2025-12-09
