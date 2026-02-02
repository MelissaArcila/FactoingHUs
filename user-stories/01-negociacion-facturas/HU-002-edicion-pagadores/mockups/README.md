# Mockups - HU-002: Edición de Información de Pagadores

Este directorio contiene los mockups interactivos en formato HTML para la historia de usuario HU-002 (Edición de Información de Pagadores).

## 🚀 Inicio Rápido

**¿Primera vez viendo estos mockups?** Tienes dos opciones para comenzar:

### Opción 1: [🏠 Índice Visual Interactivo](./index.html) ⭐ MUY RECOMENDADO

Página de inicio con vista de tarjetas de todos los mockups, descripciones y acceso directo a cada uno. **Ideal para navegar y explorar todos los diseños.**

### Opción 2: [📋 Flujo Completo de Usuario](./00-flujo-completo.html) ⭐ RECOMENDADO

Este mockup interactivo te guía paso a paso por todo el flujo de la funcionalidad, desde que el usuario ve la tabla hasta que guarda exitosamente los cambios. **Ideal para entender rápidamente cómo funciona toda la feature.**

---

## Índice de Mockups

### 1. [Tabla de Pagadores con Columna de Acciones](./01-tabla-pagadores-con-acciones.html)
**Escenario:** Vista principal de gestión de pagadores
**Descripción:**
- Muestra la tabla completa de pagadores con todas las columnas existentes
- Incluye la nueva columna "Acciones" destacada al final
- Cada fila tiene un icono de editar (lápiz) clickeable
- Mantiene el diseño y estilo actual de la aplicación

**Elementos clave:**
- Filtros de búsqueda en la parte superior
- Columna de acciones resaltada con fondo de color suave
- Iconos de edición con hover effects
- Header con logo y usuario

---

### 2. [Modal de Edición](./02-modal-edicion.html)
**Escenario:** Usuario hace clic en el icono de editar
**Descripción:**
- Modal centrada con overlay de fondo oscuro
- Secciones claramente separadas: Información del Pagador, Configuración de Tasas, Contacto
- Campos no editables (Razón Social y NIT) con estilo disabled
- Campos editables con validación visual
- Inputs numéricos con sufijo de porcentaje (%)
- Nota visual de campos requeridos

**Elementos clave:**
- Header con título y botón de cerrar
- Campos de solo lectura con fondo gris claro
- Inputs de tasas con sufijo "%"
- Footer con botones de acción (Cancelar/Guardar)
- Hints informativos debajo de cada campo

---

### 3. [Modal de Confirmación](./03-modal-confirmacion.html)
**Escenario:** Usuario hace clic en "Guardar" después de editar
**Descripción:**
- Modal de confirmación con diseño destacado
- Muestra resumen de los datos del pagador
- Resalta los nuevos valores con fondo amarillo suave
- Mensaje de advertencia sobre auditoría
- Botones de confirmación final

**Elementos clave:**
- Header con color de fondo distintivo (rojo suave)
- Sección de información del pagador (solo lectura)
- Sección de "Nuevos Valores" con highlighting
- Warning box sobre registro de auditoría
- Botones: Cancelar y Confirmar

---

### 4. [Modal con Errores de Validación](./04-modal-con-errores-validacion.html)
**Escenario:** Usuario intenta guardar con datos inválidos
**Descripción:**
- Muestra el estado de error en la modal de edición
- Banner de resumen de errores en la parte superior
- Campos con error resaltados con borde rojo y fondo rosado
- Mensajes de error específicos debajo de cada campo
- Botón "Guardar" deshabilitado

**Elementos de validación mostrados:**
- Campo requerido vacío: "Este campo es requerido"
- Valor fuera de rango: "El valor debe estar entre 0 y 100"
- Email inválido: "Debe ser un correo electrónico válido"

**Elementos clave:**
- Alert box de resumen de errores
- Inputs con clase de error (border rojo)
- Iconos de error al lado de mensajes
- Animación de shake al intentar guardar

---

### 5. [Mensajes de Estado](./05-mensajes-estado.html)
**Escenario:** Diferentes estados de feedback al usuario
**Descripción:**
Muestra todos los tipos de mensajes de estado que puede ver el usuario:

**4.1. Toast Notifications (Recomendado)**
- Éxito: "Datos del pagador actualizados correctamente"
- Error: "No se pudieron guardar los cambios"
- Aparecen en esquina superior derecha
- Se ocultan automáticamente después de 4-5 segundos

**4.2. Estado de Carga**
- Spinner animado con texto "Guardando cambios..."
- Overlay sobre la modal o contenido
- Previene interacciones mientras se procesa

