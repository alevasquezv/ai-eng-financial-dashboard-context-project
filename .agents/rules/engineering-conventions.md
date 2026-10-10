# Hallazgos y reglas propuestas

> **Estado:** análisis de origen (fase 2). Las reglas vigentes y vinculantes están en los demás archivos de esta carpeta (`api-and-contracts`, `dates-and-periods`, `testing`, `naming-and-structure`, `security-and-dependencies`, `docs-and-commits`), que desarrollan y refinan las diez reglas de abajo tras validarlas con tareas reales. Si este documento difiere de ellos, prevalecen ellos.

## Hallazgos agrupados

### Arquitectura y API
- `backend/app/routes.py` concentra modelos Pydantic, generación de datos, filtros, agregaciones y handlers HTTP.
- Los endpoints usan `response_model` y tipos `Literal`, lo que da un contrato explícito en la API.

### Contratos entre frontend y backend
- Los modelos del backend y los tipos TypeScript representan por separado el movimiento financiero y sus categorías.
- Cambiar campos o valores permitidos implica mantener ambas definiciones compatibles.

### Testing
- Las pruebas backend cubren salud, endpoints, filtros, facets, resúmenes, categorías, comparaciones y alertas.
- Las pruebas frontend cubren cálculos financieros, agregación mensual y formatters; no se observan pruebas de componentes React en el inventario actual.
- Los casos de agregación mensual no incluyen movimientos fechados el primer día del mes ni verifican el comportamiento en distintas zonas horarias.

### Fechas y presentación de datos
- El backend genera movimientos para los doce meses y devuelve fechas como fechas de calendario.
- El frontend convierte `create_date` con `new Date(...)` y luego obtiene año y mes en hora local; en zonas al oeste de UTC, una fecha ISO sin hora puede caer en el día o mes local anterior.
- El encabezado del dashboard muestra el periodo fijo `2024 - Full Year`, mientras el backend genera datos tomando como referencia la fecha actual.

### Naming y organización frontend
- El backend usa nombres `snake_case`; los componentes React usan `PascalCase` y sus archivos usan nombres `kebab-case`.
- El código de interfaz separa componentes de dashboard, componentes UI y utilidades/tipos bajo `frontend/src/`; el alias `@` apunta a `src`.

### Seguridad y reproducibilidad
- CORS permite todos los orígenes, métodos y headers, además de credenciales.
- `backend/requirements.txt` no fija versiones. El frontend sí tiene `package-lock.json`, pero su Dockerfile ejecuta `npm install` en lugar de instalar estrictamente desde el lockfile.

### Documentación y agentes
- `README.es.md` documenta Docker Compose, puertos y proxy `/api`, y pide documentar reglas propuestas en `.agents/rules/`.
- `AGENTS.md` instruye a los agentes revisar `.agents/rules/`, `.agents/skills/` y `memory-bank/` antes de actuar.

## Reglas propuestas

### Arquitectura y API

