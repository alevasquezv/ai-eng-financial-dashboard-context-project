# Pruebas: qué probar y dónde

**Nombre:** `testing`
**Alcance:** todo cambio en `backend/` o `frontend/src/`, y las tareas cuyo único objetivo son las pruebas.
**Por qué:** hay dos suites pequeñas y sanas (pytest en el backend, vitest en el frontend) pero con huecos conocidos: no hay pruebas de componentes React, vitest no ve los tipos de TypeScript, las pruebas de fechas dependían de la zona horaria de la máquina y había pruebas débiles que solo comprobaban «que haya datos» o «que existan las claves».

## Reglas

1. **Prueba cada regla de negocio en la capa donde vive.**

   | Qué cambias | Dónde va la prueba | Archivo |
   | --- | --- | --- |
   | Función pura del backend (filtro, agregación, alerta) | Unitaria, sin HTTP | `backend/tests/test_routes.py` |
   | Contrato de un endpoint (campos, filtros, orden) o lógica que vive en un handler | `TestClient` sobre la ruta | `backend/tests/test_routes.py` |
   | Configuración leída al importar (p. ej. CORS) | Dos pruebas: la función que interpreta la configuración (con `monkeypatch`) y el cableado real con `TestClient`. Como `app` se crea al importar, `backend/tests/conftest.py` limpia la variable (`os.environ.pop(...)`) **antes** de importarla, para que el resultado no dependa del shell. | `backend/tests/` |
   | Cálculo o formato del frontend | Unitaria | `frontend/src/lib/financial-utils.test.ts` (o `<módulo>.test.ts` junto al módulo) |
   | Decisión de un componente (mostrar/ocultar, formatear) | Extráela a una función pura en `lib/` y pruébala allí; el componente solo la renderiza | `frontend/src/lib/*.test.ts` |
   | Componente React con estado o interacción propia | Prueba de componente solo con aprobación (regla 4) | junto al componente |

2. **Cada tipo de cambio tiene su comprobación, y debe poder fallar sin el cambio:**

   | Tipo de cambio | Cómo lo compruebas |
   | --- | --- |
   | Comportamiento (arreglo o función nueva) | Prueba nueva que **falla sin el cambio**. Escríbela primero. |
   | Solo tipos espejo del frontend | Vitest borra los tipos y no puede verlos. `npm run build` (`tsc -b`) solo falla sin el cambio **si algún código del frontend usa el valor o el campo nuevo**; si nada lo usa, el build pasa con o sin el tipo y el control real es comparar a mano las dos definiciones (backend y frontend). No añadas una prueba solo por tener una. |
   | Tarea solo de pruebas (reforzar una prueba débil) | Cubre lo que promete el nombre de cada prueba (si hace falta una prueba nueva para eso, añádela; los demás huecos que veas, anótalos). Comprueba con **mutaciones**: rompe temporalmente la lógica, una mutación por cada cosa que la prueba afirma (invierte un filtro, cambia un signo, quita un `abs()`), confirma que la prueba falla, y **restaura**: `git checkout -- <archivo fuente>` (o una copia de respaldo si ya tenías cambios en él) y confirma con `git diff` que solo quedan tus cambios. |
   | Refactor sin cambio de comportamiento | No exige prueba nueva: `npm run lint`, `npm run build` y las pruebas existentes deben seguir verdes. |
   | Solo documentación | No exige pruebas. Verifica los comandos que documentas (ver [docs-and-commits.md](./docs-and-commits.md)). |

3. **Un cambio de comportamiento lleva su prueba en el mismo commit** (si se commitea; ver [docs-and-commits.md](./docs-and-commits.md)).
4. **No añadas pruebas de componentes por defecto.** Hoy no hay `jsdom`/Testing Library instalados y vitest corre en entorno `node`. Si una tarea exige probar un componente, dilo y pide aprobación antes de añadir dependencias.
5. **Pruebas deterministas.**
   - Las fechas siguen [dates-and-periods.md](./dates-and-periods.md): sin ventanas literales sobre datos generados; fixtures armados a mano sí pueden llevar fechas literales. Un fixture de una prueba que no trata de fechas puede usar cualquier fecha válida.
   - No uses `if payload:` para saltarte aserciones cuando el caso podría quedar vacío; asegúralo con una precondición (`assert payload`).
   - Una prueba que depende de variables de entorno las fija y las limpia ella misma.
6. **Antes de dar por terminada una tarea de código, corre las cuatro comprobaciones** y reporta los resultados reales (si algo falla o no se pudo correr, dilo):
   ```bash
   cd backend  && python -m pytest -q
   cd frontend && npm test
   cd frontend && npm run lint
   cd frontend && npm run build
   ```
   Córrelas también **antes** de empezar: ese es tu estado base.

## Guía específica de este repo

- Estado base verificado el 2026-10-09: las cuatro comprobaciones en verde (el build avisa de un chunk >500 kB: es informativo, no un fallo).
- **Python ≥ 3.10** (los modelos usan `float | None`); el Dockerfile usa 3.13. En 3.9 la colección de pruebas falla con `Unable to evaluate type annotation` salvo que el entorno tenga `eval_type_backport`, que `requirements.txt` no incluye: no lo añadas al proyecto para sortearlo.
- **Node `^20.19.0 || >=22.12.0`** (requisito de Vite 8); el Dockerfile usa Node 24.
- **Comandos canónicos** (los README los copian tal cual):
  - Sin Docker, instalar: `python3 -m venv backend/.venv && backend/.venv/bin/pip install -r backend/requirements.txt` y `cd frontend && npm ci`.
  - Sin Docker, probar: `cd backend && .venv/bin/python -m pytest -q` y, en `frontend/`, `npm test`, `npm run lint`, `npm run build`.
  - Con Docker: `docker compose exec backend python -m pytest -q`.
- Un pytest verde con un entorno que no se creó desde `requirements.txt` **no valida** las versiones fijadas en ese archivo.

## Origen

Regla 6 de la fase 2 en [engineering-conventions.md](./engineering-conventions.md). Refinada tras validarla con la tarea «reforzar dos pruebas débiles del backend» y con las pruebas de las demás tareas.
