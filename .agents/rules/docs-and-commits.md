# Documentación, commits y flujo de agentes

**Nombre:** `docs-and-commits`
**Alcance:** `README.md`, `README.es.md`, `AGENTS.md`, `.agents/`, `memory-bank/`, `verification.md`, y cada commit que crees.
**Por qué:** el repo se usa también como material de curso y lo retoman otros agentes y contribuidores; la documentación desactualizada y los commits mezclados son lo que más cuesta recuperar.

## Reglas

### Antes de empezar
1. Lee `AGENTS.md`, todos los archivos de `.agents/rules/`, los de `.agents/skills/` (hoy esa carpeta no existe) y `memory-bank/` si existe. Verifica lo que digan contra el código y el estado de Git antes de apoyarte en ello.
2. **Las reglas son vinculantes.** `engineering-conventions.md` es el análisis de origen (fase 2): si algo difiere de los demás archivos de esta carpeta, prevalecen estos. En un conflicto entre reglas aplica, en orden: la petición explícita del usuario; la regla más específica sobre la general; y «un cambio, un propósito» sobre «arregla lo que veas» (lo que veas fuera de alcance, anótalo en tu informe).

### Documentación
3. **Si cambias cómo se ejecuta el proyecto** (comandos, puertos, proxy `/api`, variables de entorno, versiones requeridas), actualiza **a la vez** `README.md` y `README.es.md`. `frontend/.env.example` es solo para variables del frontend; las variables del backend que existan en el código (p. ej. `CORS_ORIGINS`, cuando se introduzca) se documentan en los README, porque no hay plantilla `.env` del backend. No documentes variables que el código aún no lee.
4. **Estructura de los README** (cuando la tarea es de documentación; en cualquier otra tarea documenta solo lo que tú cambiaste, no amplíes los README con requisitos o pruebas): tras «How to run locally» / «Cómo ejecutar en local», dos secciones con estos títulos: `## Requirements` / `## Requisitos` (Python ≥ 3.10, Node `^20.19.0 || >=22.12.0`, y una línea para comprobar versiones con `python3 --version` y `node --version`) y `## Running the tests` / `## Cómo correr las pruebas`. Avisa en Requisitos de que con Python 3.9 la colección de pruebas del backend falla. **Copia los comandos tal cual de [testing.md](./testing.md)**: ahí está su forma canónica; no inventes variantes.
5. **Comprueba los comandos que documentas.** Ejecuta los que puedas. Los que no puedas ejecutar en tu entorno (Docker, `pip install`/`npm ci` sin permiso de instalar) se documentan solo si son el procedimiento estándar de `testing.md`, y se marcan como **«Not verified»** en el cuerpo del commit (en inglés, como el resto del historial) o en tu informe: ni los omitas ni los presentes como probados.
6. **Notas de traspaso (`memory-bank/`):** refleja el estado real (qué está hecho, qué falta, qué bloqueos siguen vigentes) y elimina las notas que dejen de ser ciertas. `verification.md` es el registro de la fase 1: no lo reescribas. Una verificación nueva va en un archivo nuevo solo si la tarea es una fase de verificación; en una tarea normal basta el cuerpo del commit o tu informe.
7. **Escribe los hechos con su evidencia** (comando ejecutado, archivo y función), no con frases generales. Prefiere nombrar la función o el archivo antes que un número de línea, que caduca.

### Commits
8. **Cuándo:** no commitees salvo que la tarea o el usuario lo pidan. Si no lo piden, deja los cambios sin stagear y repórtalos.
9. **Formato del mensaje:** `tipo: resumen en imperativo` con tipo en {`feat`, `fix`, `docs`, `test`, `refactor`, `chore`}, resumen ≤ 72 caracteres, en inglés como el historial reciente (`docs: add phase 1 verification`). Añade cuerpo solo si explica el porqué o lista lo «Not verified» (regla 5). Si tu herramienta exige una línea `Co-Authored-By`, va al final y no cuenta como cuerpo.
10. **Un commit, un propósito.** Un propósito es una intención que puedes describir en una frase. Documentación, arreglos de código, cambios de dependencias y reformateos son propósitos distintos. Si dos propósitos tocan los mismos archivos y no puedes separarlos sin herramientas interactivas (`git add -p`), haz un solo commit y lista cada propósito en el cuerpo. El código de una intención y su documentación mínima pueden ir juntos.
11. **Stagea archivos por nombre** (`git add <ruta>`), nunca `git add -A` ni `git add .`: el árbol suele tener archivos ajenos sin trackear (p. ej. `memory-bank/`).
12. **Antes de commitear:** `git status --short` y `git diff --staged --stat` deben mostrar solo lo que pretendes. No commitees `.env`, `backend/.venv/`, `frontend/node_modules/` ni `frontend/dist/` (ya están en `.gitignore`).
13. **No hagas `push`, `amend` ni `rebase` de commits existentes** sin que el usuario lo pida.

## Verificación

```bash
git status --short && git diff --staged --stat
git log --oneline -5      # el mensaje nuevo sigue el formato de la regla 9
```

## Origen

Regla 10 de la fase 2 en [engineering-conventions.md](./engineering-conventions.md); las reglas de commits se derivan del historial (`git log`) y del flujo de las fases 1-2. Refinada tras validarla con la tarea «documentar cómo correr las pruebas y la versión mínima de Python» y con los commits de las demás tareas.
