# Edición de Información de Pagadores

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

Como usuario del sistema de factoring, quiero poder editar la información de configuración de los **pagadores** (tasa de descuento por defecto, tasa de desembolso por defecto y correo de contacto) desde la pantalla de gestión de pagadores, para mantener actualizada la información de las tasas y contactos de cada pagador sin necesidad de soporte técnico.

### Roles
- **Administrador**: Usuario con permisos para editar información de pagadores
- **Operaciones**: Usuario con permisos para editar información de pagadores

### Característica / Funcionalidad
Edición de parámetros de configuración de pagadores mediante modal con validaciones y confirmación

### Razón / Resultado
Permitir la actualización ágil de las tasas por defecto y correos de contacto de los pagadores, mejorando la flexibilidad operativa y reduciendo la dependencia de cambios técnicos para ajustes de configuración comercial.

---

## Escenarios

| Número | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|--------|--------------------------------|----------|--------|-------------------------------------|------------|----|--------------|
| 1 | Visualización de columna de acciones | El usuario accede a la pantalla de gestión de pagadores | El usuario visualiza la tabla de pagadores | Se muestra una nueva columna "Acciones" en la tabla de pagadores, ubicada después de las columnas existentes. Cada fila debe tener un icono de "Editar" (lápiz/pencil) que sea clickeable. | ☐ | ☐ | ☐ |
| 2 | Abrir modal de edición | El usuario visualiza la tabla de pagadores con la columna de acciones | El usuario hace clic en el icono "Editar" de un pagador específico | Se abre una modal titulada "Editar Pagador" que muestra:<br>**Campos no editables (solo lectura)**:<br>- Razón Social del pagador<br>- NIT del pagador<br><br>**Campos editables**:<br>- Tasa de descuento por defecto* (campo numérico, requerido)<br>- Tasa de desembolso por defecto* (campo numérico, requerido)<br>- Correo de contacto (campo email, opcional)<br><br>Los campos editables deben venir pre-cargados con los valores actuales de la base de datos si existen. | ☐ | ☐ | ☐ |
| 3 | Validación de campos requeridos | El usuario tiene la modal de edición abierta | El usuario intenta guardar sin completar los campos marcados con asterisco (*) | El sistema debe:<br>1. Prevenir el guardado<br>2. Mostrar mensajes de error debajo de cada campo requerido vacío<br>3. Resaltar visualmente los campos con error (ej: borde rojo)<br>4. Mensaje: "Este campo es requerido"<br><br>Los campos requeridos son:<br>- Tasa de descuento por defecto*<br>- Tasa de desembolso por defecto* | ☐ | ☐ | ☐ |
| 4 | Validación de tipos de datos | El usuario ingresa datos en los campos editables | El usuario ingresa datos y intenta guardar | El sistema debe validar:<br><br>**Tasa de descuento por defecto**:<br>- Tipo: Numérico decimal<br>- Formato: Porcentaje (ej: 2.2, 0.5, 10)<br>- Rango: Mayor o igual a 0<br>- Decimales permitidos según BD<br><br>**Tasa de desembolso por defecto**:<br>- Tipo: Numérico decimal<br>- Formato: Porcentaje (ej: 90, 95.5, 100)<br>- Rango: Mayor o igual a 0, menor o igual a 100<br>- Decimales permitidos según BD<br><br>**Correo de contacto**:<br>- Tipo: String con formato email<br>- Validación de formato: xxx@xxx.xxx<br>- Puede estar vacío (opcional)<br><br>Si la validación falla, mostrar mensaje de error específico por campo. | ☐ | ☐ | ☐ |
| 5 | Cancelar edición | El usuario tiene la modal de edición abierta con cambios sin guardar | El usuario hace clic en el botón "Cancelar" o en la X de cerrar modal | La modal se cierra sin guardar cambios. No se realizan modificaciones en la base de datos. Los datos del pagador permanecen sin cambios. Opcionalmente, si hay cambios sin guardar, mostrar confirmación: "¿Está seguro de cerrar sin guardar los cambios?" | ☐ | ☐ | ☐ |
| 6 | Solicitar confirmación de guardado | El usuario completa todos los campos requeridos con datos válidos | El usuario hace clic en el botón "Guardar" | Se muestra una segunda modal de confirmación titulada "Confirmar Cambios" que muestra:<br><br>**Datos del pagador**:<br>- Razón Social: [nombre]<br>- NIT: [número]<br><br>**Datos a actualizar**:<br>- Tasa de descuento por defecto: [valor nuevo]%<br>- Tasa de desembolso por defecto: [valor nuevo]%<br>- Correo de contacto: [email nuevo]<br><br>Con botones:<br>- "Cancelar" (vuelve a la modal de edición)<br>- "Confirmar" (procede con el guardado) | ☐ | ☐ | ☐ |
| 7 | Guardar cambios exitosamente | El usuario confirma los cambios en la modal de confirmación | El usuario hace clic en "Confirmar" | Se ejecutan las siguientes acciones:<br>1. Se envía la petición al backend para actualizar los datos<br>2. Se muestra un indicador de carga/procesamiento<br>3. Al recibir respuesta exitosa:<br>   - Se cierran ambas modales (confirmación y edición)<br>   - Se muestra mensaje de éxito: "Datos del pagador actualizados correctamente"<br>   - Se actualiza la tabla de pagadores con los nuevos valores<br>   - Se registra auditoría del cambio (usuario, fecha, hora, campos modificados) | ☐ | ☐ | ☐ |
| 8 | Manejo de errores en guardado | El usuario confirma los cambios | Ocurre un error al intentar guardar en la base de datos | Se muestra mensaje de error al usuario:<br>"No se pudieron guardar los cambios. Por favor, intente nuevamente."<br><br>La modal de confirmación se cierra y regresa a la modal de edición con los datos ingresados por el usuario (no se pierden los cambios). El usuario puede:<br>- Intentar guardar nuevamente<br>- Modificar los datos<br>- Cancelar la operación | ☐ | ☐ | ☐ |
| 9 | Pre-carga de valores existentes | El usuario abre la modal de edición de un pagador que ya tiene datos configurados | El usuario hace clic en "Editar" | Los campos editables deben mostrar los valores actuales almacenados en la base de datos:<br>- Si el campo tiene valor en BD, se muestra ese valor<br>- Si el campo opcional está vacío en BD, se muestra el input vacío<br>- Los campos requeridos siempre deben tener un valor (por configuración inicial o por esta funcionalidad)<br><br>Ejemplo:<br>- Tasa descuento: 2.2 (valor actual)<br>- Tasa desembolso: 90 (valor actual)<br>- Correo: test@gmail.com (valor actual o vacío) | ☐ | ☐ | ☐ |
| 10 | Validación de permisos | Usuario sin permisos de edición accede a la pantalla de pagadores | El usuario visualiza la tabla de pagadores | El icono de "Editar" no debe estar visible o debe estar deshabilitado para usuarios sin permisos de edición. Alternativamente, si intenta editar, mostrar mensaje: "No tiene permisos para editar pagadores" | ☐ | ☐ | ☐ |
| 11 | Comportamiento responsive de la modal | El usuario abre la modal de edición en diferentes dispositivos | El usuario accede desde desktop, tablet o móvil | La modal debe adaptarse correctamente a diferentes tamaños de pantalla manteniendo la usabilidad y legibilidad de todos los campos y botones. | ☐ | ☐ | ☐ |

