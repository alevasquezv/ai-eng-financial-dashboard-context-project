# Memory bank

Memoria del proyecto para agentes y contribuidores: lo que **se sabe y está verificado** sobre este repositorio. `AGENTS.md` pide leerla antes de actuar.

| Archivo | Para qué |
| --- | --- |
| [project-overview.md](./project-overview.md) | Qué es el producto, qué hace hoy y qué no se sabe |
| [tech-stack.md](./tech-stack.md) | Lenguajes, frameworks, infraestructura y versiones (declaradas y resueltas) |
| [current-state.md](./current-state.md) | Qué funciona, qué no se pudo verificar, brechas conocidas y prioridades |

Las reglas de trabajo viven aparte, en `.agents/rules/`. Aquí no se repiten: se enlazan.

## Reglas de este banco

1. **Cada afirmación lleva evidencia:** un archivo, un comando o una medición con su fecha. Sin fuente, no entra.
2. **Se rechazan los claims de producto sin soporte** (usuarios objetivo, objetivos de negocio, despliegue, «tiempo real», «seguro», etc.) y los roadmaps inventados. Lo que no se sabe se escribe como «no se sabe».
3. **Hechos verificados e hipótesis van separados.** Una hipótesis nunca se escribe como brecha confirmada.
4. **No copies datos que caducan** (listas de valores permitidos, conteos de líneas): enlaza a la fuente.
5. **Actualiza al cambiar la realidad.** Si un hecho deja de ser cierto, corrígelo o elimínalo en el mismo cambio y actualiza la fecha de «Última verificación». Una nota obsoleta es peor que no tener nota (ver `.agents/rules/docs-and-commits.md`, regla 6).
