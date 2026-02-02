# Historias de Usuario - Proyecto Factoring

Este directorio contiene todas las historias de usuario del proyecto de factoring, organizadas por épicas.

## Estructura de Directorios

```
user-stories/
├── 01-negociacion-facturas/                    # Épica: Negociación de Facturas
│   ├── HU-001-aprobacion-facturas-negociadas/
│   │   ├── HU-001-aprobacion-facturas-negociadas.md
│   │   └── pruebas-funcionales.md
│   ├── HU-002-edicion-pagadores/
│   │   ├── HU-002-edicion-pagadores.md
│   │   └── pruebas-funcionales.md
│   ├── HU-003-edicion-masiva-pagadores/
│   │   ├── HU-003-edicion-masiva-pagadores.md
│   │   └── pruebas-funcionales.md
│   └── HU-004-carga-facturas-externas/
│       ├── HU-004-carga-facturas-externas.md
│       └── pruebas-funcionales.md
├── 02-gestion-pagos/                           # Épica: Gestión de Pagos (futuro)
├── 03-reportes/                                # Épica: Reportes y Analytics (futuro)
└── README.md
```

## Épicas del Proyecto

### 1. Negociación de Facturas
**Estado**: En desarrollo
**Descripción**: Funcionalidades relacionadas con el proceso de negociación y aprobación de facturas.

**Historias de Usuario**:
- [HU-001: Aprobación de Facturas Negociadas](./01-negociacion-facturas/HU-001-aprobacion-facturas-negociadas/HU-001-aprobacion-facturas-negociadas.md) - BORRADOR | [Pruebas](./01-negociacion-facturas/HU-001-aprobacion-facturas-negociadas/pruebas-funcionales.md)
- [HU-002: Edición de Información de Pagadores](./01-negociacion-facturas/HU-002-edicion-pagadores/HU-002-edicion-pagadores.md) - BORRADOR | [Pruebas](./01-negociacion-facturas/HU-002-edicion-pagadores/pruebas-funcionales.md)
- [HU-003: Edición Masiva de Pagadores](./01-negociacion-facturas/HU-003-edicion-masiva-pagadores/HU-003-edicion-masiva-pagadores.md) - BORRADOR | [Pruebas](./01-negociacion-facturas/HU-003-edicion-masiva-pagadores/pruebas-funcionales.md)
- [HU-004: Carga de Facturas Electrónicas Externas](./01-negociacion-facturas/HU-004-carga-facturas-externas/HU-004-carga-facturas-externas.md) - BORRADOR | [Pruebas](./01-negociacion-facturas/HU-004-carga-facturas-externas/pruebas-funcionales.md)

### 2. Gestión de Pagos
**Estado**: Planificado
**Descripción**: Funcionalidades para la gestión y seguimiento de pagos de facturas.

### 3. Reportes y Analytics
**Estado**: Planificado
**Descripción**: Generación de reportes y análisis de datos del sistema de factoring.

## Organización de Historias de Usuario

Cada historia de usuario está organizada en su propia carpeta que contiene:

1. **Documento de Historia de Usuario** (`HU-XXX-nombre.md`): Descripción completa de la funcionalidad
2. **Documento de Pruebas Funcionales** (`pruebas-funcionales.md`): Casos de prueba detallados

Esta estructura permite mantener toda la documentación relacionada con cada historia en un solo lugar.

## Formato de Historias de Usuario

Todas las historias de usuario siguen el siguiente formato estándar:

1. **Información General**: Release, Epic, Estado, Roles
2. **Contexto**: Enunciado general, roles involucrados, funcionalidad y razón
3. **Escenarios**: Criterios de aceptación en formato tabular con columnas:
   - Número
   - Criterio de aceptación (Título)
   - Contexto
   - Evento
   - Resultado / Comportamiento esperado
   - Desarrollo (checkbox)
   - QA (checkbox)
   - Product Owner (checkbox)
4. **Interacción con el usuario y prototipo**: Flujos de interacción y consideraciones de UX
5. **Definición de Terminado (DoD)**: Checklist de completitud
6. **Notas Técnicas**: Detalles de implementación, endpoints, validaciones
7. **Dependencias y Riesgos**: Elementos necesarios y riesgos identificados

## Formato de Pruebas Funcionales

Cada documento de pruebas funcionales contiene:

1. **Objetivo**: Propósito general de las pruebas
2. **Pre-requisitos**: Condiciones necesarias para ejecutar las pruebas
3. **Casos de Prueba**: Cada caso incluye:
   - **Descripción**: Qué es lo que se quiere probar o descartar
   - **Datos de Entrada**: Información específica y valores de prueba
   - **Pasos**: Secuencia de acciones a realizar
   - **Resultado Esperado**: Comportamiento esperado del sistema
4. **Matriz de Cobertura**: Mapeo de casos de prueba a escenarios de aceptación
5. **Datos de Prueba Sugeridos**: Conjuntos de datos para ejecutar las pruebas
6. **Criterios de Aceptación**: Condiciones para considerar las pruebas exitosas

## Convenciones de Nomenclatura

- **Directorios de épicas**: `XX-nombre-epica/`
- **Directorios de HU**: `HU-XXX-nombre-descriptivo/` (dentro de la carpeta de épica)
- **Archivos de HU**: `HU-XXX-nombre-descriptivo.md` (dentro de su carpeta)
- **Archivos de pruebas**: `pruebas-funcionales.md` (dentro de la carpeta de la HU)
- Usar kebab-case para nombres de archivos y directorios
- Numerar las HU de forma secuencial por épica
- Cada HU debe tener su propia carpeta conteniendo todos sus documentos relacionados

## Estados de Historias de Usuario

- **BORRADOR**: Historia en proceso de definición
- **PENDIENTE REVISIÓN**: Lista para revisión del equipo
- **APROBADA**: Aprobada y lista para desarrollo
- **EN DESARROLLO**: En proceso de implementación
- **EN QA**: En proceso de pruebas
- **COMPLETADA**: Implementada y desplegada

## Proceso de Trabajo

1. **Creación**: El Product Owner o analista crea la HU en estado BORRADOR
2. **Revisión**: El equipo técnico revisa y aporta feedback
3. **Aprobación**: El Product Owner aprueba la HU
4. **Desarrollo**: El equipo de desarrollo implementa
5. **QA**: El equipo de QA valida los criterios de aceptación
6. **Aceptación**: El Product Owner valida la implementación
7. **Completado**: La HU se marca como completada

## Roles del Proyecto

- **Product Owner**: Define y prioriza historias de usuario
- **Arquitecto**: Revisa aspectos técnicos y de arquitectura
- **Desarrolladores**: Implementan las funcionalidades
- **Tester/QA**: Valida los criterios de aceptación
- **Operaciones**: Usuario final con permisos de aprobación de facturas

## Contribuir

Para agregar una nueva historia de usuario:

1. Crear una carpeta para la HU en el directorio de la épica correspondiente: `HU-XXX-nombre/`
2. Crear el archivo de la historia de usuario siguiendo el formato estándar
3. Crear el archivo `pruebas-funcionales.md` con los casos de prueba
4. Asignar el siguiente número de HU disponible
5. Actualizar este README con la nueva HU y enlace a sus pruebas
6. Crear un commit descriptivo y abrir un PR para revisión

**Ejemplo de estructura al crear HU-003**:
```
01-negociacion-facturas/
└── HU-003-nombre-funcionalidad/
    ├── HU-003-nombre-funcionalidad.md
    └── pruebas-funcionales.md
```

---

**Última actualización**: 2025-12-09
**Mantenido por**: Melissa Arcila Restrepo