1. **Mantén los handlers HTTP delgados.** Deja en las rutas la validación y coordinación de la petición; mueve generación, filtros y cálculos a módulos de servicio cuando el archivo o la lógica crezcan. Esto limita el acoplamiento y facilita probar la lógica sin invocar HTTP. **Hecho del repo:** modelos, generación, filtros, agregaciones y endpoints están en el mismo archivo: [`backend/app/routes.py`](../../backend/app/routes.py#L22), [`backend/app/routes.py`](../../backend/app/routes.py#L94), [`backend/app/routes.py`](../../backend/app/routes.py#L107), [`backend/app/routes.py`](../../backend/app/routes.py#L161), [`backend/app/routes.py`](../../backend/app/routes.py#L243).

2. **Define y conserva contratos explícitos en la API.** Usa modelos Pydantic y `response_model` en endpoints; ante cambios en campos o valores permitidos, actualiza las pruebas que verifican la respuesta. **Hecho del repo:** el backend declara tipos `Literal`, modelos Pydantic y `response_model` en endpoints: [`backend/app/routes.py`](../../backend/app/routes.py#L11), [`backend/app/routes.py`](../../backend/app/routes.py#L22), [`backend/app/routes.py`](../../backend/app/routes.py#L248).

### Contratos y datos

3. **Sincroniza el contrato frontend/backend en el mismo cambio.** Si cambia un campo, categoría o tipo de operación, actualiza tanto el modelo Pydantic como los tipos TypeScript y añade o ajusta una prueba de contrato/comportamiento. **Hecho del repo:** el backend y el frontend mantienen definiciones separadas de `FinancialMovement`, `OperationType`, `Category` y `BusinessType`: [`backend/app/routes.py`](../../backend/app/routes.py#L11), [`backend/app/routes.py`](../../backend/app/routes.py#L22), [`frontend/src/lib/financial-types.ts`](../../frontend/src/lib/financial-types.ts#L1).

4. **Trata las fechas `YYYY-MM-DD` como fechas de calendario, no como instantes locales.** Evita que el parsing UTC y los getters locales desplacen el mes; agrega casos del primer día del mes y cobertura con una zona horaria al oeste de UTC. **Hecho del repo:** `computeMonthlyData` parsea `create_date` con `new Date(...)` y `toYearMonthKey` usa getters locales: [`frontend/src/lib/financial-utils.ts`](../../frontend/src/lib/financial-utils.ts#L7), [`frontend/src/lib/financial-utils.ts`](../../frontend/src/lib/financial-utils.ts#L41). La prueba mensual actual usa días 3, 5 y 8: [`frontend/src/lib/financial-utils.test.ts`](../../frontend/src/lib/financial-utils.test.ts#L63).

5. **Haz que los periodos visibles se deriven del conjunto de datos o se expliquen como fijos.** No dejes un periodo codificado como dinámico si la API entrega otro rango. **Hecho del repo:** el título del dashboard está fijado a `2024 - Full Year`, y la generación del backend calcula años en relación con `date.today()`: [`frontend/src/App.tsx`](../../frontend/src/App.tsx#L49), [`backend/app/routes.py`](../../backend/app/routes.py#L65), [`backend/app/routes.py`](../../backend/app/routes.py#L97).

### Testing

6. **Prueba la regla de negocio en la capa donde se implementa y los contratos en la frontera HTTP.** Añade pruebas de componentes solo cuando se introduzca comportamiento relevante en esos componentes; conserva pruebas unitarias para cálculos y pruebas de endpoints para filtros/respuestas. **Hecho del repo:** las pruebas backend cubren filtros y endpoints, mientras el archivo frontend existente prueba utilidades financieras y formatters: [`backend/tests/test_routes.py`](../../backend/tests/test_routes.py#L29), [`backend/tests/test_routes.py`](../../backend/tests/test_routes.py#L121), [`frontend/src/lib/financial-utils.test.ts`](../../frontend/src/lib/financial-utils.test.ts#L35), [`frontend/src/lib/financial-utils.test.ts`](../../frontend/src/lib/financial-utils.test.ts#L106).

### Naming y organización

7. **Sigue las convenciones idiomáticas de cada capa y la organización existente.** Usa `snake_case` para Python, `PascalCase` para componentes React, `kebab-case` para archivos de componentes y agrupa UI, dashboard y lógica reutilizable en sus directorios actuales. **Hecho del repo:** [`backend/app/routes.py`](../../backend/app/routes.py#L22), [`frontend/src/components/dashboard/`](../../frontend/src/components/dashboard/), [`frontend/src/components/ui/`](../../frontend/src/components/ui/), [`frontend/src/lib/`](../../frontend/src/lib/).

### Seguridad y dependencias

8. **No amplíes CORS globalmente por conveniencia.** Para despliegues compartidos o públicos, configura los orígenes necesarios explícitamente y habilita credenciales solo si el flujo las requiere; conserva la configuración amplia únicamente en un entorno local claramente delimitado. **Hecho del repo:** la configuración actual usa `allow_origins=["*"]`, `allow_methods=["*"]`, `allow_headers=["*"]` y `allow_credentials=True`: [`backend/app/main.py`](../../backend/app/main.py#L7).

9. **Haz reproducibles las instalaciones de dependencias.** Fija las dependencias del backend mediante versiones o un lockfile y usa `npm ci` en la imagen frontend para respetar `package-lock.json`. **Hecho del repo:** los requisitos Python no tienen versiones fijadas y Docker instala con `pip install`; existe lockfile frontend, pero Docker usa `npm install`: [`backend/requirements.txt`](../../backend/requirements.txt#L1), [`backend/Dockerfile`](../../backend/Dockerfile#L5), [`frontend/package-lock.json`](../../frontend/package-lock.json), [`frontend/Dockerfile`](../../frontend/Dockerfile#L5).

### Documentación y agentes

10. **Actualiza las instrucciones de ejecución y de agentes cuando cambie el flujo real.** Mantén claros los comandos, puertos, proxy y ubicaciones de reglas; antes de actuar, revisa las instrucciones y memoria del proyecto si existen. **Hecho del repo:** la ejecución documentada usa `docker compose up --build` y el proxy `/api`; `AGENTS.md` requiere revisar reglas, skills y `memory-bank/`: [`README.es.md`](../../README.es.md#L39), [`AGENTS.md`](../../AGENTS.md#L3).

## Prioridad sugerida

1. Corregir/definir el tratamiento de fechas y el periodo mostrado, porque afectan la interpretación de los gráficos.
2. Mantener sincronizados los contratos frontend/backend y ampliar pruebas de límites de fecha.
3. Mejorar reproducibilidad de dependencias y acotar CORS antes de desplegar fuera del entorno local.
4. Separar lógica de `routes.py` cuando se añadan nuevos endpoints o aumente su complejidad.