---

## Interacción con el usuario y prototipo

### Flujo de interacción:

#### 1. **Pantalla de Gestión de Pagadores**
```
┌─────────────────────────────────────────────────────────────────────────┐
│ Razón Social │ NIT │ Tasa │ % Desembolso │ Correo │ Estado │ Acciones │
├─────────────────────────────────────────────────────────────────────────┤
│ CADENA S.A.  │ 123 │ 2%   │ 90%          │ ...    │ Activo │  [✏️]    │
│ PRESIZA S.A.S│ 456 │ 2.2% │ 90%          │ ...    │ Activo │  [✏️]    │
└─────────────────────────────────────────────────────────────────────────┘
```

#### 2. **Modal de Edición** (al hacer clic en ✏️)

```
┌──────────────────────────────────────────────┐
│  Editar Pagador                          [X] │
├──────────────────────────────────────────────┤
│                                              │
│  Información del Pagador                     │
│  ───────────────────────────                 │
│  Razón Social: CADENA S.A. (no editable)    │
│  NIT: 123456789 (no editable)               │
│                                              │
│  Configuración de Tasas                      │
│  ───────────────────────                     │
│  Tasa de descuento por defecto *             │
│  ┌────────────────────┐                      │
│  │ 2.2               │ %                    │
│  └────────────────────┘                      │
│                                              │
│  Tasa de desembolso por defecto *            │
│  ┌────────────────────┐                      │
│  │ 90                │ %                    │
│  └────────────────────┘                      │
│                                              │
│  Correo de contacto                          │
│  ┌────────────────────────────────┐          │
│  │ test@gmail.com                │          │
│  └────────────────────────────────┘          │
│                                              │
│  * Campos requeridos                         │
│                                              │
│        [Cancelar]        [Guardar]           │
└──────────────────────────────────────────────┘
```

#### 3. **Modal de Confirmación** (al hacer clic en Guardar)

