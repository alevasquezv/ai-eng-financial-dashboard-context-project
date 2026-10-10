# Verificación de la fase 1

## Alcance

Revisión estática de la estructura, los entry points, la comunicación frontend-backend y la forma en que se generan los datos. No se modificó código funcional ni se inició la aplicación.

## Evidencia

| Punto revisado | Resultado | Evidencia |
| --- | --- | --- |
| Stack y arranque | Confirmado: frontend React + TypeScript con Vite; API FastAPI/Uvicorn. Docker Compose levanta ambos servicios; los Dockerfiles definen sus comandos de desarrollo. | `frontend/package.json`, `frontend/Dockerfile`, `backend/requirements.txt`, `backend/Dockerfile`, `docker-compose.yml` |
| Entry point del frontend | Confirmado: `frontend/index.html` carga `frontend/src/main.tsx`, que monta la aplicación de `frontend/src/App.tsx`. | `frontend/index.html`, `frontend/src/main.tsx`, `frontend/src/App.tsx` |
| Entry point del backend | Confirmado: `backend/app/main.py` crea la aplicación FastAPI e incorpora el router definido en `backend/app/routes.py`. | `backend/app/main.py`, `backend/app/routes.py` |
| Conexión entre servicios | Confirmado: la pantalla solicita `GET /api/metrics`; Vite envía `/api` a `http://backend:8000` en Docker Compose. `VITE_API_BASE_URL` permite configurar otro origen. | `frontend/src/App.tsx`, `frontend/vite.config.ts`, `frontend/.env.example`, `docker-compose.yml` |
| Datos y persistencia | Confirmado: la API genera 360 movimientos sintéticos (30 por cada mes) con semilla `42`. No se encontró una base de datos ni una capa de persistencia en las dependencias o el flujo revisado. | `backend/app/routes.py`, `backend/requirements.txt` |
| Rango temporal mostrado | Discrepancia: la generación toma los últimos 12 meses de calendario según la fecha actual; para octubre de 2026, los movimientos cubren oct-2025 a sep-2026. La cabecera, en cambio, muestra fijo `2024 - Full Year`. | `backend/app/routes.py`, `frontend/src/App.tsx` |
| Endpoints disponibles vs. consumidos | La API expone salud, movimientos, facetas, resumen, categorías principales, comparación, alertas y filtros B2B/B2C. La pantalla revisada consume únicamente `/api/metrics` y calcula KPIs y agregados mensuales en el frontend. | `backend/app/routes.py`, `frontend/src/App.tsx`, `frontend/src/lib/financial-utils.ts` |
| Ejecución documentada | Confirmado: `docker compose up --build`; frontend en `localhost:5173`, API en `localhost:8000` y documentación interactiva en `localhost:8000/docs`. | `README.md`, `README.es.md`, `docker-compose.yml` |

## Conclusión

El repositorio es una base didáctica de dashboard financiero: React presenta KPIs y gráficos, mientras FastAPI entrega datos de ejemplo reproducibles. La arquitectura está conectada y lista para desarrollo local con Docker Compose, pero no representa información financiera real ni persiste datos.

## Límites de la verificación

- La revisión fue estática: no se ejecutaron contenedores, pruebas, build ni lint.
- Conviene alinear la etiqueta fija `2024 - Full Year` con el rango que realmente devuelve la API antes de presentar el dashboard como un periodo financiero concreto.
