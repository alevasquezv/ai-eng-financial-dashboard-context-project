# Nombres, estructura y estilo

**Nombre:** `naming-and-structure`
**Alcance:** archivos nuevos o movidos en `backend/` y `frontend/src/`, y cualquier edición de código existente.
**Por qué:** cada capa tiene su convención idiomática y el repo ya las sigue; romperlas hace que el código nuevo no se encuentre ni se lea igual que el resto. No hay Prettier ni formateador configurado, así que el estilo de comillas y punto y coma **difiere entre archivos**: reformatear por costumbre genera diffs enormes que esconden el cambio real.

## Reglas

1. **Python:** `snake_case` para funciones, variables y módulos; `PascalCase` para modelos Pydantic y alias de tipo (`FinancialMovement`, `OperationType`); constantes en `MAYÚSCULAS` (`OUTCOME_CATEGORIES`). Funciones auxiliares internas con prefijo `_` (`_build_movement`). Comillas dobles. Comentarios y docstrings en inglés, como los identificadores. Código nuevo de **≤ 88 columnas**. `routes.py` ya tiene algunas líneas más largas: no las reflowees. Si al modificar una línea existente esta pasa de 88, parte solo esa sentencia.
2. **React/TypeScript:** componentes y tipos en `PascalCase`; **archivos en `kebab-case`** (`kpi-card.tsx`, `financial-utils.ts`); un componente principal por archivo; props tipadas con `interface <Nombre>Props`; **export nombrado** (`export function KPICard`). Solo `App.tsx` usa `export default`.
3. **Dónde va cada cosa (frontend):**
   - `src/components/dashboard/` → componentes propios del dashboard: KPIs, gráficas, cabecera, y cualquier pieza con textos o comportamiento específicos de esta aplicación (p. ej. un banner de error).
   - `src/components/ui/` → solo primitivas genéricas estilo shadcn (`card`, `skeleton`), sin textos ni lógica de la aplicación.
   - Ante la duda: **`dashboard/`**.
   - `src/lib/` → tipos (`financial-types.ts`), cálculos y formateo (`financial-utils.ts`) y sus pruebas `*.test.ts`.
   - Importa con el alias `@/` (apunta a `src/`), no con rutas relativas largas (`../../`).
4. **Respeta el estilo del archivo que editas, y el de su carpeta si el archivo es nuevo.** Estado real medido:

   | Archivos | Comillas | `;` al final |
   | --- | --- | --- |
   | `components/**` (dashboard y ui), `main.tsx`, `lib/financial-types.ts` | simples | no |
   | `App.tsx`, `lib/financial-utils.ts`, `lib/financial-utils.test.ts` | dobles | sí |
   | `lib/utils.ts` | dobles | no |

   Archivo nuevo en `components/` → comillas simples sin `;`. Archivo nuevo en `lib/` → copia a su hermano más parecido (lógica: `financial-utils.ts`; tipos: `financial-types.ts`; prueba: `financial-utils.test.ts`). No reformatees archivos enteros ni mezcles estilos dentro de un mismo archivo.
5. **Un cambio, un propósito.** No renombres, muevas ni reformatees archivos como efecto secundario de otra tarea. Si, mientras trabajas, ves algo que incumple otra regla y queda **fuera del alcance de la tarea** (p. ej. un periodo hardcodeado mientras extraes un componente), **no lo corrijas en el mismo cambio: anótalo en tu informe**. Excepción: la propia tarea lo pide.
6. **Extraer o mover un componente es un refactor: no cambia comportamiento.** Conserva props, clases y textos tal cual (no traduzcas ni reescribas el texto al moverlo) y deja el condicional de renderizado donde estaba, en el padre.
7. **No uses código muerto como referencia.** `frontend/src/lib/mock-data.ts` no lo importa ningún archivo; la fuente de datos es la API.
8. **Textos de interfaz:** el dashboard está en inglés (`Financial Overview`, `Total Income`), pero el mensaje de error de `App.tsx` está en español. Al **añadir** texto visible, sigue el idioma del componente donde lo pones; no traduzcas el resto como parte de otra tarea.

## Verificación

```bash
cd frontend && npm run lint && npm run build
git status --short   # archivos modificados Y nuevos: solo lo que querías cambiar
git diff --stat      # tamaño de los cambios en archivos ya trackeados (no muestra los nuevos)
```

## Origen

Regla 7 de la fase 2 en [engineering-conventions.md](./engineering-conventions.md). Refinada tras validarla con la tarea «extraer el banner de error de `App.tsx` a su propio componente».