```
┌──────────────────────────────────────────────┐
│  Confirmar Cambios                       [X] │
├──────────────────────────────────────────────┤
│                                              │
│  ¿Confirma la actualización de los datos    │
│  del siguiente pagador?                      │
│                                              │
│  Pagador                                     │
│  ───────────────────────────                 │
│  Razón Social: CADENA S.A.                   │
│  NIT: 123456789                              │
│                                              │
│  Nuevos valores                              │
│  ───────────────────────────                 │
│  Tasa de descuento: 2.5%                     │
│  Tasa de desembolso: 95%                     │
│  Correo: nuevo@email.com                     │
│                                              │
│        [Cancelar]        [Confirmar]         │
└──────────────────────────────────────────────┘
```

#### 4. **Mensaje de Éxito**
```
┌────────────────────────────────────┐
│  ✓ Datos del pagador actualizados │
│    correctamente                   │
└────────────────────────────────────┘
```

#### 5. **Mensaje de Error**
```
┌────────────────────────────────────┐
│  ⚠ No se pudieron guardar los     │
│    cambios. Por favor, intente    │
│    nuevamente.                     │
└────────────────────────────────────┘
```

### Consideraciones de UX:

- **Indicadores visuales claros**: Campos requeridos marcados con asterisco (*)
- **Feedback inmediato**: Validaciones en tiempo real mientras el usuario escribe
- **Confirmación de cambios**: Siempre mostrar resumen antes de guardar
- **Mensajes descriptivos**: Errores específicos por campo (no genéricos)
- **Loading states**: Indicador de carga durante el guardado
- **Accesibilidad**:
  - Modal cerrable con tecla ESC
  - Navegación con tabulador entre campos
  - Labels descriptivos para lectores de pantalla
- **Preservación de datos**: Si hay error, no perder los datos ingresados por el usuario

---

## Definición de Terminado (DoD)

- [ ] El código cumple con los estándares de desarrollo del proyecto
- [ ] Se han implementado todos los escenarios de aceptación
- [ ] Las validaciones de frontend y backend están implementadas
- [ ] Las pruebas unitarias tienen una cobertura mínima del 80%
- [ ] Las pruebas de integración funcionan correctamente
- [ ] La funcionalidad ha sido probada en diferentes navegadores (Chrome, Firefox, Safari, Edge)
- [ ] La modal es responsive y funciona en mobile, tablet y desktop
- [ ] La documentación técnica está actualizada
- [ ] El registro de auditoría está funcionando correctamente
- [ ] El Product Owner ha validado la funcionalidad
- [ ] No existen bugs críticos pendientes

---

## Notas Técnicas

### Endpoint API (sugerido):

#### Obtener datos del pagador
```
GET /api/pagadores/{pagadorId}
```

**Response:**
```json
{
  "id": "uuid",
  "razonSocial": "CADENA S.A.",
  "nit": "123456789",
  "tasaDescuentoDefecto": 2.2,
  "tasaDesembolsoDefecto": 90,
  "correoContacto": "test@gmail.com",
  "estado": "Activo",
  "fechaCreacion": "2025-01-15T10:30:00Z",
  "fechaActualizacion": "2025-12-03T14:20:00Z"
}
```

#### Actualizar datos del pagador
```
PUT /api/pagadores/{pagadorId}
```

**Request Payload:**
```json
{
  "tasaDescuentoDefecto": 2.5,
  "tasaDesembolsoDefecto": 95,
  "correoContacto": "nuevo@email.com",
  "usuarioModificador": "user_id"
}
```

**Response (Éxito):**
```json
{
  "success": true,
  "message": "Pagador actualizado correctamente",
  "data": {
    "id": "uuid",
    "razonSocial": "CADENA S.A.",
    "nit": "123456789",
    "tasaDescuentoDefecto": 2.5,
    "tasaDesembolsoDefecto": 95,
    "correoContacto": "nuevo@email.com",
    "fechaActualizacion": "2025-12-03T15:45:00Z"
  }
}
```

**Response (Error):**
```json
{
  "success": false,
  "message": "Error al actualizar el pagador",
  "errors": [
    {
      "field": "tasaDescuentoDefecto",
      "message": "Debe ser un valor numérico mayor o igual a 0"
    }
  ]
}
```

### Validaciones en Backend:

```javascript
// Ejemplo de validaciones
{
  tasaDescuentoDefecto: {
    type: 'decimal',
    required: true,
    min: 0,
    precision: 2, // Ajustar según esquema de BD
    errorMessage: 'La tasa de descuento debe ser mayor o igual a 0'
  },
  tasaDesembolsoDefecto: {
    type: 'decimal',
    required: true,
    min: 0,
    max: 100,
    precision: 2, // Ajustar según esquema de BD
    errorMessage: 'La tasa de desembolso debe estar entre 0 y 100'
  },
  correoContacto: {
    type: 'string',
    required: false,
    format: 'email',
    maxLength: 255, // Ajustar según esquema de BD
    errorMessage: 'Debe ser un correo electrónico válido'
  }
}
```

