# Seguridad y dependencias

**Nombre:** `security-and-dependencies`
**Alcance:** `backend/app/main.py`, `backend/requirements.txt`, ambos `Dockerfile`, `docker-compose.yml`, `frontend/package.json` y `package-lock.json`, y cualquier archivo `.env*`.
**Por qué:** la configuración actual es cómoda para desarrollo local pero insegura o irreproducible si se reutiliza fuera de él. Es un proyecto didáctico sin datos reales, por eso hoy el riesgo es bajo; las reglas evitan que estos atajos se copien a un despliegue.

## Hechos medidos (2026-10-09)

- **CORS:** `allow_origins=["*"]` con `allow_credentials=True`. Probado con `TestClient`: una petición con `Origin` y cookie, o un preflight, recibe `access-control-allow-origin` igual al `Origin` (incluido `https://evil.example`); un GET sin cookie recibe `*`. En todos los casos responde `access-control-allow-credentials: true`. Cualquier sitio puede hacer peticiones con credenciales. Hoy no hay cookies ni login, así que no expone nada real.
- **Puerto de depuración:** el `Dockerfile` del backend arranca con `debugpy --listen 0.0.0.0:5678` y `docker-compose.yml` publica `5678:5678`. Un depurador accesible desde la red permite ejecutar código arbitrario.
- **Dependencias sin fijar:** `backend/requirements.txt` lista seis paquetes sin versión; cada build puede traer versiones distintas. El Dockerfile del frontend ejecuta `npm install` aunque existe `package-lock.json`.
- **Vulnerabilidades:** `npm ci` reporta 13 (1 baja, 5 moderadas, 7 altas) en el lockfile actual.
- **Sin `.dockerignore`:** el `COPY . .` de los Dockerfiles puede traer a la imagen un `node_modules` o `.venv` del equipo anfitrión y pisar lo instalado en el build. (No verificado: no hay Docker en el entorno de análisis.)

## Reglas

1. **No amplíes CORS por conveniencia; esta regla prevalece sobre «configuración amplia en local».** En desarrollo no hace falta CORS: el frontend usa el proxy de Vite para `/api`.
   - Los orígenes salen de la variable de entorno `CORS_ORIGINS` (lista separada por comas; recorta espacios y barras finales). Si no existe o está vacía, el valor por defecto es `http://localhost:5173`.
   - **Nunca `*` como origen**: ni en el valor por defecto ni aceptado desde la variable. Si algún origen contiene `*` (incluido `https://*.ejemplo.com`), **falla al arrancar** con un error claro; no lo ignores en silencio. Las entradas vacías sueltas (`"a, ,b"`) se descartan y los duplicados no hacen daño.
   - `allow_credentials=True` solo si se introduce un flujo con cookies; hoy debe ser `False`.
   - `allow_methods` y `allow_headers` pueden quedarse en `["*"]` o acotarse: el riesgo está en origen + credenciales, no en ellos.
   - Documenta la variable en ambos README (no hay plantilla `.env` para el backend) y pásala en `docker-compose.yml` con `CORS_ORIGINS=${CORS_ORIGINS:-}` (así el valor por defecto vive solo en el código).
2. **Depuración solo local.** No publiques `5678` ni ejecutes `debugpy` en una imagen o entorno que no sea de desarrollo; si separas configuración, `debugpy`, `pytest` y `pytest-cov` van en un archivo de dependencias de desarrollo. No es parte de una tarea de CORS o de versiones: anótalo en el informe, no lo cambies de paso.
3. **Instalaciones reproducibles.**
   - Frontend: `RUN npm ci` en el Dockerfile (respeta `package-lock.json`). No edites `package-lock.json` a mano.
   - Añade un `.dockerignore` **por contexto de build** para que `COPY . .` no meta artefactos del equipo anfitrión: `frontend/.dockerignore` (`node_modules`, `dist`) y `backend/.dockerignore` (`.venv`, `__pycache__`, `.pytest_cache`).
   - Backend: fija con `==` las dependencias **directas** de `requirements.txt` (mínimo exigido). Un lock completo de transitivas (`pip freeze`, pip-tools) es opcional y va en un cambio aparte.
   - Resuelve las versiones **para Python 3.13 en Linux** (el del Dockerfile), aunque tu intérprete local sea otro. No necesitas tenerlo instalado (usa el `pip` de cualquier entorno y un directorio temporal):
     ```bash
     D=$(mktemp -d)
     pip install --dry-run --ignore-installed --python-version 3.13 \
       --platform manylinux2014_x86_64 --only-binary=:all: --target "$D/t" \
       -r backend/requirements.txt --report "$D/report.json" -q
     ```
     Lee solo las versiones de las dependencias **directas** en `install[].metadata` del informe: la lista de transitivas es poco fiable (evalúa los marcadores de entorno con tu Python local). Si no puedes resolverlas, **no inventes versiones**: deja el archivo como estaba y repórtalo como pendiente.
4. **Actualizar dependencias es un cambio propio.** No ejecutes `npm audit fix` ni `npm audit fix --force`, ni subas versiones, dentro de otra tarea. Haz un commit dedicado y corre las cuatro comprobaciones de [testing.md](./testing.md).
5. **Secretos fuera del repo.** `.env` y `.env.*` están ignorados por `.gitignore` (solo `.env.example` se versiona, y es del frontend). No escribas claves ni URLs privadas en código, en `.env.example` ni en la documentación.

## Verificación

```bash
cd backend  && python -m pytest -q          # incluye las pruebas de CORS
cd frontend && npm test && npm run lint && npm run build
git diff --stat -- docker-compose.yml backend/Dockerfile frontend/Dockerfile backend/requirements.txt
```

Qué **no** valida lo anterior, y debes reportar como «no verificado» si lo tocaste: el build de las imágenes Docker, `npm ci` dentro de la imagen, la interpolación de variables en `docker-compose.yml` y las versiones fijadas (un pytest verde con un entorno que no se creó desde `requirements.txt` no las prueba).

## Origen

Reglas 8-9 de la fase 2 en [engineering-conventions.md](./engineering-conventions.md). Refinada tras validarla con la tarea «restringir CORS y hacer reproducibles las instalaciones».
