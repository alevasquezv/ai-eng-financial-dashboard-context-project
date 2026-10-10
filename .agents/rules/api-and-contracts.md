# API y contratos frontend/backend

**Nombre:** `api-and-contracts`
**Alcance:** todo cambio en `backend/app/` (rutas, modelos, generación de datos) y en `frontend/src/lib/financial-types.ts` o en cualquier código que consuma `/api/*`.
**Por qué:** el contrato se define dos veces (Pydantic en Python, interfaces en TypeScript) y nada lo sincroniza automáticamente; un campo o valor nuevo en un lado rompe el otro sin avisar. Además `routes.py` mezcla modelos, generación, filtros y handlers, así que sin disciplina crece y se vuelve difícil de probar.

## Reglas

1. **Handlers delgados.** Una ruta solo declara parámetros (`Query`), llama a funciones puras y devuelve el resultado. La lógica nueva (filtros, agregaciones, cálculos) va en una función pura con tipos, sin `Query` ni objetos HTTP, para probarla sin `TestClient`.
2. **Extrae cuando crezca, y en un cambio aparte.**
   - Si `wc -l backend/app/routes.py` supera ~400 líneas, o añades más de ~15 líneas de lógica nueva, mueve las funciones puras a un módulo hermano (p. ej. `backend/app/services.py`) y deja en `routes.py` solo los endpoints.
   - No hagas esa extracción como efecto secundario de otra tarea.
   - Mientras tanto, la lógica que ya vive dentro de un handler (p. ej. las ventanas de `/api/metrics/comparison`) se prueba a través de `TestClient`.
3. **Todo endpoint declara su contrato.** Usa `response_model=` con un modelo Pydantic y tipos `Literal` para valores cerrados (`OperationType`, `Category`, `BusinessType`, `GroupBy`). No devuelvas `dict` sueltos salvo `/health`.
4. **Un cambio de contrato se hace completo, en el mismo commit.** Se considera cambio de contrato: añadir/quitar/renombrar un campo, añadir/quitar un valor de un `Literal`, o cambiar el formato de un campo (p. ej. fecha). Sitios a tocar:
   - el modelo o `Literal` en `backend/app/routes.py`;
   - si es una **categoría de gasto**, también `OUTCOME_CATEGORIES` (si no, el generador nunca la produce);
   - el tipo espejo en `frontend/src/lib/financial-types.ts`;
   - las pruebas (ver [testing.md](./testing.md)): actualiza a mano la lista de categorías del test de facets (es el único sitio donde el contrato se fija a propósito con un literal) y añade dos pruebas para un valor nuevo: el generador lo produce solo con el `operation_type` que corresponde, y `GET /api/metrics?category=<valor>` devuelve solo ese valor. Para un campo nuevo, una prueba que compruebe que la respuesta lo incluye.
5. **Al añadir un valor nuevo a una lista:**
   - Ponlo **al final** de las tres listas: el `Literal` de Python, `OUTCOME_CATEGORIES` y la unión del tipo TypeScript. En `OUTCOME_CATEGORIES` cualquier posición cambia los datos que produce la semilla 42, así que, salvo la lista de facets, las expectativas de las pruebas se derivan de los datos generados, no de valores memorizados.
   - Usa los rangos de importe que ya tiene `_build_movement` para ese tipo de operación, salvo que la tarea diga otra cosa. Una categoría de gasto se genera solo para `outcome`.
6. **Los parámetros de filtro nuevos son opcionales** (`= Query(default=None)`) para no romper a los clientes existentes, y se aplican con las funciones existentes (`filter_movements`, `filter_movements_by_date`) en lugar de reimplementar el filtrado.

## Guía específica de este repo

- Los datos son sintéticos: `generate_mock_movements(seed=42)` se llama en cada petición, no hay base de datos. No asumas persistencia ni IDs.
- Los valores permitidos viven en los `Literal` de `routes.py` y en los tipos de `financial-types.ts`. **No los copies en documentación ni en comentarios**: se desactualizan en cuanto cambia el contrato.
- `FinancialMovement` (Python) y `FinancialMovement` (TypeScript) deben tener los mismos campos.
- Consumidores: la UI solo llama a `GET /api/metrics`; el resto de endpoints existe sin consumidor. Compruébalo con `grep -rn "/api/" frontend/src` antes de asumir quién usa un endpoint.

## Verificación

```bash
cd backend && python -m pytest -q          # contrato HTTP y funciones puras
cd frontend && npm test && npm run build   # tsc -b detecta tipos desalineados
```

## Origen

Hallazgos y reglas 1-3 de la fase 2 en [engineering-conventions.md](./engineering-conventions.md). Refinada tras validarla con la tarea «añadir la categoría `payroll`».