### Registro de Auditoría:

Cada actualización debe registrarse en una tabla de auditoría con:
- ID del pagador
- Usuario que realizó el cambio
- Fecha y hora del cambio
- Campos modificados (before/after)
- IP del usuario (opcional)

```sql
-- Ejemplo de registro de auditoría
INSERT INTO auditoria_pagadores (
  pagador_id,
  usuario_id,
  accion,
  valores_anteriores,
  valores_nuevos,
  fecha_modificacion,
  ip_usuario
) VALUES (
  'uuid-pagador',
  'uuid-usuario',
  'ACTUALIZAR_CONFIGURACION',
  '{"tasaDescuentoDefecto": 2.2, "tasaDesembolsoDefecto": 90}',
  '{"tasaDescuentoDefecto": 2.5, "tasaDesembolsoDefecto": 95}',
  NOW(),
  '192.168.1.1'
);
```

### Tipos de Datos en Base de Datos:

**IMPORTANTE**: El desarrollador debe verificar los tipos de datos exactos en el esquema de la base de datos y aplicar las validaciones correspondientes.

Campos a verificar:
- `razon_social` - Tipo: VARCHAR/TEXT - Solo lectura
- `nit` - Tipo: VARCHAR - Solo lectura
- `tasa_descuento_defecto` - Tipo: DECIMAL/FLOAT - Verificar precisión
- `tasa_desembolso_defecto` - Tipo: DECIMAL/FLOAT - Verificar precisión
- `correo_contacto` - Tipo: VARCHAR - Verificar longitud máxima

### Componentes Reutilizables:

- **Modal genérica**: Reutilizar componente modal existente o crear uno nuevo
- **Input numérico con sufijo**: Para campos de porcentaje
- **Input email**: Con validación de formato
- **Botones de acción**: Mantener consistencia con el resto de la aplicación
- **Mensajes toast/snackbar**: Para feedback de éxito/error

---

## Dependencias

- Sistema de autenticación y autorización por roles
- Base de datos con modelo de pagadores
- Componente modal reutilizable
- Sistema de notificaciones/mensajes al usuario
- Sistema de auditoría implementado
- Librería de validación de formularios (ej: Yup, Joi, Zod)

---

## Riesgos y Mitigaciones

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|--------------|---------|------------|
| Concurrencia: dos usuarios editan el mismo pagador simultáneamente | Media | Alto | Implementar bloqueo optimista con versioning o timestamp de última actualización |
| Validaciones diferentes entre frontend y backend | Media | Medio | Mantener validaciones sincronizadas, idealmente compartir esquema de validación |
| Pérdida de datos ingresados por error de red | Media | Medio | Implementar auto-guardado en localStorage o mantener datos en modal al mostrar error |
| Tipos de datos incorrectos que rompan la BD | Baja | Alto | Validación estricta en backend antes de guardar, usar prepared statements |
| Usuario no entiende qué campos son requeridos | Baja | Bajo | Marcadores visuales claros (*), tooltips explicativos, mensajes de error descriptivos |
| Datos precargados no coinciden con BD actual | Baja | Medio | Siempre hacer GET antes de mostrar modal, implementar caché invalidation apropiado |

---

## Casos de Prueba Sugeridos

### Pruebas Funcionales:
1. Editar pagador cambiando solo tasa de descuento
2. Editar pagador cambiando solo tasa de desembolso
3. Editar pagador cambiando solo correo de contacto
4. Editar pagador cambiando todos los campos
5. Intentar guardar con campos requeridos vacíos
6. Intentar guardar con valores no numéricos en tasas
7. Intentar guardar con email inválido
8. Cancelar edición sin guardar
9. Cancelar en modal de confirmación
10. Editar pagador sin correo actual (campo opcional vacío)

### Pruebas de Validación:
1. Tasas con decimales (2.5, 2.25, 0.5)
2. Tasas sin decimales (2, 90, 100)
3. Tasa de descuento negativa (debe fallar)
4. Tasa de desembolso mayor a 100 (debe fallar)
5. Email con formato incorrecto
6. Email con caracteres especiales válidos
7. Campos muy largos (más allá del límite de BD)

### Pruebas de Integración:
1. Verificar que los cambios se reflejan en la tabla inmediatamente
2. Verificar que los cambios persisten después de recargar la página
3. Verificar que el registro de auditoría se crea correctamente
4. Verificar comportamiento al perder conexión durante guardado

---

**Fecha de creación**: 2025-12-09
**Última actualización**: 2025-12-09
