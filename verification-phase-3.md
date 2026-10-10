# Verificación de la fase 3: reglas del repositorio

## Qué se hizo

Se convirtieron las diez reglas propuestas de la fase 2 en seis archivos operativos dentro de `.agents/rules/` (nombre, alcance, justificación y guía específica del proyecto cada uno), y se validaron con tareas pequeñas reales sobre este repo.

| Archivo de reglas | Reglas de la fase 2 | Tarea de validación |
| --- | --- | --- |
| `api-and-contracts.md` | 1-3 | Añadir la categoría de gasto `payroll` (backend, tipo del frontend, pruebas) |
| `dates-and-periods.md` | 4-5 | Corregir los totales mensuales en `America/Mexico_City` y mostrar el periodo real en la cabecera |
| `testing.md` | 6 | Reforzar dos pruebas débiles del backend |
| `naming-and-structure.md` | 7 | Extraer el banner de error de `App.tsx` a un componente |
| `security-and-dependencies.md` | 8-9 | Restringir CORS y hacer reproducibles las instalaciones |
| `docs-and-commits.md` | 10 + higiene de commits | Documentar cómo correr las pruebas y la versión mínima de Python, y commitear |

`engineering-conventions.md` (fase 2) se conserva como análisis de origen; su cabecera ahora dice que prevalecen los archivos nuevos.

## Método

1. **Estado base** medido antes de tocar nada: pytest 15 OK, vitest 5 OK, eslint limpio, build OK.
2. **Ronda 1:** un agente nuevo por tarea, en un worktree git desechable, que solo conocía `AGENTS.md` y las reglas (como un contribuidor nuevo). Debía hacer la tarea, correr las cuatro comprobaciones y reportar dónde la regla fue ambigua, faltó o se contradijo.
3. **Refinamiento** de las seis reglas con esa fricción.
4. **Ronda 2:** las mismas seis tareas con agentes nuevos sobre las reglas refinadas, pidiendo además cuántas decisiones tuvieron que tomar porque ninguna regla las cubría.
5. Correcciones finales con la fricción de la ronda 2, verificadas directamente (ver «Límites»).

Los cambios de código de las tareas **se descartaron** (los worktrees y ramas temporales se eliminaron); aquí queda solo la evidencia.

## Resultados

| Tarea | Ronda 1: puntos de fricción listados | Ronda 2: decisiones sin regla | Comprobaciones de la ronda 2 |
| --- | --- | --- | --- |
| `payroll` | ~11 | 2 | pytest 17 OK (3 fallos antes del cambio), vitest 5, lint, build |
| Fechas y periodo | ~8 | 5 | vitest 12 OK en 3 zonas; con el bug reintroducido fallan 6 pruebas **incluso con `TZ=UTC`**; datos reales: 0 de 12 meses incorrectos |
| Pruebas débiles | ~7 | 6 | pytest 16 OK; 12 de 13 mutaciones detectadas; verde en 144 fechas simuladas de «hoy» |
| Banner | ~10 | 3 | las 4 comprobaciones sin cambios |
| CORS y dependencias | ~9 | 6 | pytest 28 OK (12 pruebas CORS nuevas); versiones de las 6 dependencias directas resueltas para Python 3.13 |
| Docs y commit | ~9 | 6 | commit con solo `README.md` y `README.es.md`; lo no verificado listado en el cuerpo |

Total aproximado: de ~54 puntos de fricción a 28 decisiones propias. **Las dos cifras no miden exactamente lo mismo** (la ronda 1 no pidió el conteo; lo reconstruí contando los puntos de cada informe), así que indican una tendencia, no una medida exacta. Ninguna decisión de la ronda 2 bloqueó la tarea.

## Defectos de las propias reglas que la validación encontró

Ronda 1 (arreglados antes de la ronda 2):
- Las reglas copiaban valores que caducan («`Category` = …», «hoy tiene 391 líneas», conteos de pruebas).
- La regla de fechas pedía correr `TZ=... npm test` a mano: en un CI con `TZ=UTC` la prueba pasaba **sin** el arreglo.
- Faltaban: cuándo commitear, qué hacer con comandos que no se pueden ejecutar, cómo arbitrar entre reglas, y qué es vinculante (`engineering-conventions.md` se declaraba «propuesta»).

Ronda 2 (errores que **introduje yo** al refinar; arreglados después):
- El fragmento para fijar `TZ` dentro de la prueba **rompía `npm run build`** (`TS2591`, `tsconfig.app.json` solo carga los tipos de `vite/client`). Verificado: con `/// <reference types="node" />` compila.
- «`npm run build` falla sin el cambio» es falso para un tipo que nada consume.
- El comando de `pip` para resolver versiones de Python 3.13 no fijaba plataforma y exponía transitivas de un Python 3.9; verificado con `--platform manylinux2014_x86_64`.
- `git diff --stat` no muestra archivos nuevos sin trackear.

## Límites (qué NO se comprobó)

- **Mismas tareas en las dos rondas:** las reglas se refinaron con esas tareas, así que la ronda 2 prueba menos su generalidad (un agente lo señaló).
- **Las correcciones finales no se repitieron con agentes.** Las dos de contenido técnico (`reference` de TypeScript y el comando de `pip`) las probé yo; el resto son cambios de redacción.
- **Docker no estaba disponible:** no se construyó ninguna imagen, ni se probó `npm ci` en imagen, ni la interpolación de `CORS_ORIGINS` en `docker-compose.yml`, ni el efecto de los `.dockerignore`.
- **Python local 3.9** con `eval_type_backport` solo en el venv de validación (el proyecto necesita ≥ 3.10). Las versiones fijadas (p. ej. fastapi 0.143) **nunca se ejecutaron**: las pruebas corrieron con fastapi 0.128.8.
- Las cuatro comprobaciones se corrieron en mi entorno; el build de Vite avisa de un chunk > 500 kB (informativo).

## Hallazgos del repo observados por el camino (sin corregir)

- `npm ci` reporta 13 vulnerabilidades (1 baja, 5 moderadas, 7 altas) en el lockfile.
- `debugpy` escucha en `0.0.0.0:5678` y `docker-compose.yml` lo publica.
- CORS refleja cualquier origen con `allow-credentials: true`.
- `frontend/src/lib/mock-data.ts` no lo importa nadie; el periodo `2024 - Full Year` sigue fijo en `App.tsx`.
- `test_metrics_alerts_returns_anomaly_candidates` usa `if payload:` (lo que `testing.md` desaconseja).
