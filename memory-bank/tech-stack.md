# Stack tecnológico

**Última verificación:** 2026-10-09. «Declarado» = lo que dice el manifiesto del repo; «resuelto» = lo que se instaló al verificar. Para el backend, `requirements.txt` no fija versiones, así que no hay una versión «oficial».

## Lenguajes y runtimes

| Capa | Lenguaje | Evidencia |
| --- | --- | --- |
| Frontend | TypeScript 6.0.2 (`~6.0.2`) sobre React | `frontend/package.json`, `frontend/node_modules` |
| Backend | Python. Imagen Docker `python:3.13-slim`; el código exige **≥ 3.10** (usa `float \| None` en modelos Pydantic) | `backend/Dockerfile`; con Python 3.9 la colección de pruebas falla con `Unable to evaluate type annotation` (medido) |
| Node | `^20.19.0 \|\| >=22.12.0` (requisito de Vite 8); imagen Docker `node:24-alpine` | `frontend/node_modules/vite/package.json`, `frontend/Dockerfile` |

## Frontend (`frontend/`)

| Pieza | Declarado → resuelto | Notas |
| --- | --- | --- |
| React / React DOM | `^19.2.4` → 19.2.5 | punto de entrada `src/main.tsx` → `src/App.tsx` |
| Vite | `^8.0.4` → 8.0.8 | servidor de desarrollo en el puerto 5173; proxy `/api` → `http://backend:8000` (`vite.config.ts`) |
| Tailwind CSS | `^4.2.2` → 4.2.2 | plugin `@tailwindcss/vite`; tokens de tema en `src/index.css`; el modo oscuro se fuerza con la clase `dark` en `<main>` de `App.tsx` |
| Recharts | `^3.8.1` → 3.8.1 | solo `LineChart` en las dos gráficas |
| shadcn/ui estilo «new-york» | `components.json` | primitivas `card` y `skeleton` en `src/components/ui/`; alias `@` → `src` |
| lucide-react | `^1.8.0` → 1.8.0 | iconos de las tarjetas KPI |
| clsx, tailwind-merge, class-variance-authority | ver `package.json` | utilidad `cn` en `src/lib/utils.ts` |

Herramientas: ESLint 9.39.4 (config plana, `typescript-eslint`, hooks y refresh de React), Vitest 4.1.4 (`npm test`, entorno `node`, sin Testing Library ni jsdom), `tsc -b` dentro de `npm run build`. Comillas y `;` varían entre archivos; no hay Prettier (ver `.agents/rules/naming-and-structure.md`).

## Backend (`backend/`)

| Pieza | Declarado | Resuelto el 2026-10-09 | Notas |
| --- | --- | --- | --- |
| FastAPI | sin versión | 0.143.0 (Python 3.13, Linux)* | app en `app/main.py`, rutas y modelos en `app/routes.py` |
| Uvicorn (`[standard]`) | sin versión | 0.54.0* | `--reload` en Docker |
| Pydantic | transitiva de FastAPI | — | modelos y `response_model` |
| pytest, pytest-cov, httpx | sin versión | 9.1.1, 7.1.0, 0.28.1* | `backend/tests/` (15 pruebas) |
| debugpy | sin versión | 1.8.22* | el contenedor arranca con `debugpy --listen 0.0.0.0:5678` |

\* Resolución con `pip install --dry-run --python-version 3.13 --platform manylinux2014_x86_64`; es una **foto**, no un pin. Las pruebas de la verificación se ejecutaron con FastAPI 0.128.8 / Starlette 0.49.3 en Python 3.9.6.

Sin ORM, sin base de datos, sin migraciones.

## Infraestructura y tooling

| Pieza | Detalle | Fuente |
| --- | --- | --- |
| Docker Compose | 2 servicios: `frontend` (5173) y `backend` (8000 y 5678 de depuración); montan el código como volumen | `docker-compose.yml` |
| Arranque documentado | `docker compose up --build`; frontend en `localhost:5173`, API en `localhost:8000`, docs en `localhost:8000/docs` | `README.md` |
| CI / despliegue | **No hay** (sin `.github/`, sin configuración de despliegue) | `git ls-files` |
| Variables de entorno | Solo `VITE_API_BASE_URL` (opcional, frontend) | `frontend/.env.example` |
| Reglas y memoria para agentes | `AGENTS.md`, `.agents/rules/` (7 archivos), `memory-bank/`; **no existe** `.agents/skills/` | repo |

## Brechas de reproducibilidad (resumen; detalle en `current-state.md`)

- `backend/requirements.txt` sin versiones fijadas.
- `frontend/Dockerfile` ejecuta `npm install` aunque hay `package-lock.json`.
- 13 vulnerabilidades en el lockfile del frontend según `npm ci` (1 baja, 5 moderadas, 7 altas).