**4.3. Mensajes Inline**
- Mensajes que aparecen sobre la tabla
- Versiones de éxito y error
- Permanecen visibles hasta que el usuario continúe

**4.4. Banner Alerts**
- Mensajes prominentes con más detalle
- Ocupan todo el ancho
- Ideales para mensajes importantes

**Incluye recomendaciones de uso para cada tipo**

---

### 6. [Vista Responsive Mobile/Tablet](./06-responsive-mobile.html)
**Escenario:** Usuario accede desde dispositivos móviles o tablets
**Descripción:**
Muestra cómo se adapta la modal de edición en diferentes tamaños de pantalla:

**6.1. Mobile (375px)**
- Modal ocupa 100% del ancho
- Botones apilados verticalmente
- Padding optimizado para touch
- Campos a ancho completo
- Scroll vertical para contenido largo

**6.2. Tablet (768px)**
- Modal ocupa 90% con márgenes
- Campos pueden ir en grids de 2 columnas
- Botones horizontales en el footer
- Espaciado más generoso

**Elementos clave:**
- Ejemplos visuales de ambos dispositivos
- Guías de diseño responsive
- Consideraciones de usabilidad táctil
- Notas sobre font-sizes mínimos

---

## Cómo Usar Estos Mockups

### Visualización
1. Abre cualquier archivo HTML en tu navegador web
2. Los mockups son completamente estáticos (no interactivos)
3. Están diseñados para revisión visual y aprobación de diseño

### Para Desarrolladores
- Los estilos CSS están inline en cada archivo para facilitar la referencia
- Los colores, espaciados y tamaños siguen el sistema de diseño actual
- Los componentes pueden extraerse y adaptarse al framework usado (React, Vue, etc.)

### Para Product Owners
- Revisa cada mockup en orden secuencial (01 al 06)
- Verifica que el flujo coincida con los criterios de aceptación de HU-002
- Valida los mensajes de error y estados
- Confirma que el diseño responsive cumple con las necesidades móviles

---

## Paleta de Colores Utilizada

```css
/* Colores Principales */
Rojo principal (acciones/botones): #ef4444
Rojo hover: #dc2626
Rojo claro (backgrounds): #fef2f2

/* Grises */
Texto principal: #111827
Texto secundario: #6b7280
Bordes: #d1d5db, #e5e7eb
Fondos: #f9fafb, #fafafa

/* Estados */
Éxito (verde): #16a34a, #dcfce7
Error (rojo): #ef4444, #fef2f2
Warning (amarillo): #f59e0b, #fef3c7
Info (azul): #3b82f6, #eff6ff
```

---

## Sistema de Espaciado

```css
/* Padding/Margin */
Pequeño: 8px, 10px, 12px
Mediano: 16px, 20px, 24px
Grande: 30px, 32px, 40px

/* Border Radius */
Componentes pequeños: 4px, 6px
Modales y cards: 8px, 12px
Badges: 12px (pill shape)

/* Font Sizes */
Pequeño: 12px, 13px
Normal: 14px
Títulos: 16px, 18px, 20px
```

---

## Notas de Implementación

### Accesibilidad
- Todos los modales deben ser cerrables con tecla ESC
- Navegación por teclado (Tab) entre campos
- Labels asociados a inputs con atributo `for`
- Colores con suficiente contraste (WCAG AA)
- Mensajes de error descriptivos y únicos

### Performance
- Modales deben usar lazy loading si es posible
- Animaciones con CSS transforms (mejor performance)
- Debounce en validaciones en tiempo real
- Loading states para prevenir clicks duplicados

### Testing
Cada mockup corresponde a escenarios específicos de la historia:
- **Mockup 01:** Escenario 1 (Visualización de columna de acciones)
- **Mockup 02:** Escenarios 2, 9 (Abrir modal, Pre-carga de valores)
- **Mockup 03:** Escenario 6 (Solicitar confirmación de guardado)
- **Mockup 04:** Escenarios 3, 4 (Validaciones)
- **Mockup 05:** Escenarios 7, 8 (Guardado exitoso, Errores)
- **Mockup 06:** Escenario 11 (Comportamiento responsive)

---

## Historial de Versiones

| Versión | Fecha | Cambios |
|---------|-------|---------|
| 1.0 | 2025-12-09 | Creación inicial de todos los mockups |

---

## Contacto

**Diseñadora:** Melissa Arcila Restrepo
**Historia de Usuario:** HU-002
**Epic:** Negociación de Facturas

Para preguntas o sugerencias sobre estos mockups, por favor contactar al equipo de diseño.
