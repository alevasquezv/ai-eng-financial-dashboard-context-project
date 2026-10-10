# Overview del producto

**Última verificación:** 2026-10-09. Cada afirmación lleva su fuente (archivo, comando o medición). Lo que el repo no respalda está en la sección «Lo que NO se sabe».

## Qué es

Un dashboard de métricas financieras con frontend React + TypeScript y backend FastAPI. Es un proyecto didáctico: el README lo presenta dentro de los programas de carrera de 4Geeks Academy y pide, como pasos recomendados, que el estudiante inspeccione el código con un agente y documente reglas y memoria (`README.md`, «Recommended steps»).

- **Origen:** el historial empieza el 2026-04-13. El frontend se creó con V0 (commit `7661fa1`, «Frontend from V0 with components and types») y el backend se añadió el mismo día (`15ce4c2`, «Initial backend»). `git log`: 3 autores.
- **Naturaleza de los datos:** sintéticos. `generate_mock_movements(seed=42)` en `backend/app/routes.py` genera 360 movimientos (30 por mes) en cada petición. No hay base de datos ni capa de persistencia (`backend/requirements.txt` no incluye ninguna).

## Qué hace hoy

Una sola pantalla, «Financial Overview» (`frontend/src/App.tsx`), con:

| Elemento | Qué muestra | Fuente |
| --- | --- | --- |
| 4 tarjetas KPI | Total Income, Total Outcome, Profit, Profit Margin | `components/dashboard/kpi-row.tsx`, cálculo en `lib/financial-utils.ts` (`computeKPIs`) |
| Gráfica «Income vs. Outcome» | Líneas mensuales de ingresos y gastos | `components/dashboard/income-outcome-chart.tsx` |
| Gráfica «Profit Margin %» | Margen mensual en % con línea de referencia en 0 | `components/dashboard/profit-percent-chart.tsx` |
| Estados de carga y error | Skeletons mientras carga; banner si la API falla | `App.tsx`, `kpi-card.tsx` |

**Verificado el 2026-10-09** arrancando backend y frontend a mano y mirando el navegador: con la API accesible, los KPIs coinciden exactamente con los totales calculados desde `GET /api/metrics` (ingresos $1,258,147; gastos $762,132; beneficio $496,015; margen 39.4%). Con la API inaccesible, la pantalla muestra el banner de error y los KPIs en «—».

## Modelo de datos

Un `FinancialMovement` tiene: `create_date` (fecha `YYYY-MM-DD`), `amount`, `operation_type`, `category` y `business_type`. Los valores permitidos viven en los `Literal` de `backend/app/routes.py` y en `frontend/src/lib/financial-types.ts` (se definen dos veces; ver `.agents/rules/api-and-contracts.md`). No los copies aquí: se desactualizan.

## API

El backend expone 9 rutas GET (`backend/app/routes.py`): `/health`, `/api/metrics`, `/api/metrics/facets`, `/summary`, `/categories/top`, `/comparison`, `/alerts`, `/b2b` y `/b2c` (estas con prefijo `/api/metrics/`). **La interfaz solo consume `/api/metrics`** (`grep -rn "/api/" frontend/src` devuelve únicamente `App.tsx:16`); el resto existe, está probado en `backend/tests/test_routes.py` y no tiene consumidor en la UI. Documentación interactiva de FastAPI en `/docs` (`README.md`).

## Lo que el producto NO es (con evidencia)

- **No usa datos reales:** todo sale del generador con semilla fija.
- **No persiste nada:** sin base de datos; los datos se regeneran por petición.
- **No tiene autenticación ni usuarios:** no hay código de auth en backend ni frontend; el CORS está abierto a cualquier origen (ver `current-state.md`, G4).
- **No tiene filtros, selector de periodo ni navegación en la UI:** una sola pantalla sin controles (`App.tsx`).

## Lo que NO se sabe (no inventar)

El repo no documenta ni permite inferir, así que **no debe afirmarse**:

- quiénes son los usuarios objetivo ni qué decisiones debería habilitar el dashboard;
- objetivos de negocio, métricas de éxito o plazos;
- dónde o cómo se desplegaría (no hay configuración de despliegue; solo Docker Compose para desarrollo);
- qué funcionalidades vienen después. **No existe un roadmap de producto en el repo.** Las prioridades de `current-state.md` ordenan los problemas verificados; no son un plan de producto.
