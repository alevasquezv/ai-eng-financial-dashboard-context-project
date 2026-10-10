# Fechas y periodos mostrados

**Nombre:** `dates-and-periods`
**Alcance:** cualquier código que lea, agrupe, compare o muestre fechas: `frontend/src/lib/financial-utils.ts`, `frontend/src/App.tsx`, `frontend/src/components/dashboard/`, y en el backend `backend/app/routes.py` (generación, `summarize_movements`, `/comparison`).
**Por qué:** el backend entrega `create_date` como fecha de calendario (`YYYY-MM-DD`, sin hora ni zona). Si el frontend la convierte con `new Date("YYYY-MM-DD")`, JavaScript la interpreta como medianoche **UTC**; al leerla con getters locales (`getMonth()`), en zonas al oeste de UTC cae en el día —o el mes— anterior. Medido con los datos reales de la API en `America/Mexico_City` (UTC-6): 16 de 360 movimientos cambian de mes y 11 de los 12 meses de las gráficas salen con totales incorrectos (en UTC: 0). Los KPIs no se afectan porque no agrupan por fecha.

## Reglas

### Frontend: parseo y agrupación
1. **Una fecha `YYYY-MM-DD` es una fecha de calendario, no un instante.** Para obtener año/mes/día lee el texto (`create_date.slice(0, 7)` para `YYYY-MM`, o `split("-")`), no pases por `Date`. Prohibido `new Date(create_date)` seguido de `getFullYear()/getMonth()/getDate()`.
2. **Si necesitas un `Date`, constrúyelo con componentes locales** (`new Date(year, month - 1, day)`) o usa `timeZone: "UTC"` de forma consistente al crear y al formatear. Nunca mezcles parseo UTC con getters locales.
3. **Las pruebas de fechas fijan la zona dentro de la prueba; no dependen de la máquina ni del CI.** Una prueba mensual solo detecta el bug en una máquina al oeste de UTC; en un CI con `TZ=UTC` pasaría sin el arreglo. Node aplica un cambio de `process.env.TZ` en caliente. Fíjala en el `describe` de las pruebas de fechas:
   ```ts
   /// <reference types="node" />   // 1.ª línea del archivo: sin ella `npm run build` falla con TS2591
   import { afterAll, beforeAll, describe } from "vitest";

   describe("computeMonthlyData", () => {
     const originalTZ = process.env.TZ;
     beforeAll(() => { process.env.TZ = "America/Mexico_City"; });
     afterAll(() => {
       if (originalTZ === undefined) delete process.env.TZ;
       else process.env.TZ = originalTZ;
     });
     // ...casos
   });
   ```
   El `reference` es necesario porque `tsconfig.app.json` solo carga los tipos de `vite/client` y los archivos de prueba entran en `tsc -b`; `@types/node` ya está instalado, no cambies la configuración ni las dependencias.

   Casos mínimos, con fixtures armados a mano (aquí los literales son correctos): **un cruce de mes** (último día de un mes + día 1 del siguiente, p. ej. `2026-01-31` y `2026-02-01`, comprobando que cada uno cae en su mes) y **un cruce de año** (`2025-12-31` y `2026-01-01`). Fíjate en que el último día por sí solo no delata el bug al oeste de UTC; el que falla es el día 1, por eso van emparejados.

### Frontend: periodo mostrado
4. **El periodo que muestra la UI sale de los datos cargados**, no de una etiqueta fija.
   - Calcúlalo con una función pura en `frontend/src/lib/financial-utils.ts` a partir de las fechas de los movimientos que ya tiene `App.tsx` (no hagas otra petición).
   - Formato: `Mmm YYYY - Mmm YYYY` (mes corto en inglés, guion simple con espacios); si el rango es un solo mes, `Mmm YYYY`.
   - La función devuelve `string | null`: `null` si no hay movimientos.
   - Mientras carga o si no hay datos, **no muestres la etiqueta**: nada de valor por defecto. `App.tsx` guarda el periodo junto al resto de datos derivados (`useState`), y `DashboardHeader` recibe `period?: string | null` y no renderiza la etiqueta si falta.
   - Cubre la función con pruebas (un mes, varios meses, cambio de año, lista vacía). La decisión «mostrar u ocultar» vive en esa función pura; no hace falta una prueba de componente para el header.

### Backend: pruebas con fechas
5. **No elijas ventanas con literales sobre los datos generados.** Los datos dependen de `date.today()`; deriva las ventanas de los datos (`min_date`/`max_date` de `/api/metrics/facets` o de `generate_mock_movements`), nunca de `"2025-03-01"`.
   - Una ventana relativa («últimos 30 días») cambia de contenido según el mes en que se ejecuta: añade una precondición (`assert`) de que tiene datos.
   - Si la prueba necesita una condición concreta (movimientos en los bordes, neto previo negativo para ejercitar un `abs()`), busca en los datos generados, con un helper, una ventana que la cumpla y **aserta la condición como precondición**. Prefiere ventanas cortas (≈ 14 días): una de 30 deja de cumplirla en varios meses de ejecución. Comprueba tu helper simulando varios valores de `date.today()` (p. ej. un mes de cada estación y un cambio de año).
   - Los datos generados solo tienen días 1-28 y su cambio de año depende de `today`. Los bordes (día 31, semana ISO en frontera de año, cambio de año) se prueban sobre las funciones puras (`summarize_movements`, `filter_movements_by_date`) con `FinancialMovement` construidos a mano: ahí los literales son deterministas y correctos.

## Guía específica de este repo

- Rango real de la API: los 12 meses de calendario que terminan en el mes anterior al actual (ejecutado en octubre de 2026 cubre oct-2025 a sep-2026), con 30 movimientos por mes y días 1-28.
- `App.tsx` pasa `period="2024 - Full Year"` fijo y `DashboardHeader` tiene otro valor por defecto (`'2024 — Full Year'`). Cuando una tarea toque el periodo, elimina ambos y haz la etiqueta opcional (regla 4); si la tarea es otra, no los toques y anótalo en tu informe.
- `frontend/src/lib/mock-data.ts` contiene datos de 2024 y no lo importa nadie: no lo uses como fuente de verdad.
- `formatMonthYearLabel` usa `new Date(year, month, 1)` con componentes locales: es correcto, no lo cambies a parseo de cadena ISO.

## Verificación

```bash
cd frontend && npm test            # la zona queda fijada dentro de la prueba (regla 3)
cd backend  && python -m pytest -q
```

## Origen

Reglas 4-5 de la fase 2 en [engineering-conventions.md](./engineering-conventions.md). Refinada tras validarla con la tarea «corregir los totales mensuales en `America/Mexico_City` y mostrar el periodo real».
