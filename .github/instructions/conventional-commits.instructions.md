---
description: 'Reglas y estándares para la generación de mensajes de commit siguiendo la especificación Conventional Commits'
applyTo: '**'
---

# Conventional Commits Standards

Guía para generar mensajes de commit claros, semánticos y estandarizados siguiendo la especificación Conventional Commits.

## Instrucciones Generales

- **Estructura obligatoria**: Utiliza siempre el formato `<tipo>[ámbito opcional]: <descripción>`.
- **Imperativo**: Escribe la descripción en modo imperativo ("agrega", "corrige", "elimina") y en español.
- **Concisión**: Mantén la primera línea (encabezado) por debajo de los 72 caracteres si es posible.
- **Cuerpo**: Usa el cuerpo del mensaje para explicar el *qué* y el *por qué*, no el *cómo*.
- **Separación**: Deja siempre una línea en blanco entre el encabezado y el cuerpo.

## Tipos de Commit Permitidos

Utiliza estrictamente uno de los siguientes tipos para el encabezado:

| Tipo | Descripción | Impacto SemVer |
| :--- | :--- | :--- |
| **`feat`** | Una nueva funcionalidad para el usuario. | `MINOR` |
| **`fix`** | Una corrección de errores para el usuario. | `PATCH` |
| **`docs`** | Cambios solo en documentación. | Ninguno |
| **`style`** | Cambios de formato (espacios, comas) que no afectan lógica. | Ninguno |
| **`refactor`** | Cambio de código que no corrige errores ni añade funcionalidades. | Ninguno |
| **`perf`** | Cambios que mejoran el rendimiento. | Ninguno |
| **`test`** | Añadir o corregir pruebas existentes. | Ninguno |
| **`chore`** | Cambios en build, herramientas o configuración. | Ninguno |
| **`ci`** | Cambios en archivos de configuración de CI/CD. | Ninguno |

## Reglas de Formato y Sintaxis

### Encabezado (Header)

- **Ámbito (Scope)**: Opcional. Úsalo para indicar el módulo afectado (ej. `auth`, `api`, `ui`). Debe ir entre paréntesis.
- **Descripción**: Todo en minúsculas (salvo nombres propios), sin punto final.

### Cambios de Ruptura (Breaking Changes)

Para cambios que rompen la compatibilidad (`MAJOR`):

1. Añade un `!` después del tipo/ámbito: `feat(api)!: ...`
2. O incluye `BREAKING CHANGE:` en el pie de página.

## Ejemplos

### Good Examples

```text
// Commit simple de funcionalidad
feat(carrito): agrega botón para vaciar el carrito
````

```text
// Commit de corrección con cuerpo explicativo
fix(auth): valida tokens de sesión expirados

El sistema provocaba errores 500 en lugar de solicitar re-login.
Ahora utiliza el refresh token automáticamente.

Closes: #45
```

```text
// Breaking Change explícito
refactor!: cambia formato de respuesta API a JSONAPI

BREAKING CHANGE: La respuesta ahora envuelve los datos en un objeto "data".
```

### Bad Examples

```text
// Evita descripciones vagas o en pasado
arreglado el bug del login
```

```text
// Evita mezclar idiomas o formatos no estándar
[Feature] Added new button to navbar
```

```text
// Evita falta de tipo
actualiza dependencias del proyecto
```

## Validación

- **Linter**: Asegúrate de que el mensaje pase las reglas de `commitlint` si está configurado en el proyecto.
- **Check**: Verifica que el tipo coincida con la naturaleza real de los cambios en el código (`staged files`).