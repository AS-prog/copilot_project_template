---
description: 'Guía a Copilot para generar y refactorizar títulos de Pull Requests (PRs) y mensajes de commit que siguen el estándar Conventional Commits. Aplica las reglas de atomicidad y la plantilla de descripción obligatoria.'
applyTo: '**'
---

# 🚀 Estándares de Pull Request (PR) y Conventional Commits

Este archivo instruye a GitHub Copilot a adherirse estrictamente a los estándares de **Conventional Commits** para títulos de PRs y a utilizar la plantilla de descripción obligatoria del proyecto. El objetivo es mantener un historial de versiones limpio, atómico y semánticamente correcto.

## 1. 🎯 Principios de Commit y PR

Copilot debe considerar los siguientes principios al sugerir nuevos commits o resúmenes de PR:

* **Atomicidad**: Cada commit y PR debe centrarse en **una única tarea o concepto**. Evitar mezclar `feat`, `fix`, y `refactor`.
* **Pruebas Requeridas**: Si se modifica lógica de negocio (`feat` o `fix`), siempre debe sugerir la inclusión de un archivo de prueba o una mención a las pruebas en el `Checklist de Autor`.
* **Uso de la Plantilla**: Al generar una descripción de PR, debe generar la plantilla de descripción completa (sección 3) con los campos `Resumen del Cambio` y `Cómo Probar Manualmente` vacíos para que el usuario los complete.

## 2. 📝 Nomenclatura Estándar (Conventional Commits)

El título del commit o PR debe seguir la estructura: `<tipo>[*(ámbito)*]: <descripción>`

### Tipos de Uso Obligatorio

Copilot **debe usar** solo los siguientes tipos y entender su implicación en el versionado semántico (SemVer):

| Tipo | Propósito | Implicación SemVer |
| :--- | :--- | :--- |
| **`feat`** | Nueva funcionalidad (visible para el usuario). | `MINOR` |
| **`fix`** | Corrección de un error (visible para el usuario). | `PATCH` |
| `refactor` | Reestructuración de código (mejora interna, sin cambio de comportamiento). | Ninguna |
| `docs` | Cambios solo en la documentación o archivos `README`. | Ninguna |
| `chore` | Tareas de mantenimiento, configuración o gestión de dependencias menores. | Ninguna |

### 🚨 Guía de Cambio de Ruptura (`BREAKING CHANGE`)

Si el cambio introducido es un `MAJOR` (rompe la compatibilidad), Copilot **debe** sugerir el uso del signo de exclamación (`!`) inmediatamente después del tipo.

* **Sugerencia Correcta**: `feat(api)!: Implementación de nueva autenticación que requiere migración`
* **Sugerencia Incorrecta**: `feat(api): Cambio de autenticación (breaking change)`

## 3. 📄 Plantilla de Descripción de la PR (Obligatoria)

Cuando se pida generar el *cuerpo* de una Pull Request, Copilot debe utilizar **esta plantilla en su totalidad**.

```markdown
# [Tipo] | Título Conciso de la Pull Request

---

## 🎯 Resumen del Cambio

[Tu resumen aquí...]

### 🔗 Issues Relacionados

Cierra: 
Relacionado con:

---

## 🔎 Detalles Técnicos

[Detalles aquí...]

### Componentes Modificados:

* [ ] Lógica de negocio
* [ ] Capa de persistencia (BD)
* [ ] API / Endpoints
* [ ] Configuración / CI
* [ ] Documentación

---

## 🧪 Cómo Probar Manualmente

1. Hacer checkout de esta rama: `git checkout <nombre-de-tu-rama>`
2. Ejecutar...
3. Acceder a [URL] y verificar...
4. [Paso 4]

---

## ✅ Checklist de Autor (Antes de pedir revisión)

- [ ] El título sigue el estándar de Conventional Commits.
- [ ] La plantilla de descripción ha sido completada en su totalidad.
- [ ] El código está limpio y ha pasado el linter/formateador.
- [ ] Se han añadido o actualizado pruebas que cubren el cambio.
- [ ] Se ha realizado una **autorevisión** de todo el código modificado.
````

## 4\. Ejemplos de Interacción Correcta

### Buen Ejemplo: Título de Feature

Si el usuario está implementando la gestión de permisos de usuario.

```text
feat(auth): Añade endpoint y lógica para la gestión de roles de usuario
```

### Mal Ejemplo: Título y Commit Mezclado

Si el usuario está corrigiendo un error y, a la vez, añadiendo una nueva función.

```text
fix: Corrige error en el login y añade campo de teléfono al formulario
```

> **Instrucción para Copilot:** Sugiere al usuario dividir este cambio en dos PRs separadas: un `fix` para el error de login y un `feat` para el campo de teléfono.

### Buen Ejemplo: Título de Refactor

Si el usuario está reorganizando un archivo sin cambiar el resultado.

```text
refactor(clientes): Extrae lógica de formato de fecha a función de utilidad
```

-----

## 5\. Validación de Instrucciones

Cuando Copilot detecte un fragmento de código o un mensaje de commit que viole las reglas de atomicidad o nomenclatura, debe:

1. Señalar la violación.
2. Ofrecer una refactorización del mensaje que cumpla con el estándar Conventional Commits.
3. Si la descripción no es atómica (ej. mezcla `fix` y `feat`), sugerir la división del trabajo.
