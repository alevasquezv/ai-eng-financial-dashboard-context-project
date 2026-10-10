# Estado actual

**Última verificación:** 2026-10-09, en macOS con Python 3.9.6 (más `eval_type_backport` solo en el entorno de verificación), Node 22.23.1 y **sin Docker**. Todo lo marcado como verificado se ejecutó en ese entorno.

## Qué funciona (verificado)

| Qué | Cómo se verificó | Resultado |
| --- | --- | --- |
| Pruebas del backend | `cd backend && python -m pytest -q` | 15 pasan |
| Pruebas del frontend | `cd frontend && npm test` | 5 pasan |
| Lint y build del frontend | `npm run lint`, `npm run build` | sin avisos / compila (aviso informativo de chunk > 500 kB) |
| API en marcha | `uvicorn app.main:app --port 8000` y `curl` | `GET /health` → 200; `GET /api/metrics` → 360 movimientos entre 2025-10-02 y 2026-09-28 |
| Dashboard con datos | Vite con `VITE_API_BASE_URL=http://127.0.0.1:8000` y navegador | KPIs idénticos a los totales de la API; las dos gráficas se dibujan |
| Banner de error | Navegador con la API inaccesible | muestra el mensaje de error y KPIs en «—» |
| Reglas de agentes | `verification-phase-3.md` | 6 archivos validados con tareas reales en dos rondas |

## Qué NO se pudo verificar

- **Docker Compose de extremo a extremo** (no hay Docker): ni el build de las imágenes, ni `npm ci` en imagen, ni el proxy dentro de Compose.
- Que el backend funcione con Python ≥ 3.10 real y con las versiones actuales de FastAPI (las pruebas corrieron con 0.128.8 sobre Python 3.9).

## Brechas conocidas (con evidencia)

| ID | Brecha | Evidencia | Regla relacionada |
| --- | --- | --- | --- |
| G1 | **Las gráficas mensuales agrupan mal en zonas al oeste de UTC.** `computeMonthlyData` hace `new Date("YYYY-MM-DD")` y lee el mes con getters locales; en `America/Mexico_City`, 16 de 360 movimientos cambian de mes y 11 de 12 meses salen con totales incorrectos. Los KPIs no se ven afectados. | Medido con los datos reales de la API replicando la lógica en Node con `TZ` distinto; `frontend/src/lib/financial-utils.ts` | `dates-and-periods.md` |
| G2 | **El periodo mostrado no coincide con los datos.** La cabecera dice «2024 - Full Year» (`App.tsx`; `dashboard-header.tsx` tiene otro valor por defecto) y las gráficas muestran oct-2025 a sep-2026. | Visible en pantalla al verificar; `frontend/src/lib/mock-data.ts` (datos de 2024) no lo importa nadie | `dates-and-periods.md` |
| G3 | **Fuera de Docker el proxy de Vite falla.** `vite.config.ts` apunta a `http://backend:8000`, un hostname que solo existe en Compose; sin `VITE_API_BASE_URL` el `curl` a `:5173/api/metrics` da 502 (`ENOTFOUND backend`) y la UI muestra el banner de error. El README dice que no hacen falta variables de entorno, lo cual solo se cumple con Docker Compose. | Medido el 2026-10-09 | `docs-and-commits.md` |
| G4 | **CORS abierto con credenciales:** `allow_origins=["*"]` + `allow_credentials=True`; el backend refleja cualquier `Origin`. Hoy no hay cookies ni login, así que no expone nada real. | Probado con `TestClient`; `backend/app/main.py` | `security-and-dependencies.md` |
| G5 | **Puerto de depuración publicado:** `debugpy` escucha en `0.0.0.0:5678` y Compose lo mapea. | `backend/Dockerfile`, `docker-compose.yml` | `security-and-dependencies.md` |
| G6 | **Instalaciones no reproducibles:** `requirements.txt` sin versiones, `npm install` en el Dockerfile del frontend, 13 vulnerabilidades en el lockfile (1 baja, 5 moderadas, 7 altas). | `npm ci`, archivos citados | `security-and-dependencies.md` |
| G7 | **Pruebas con huecos:** sin pruebas de componentes; `test_metrics_comparison_returns_delta_fields` usa fechas literales de 2025 que quedan fuera del rango generado y solo comprueba las claves; el test semanal por `business_type` solo comprueba que haya datos; `test_metrics_alerts_returns_anomaly_candidates` usa `if payload:`. | `backend/tests/test_routes.py` | `testing.md` |
| G8 | **Pulido menor:** `<title>` de la página es «frontend»; el mensaje de error está en español y sin tilde («informacion»); el banner no tiene `role="alert"`. | `frontend/index.html`, `App.tsx` | `naming-and-structure.md` |

**Hipótesis no confirmada (no tratar como hecho):** `generate_mock_movements` llama a `random.seed(42)` sobre el generador global; peticiones concurrentes podrían intercalar números aleatorios y devolver datos distintos. En una réplica aislada con el intervalo de cambio de hilo por defecto no se reprodujo (0 de 1600 divergencias); con un intervalo artificialmente corto, sí (1600 de 1600). No se probó contra el servidor real.

## Prioridades derivadas de las brechas

Estas prioridades **ordenan los problemas verificados**; no son un roadmap de producto (el repo no define ninguno). Coinciden con la «Prioridad sugerida» de `.agents/rules/engineering-conventions.md`.

1. **G1 y G2: la interpretación de las gráficas.** Son los únicos fallos que hacen que el dashboard muestre algo incorrecto o engañoso. Las reglas de `dates-and-periods.md` ya describen cómo arreglarlos y probarlos.
2. **G3: la experiencia fuera de Docker.** Es un fallo de arranque; basta corregir la documentación o el proxy.
3. **G4, G5 y G6: antes de usar el proyecto fuera de un entorno local.** Hoy el riesgo es bajo porque no hay datos reales; las reglas de `security-and-dependencies.md` fijan el criterio.
4. **G7 y G8: robustez y pulido.** No cambian lo que ve el usuario hoy.

Cualquier funcionalidad nueva (filtros, consumir los endpoints sin uso, etc.) requiere antes una decisión del dueño del proyecto sobre objetivos de producto, que hoy no están documentados.

## Cómo se llegó aquí (commits de las fases del trabajo)

| Fase | Commit | Contenido |
| --- | --- | --- |
| 1 | `dd5a632` | `verification.md`: mapa del repo verificado contra el código |
| 2 | `547ff2f` | `.agents/rules/engineering-conventions.md`: hallazgos y diez reglas propuestas |
| 3 | `ec39987` | seis archivos de reglas validados + `verification-phase-3.md` |
| 4 | este commit | `memory-bank/` |
