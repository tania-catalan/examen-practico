---
name: ejecutar-prompt-artefacto
description: >
  Ejecuta la última versión de un prompt ejecutable (PRD_v#, FSD_v#, ADR_v#, C4_N1_N2_v#)
  y escribe el artefacto en docs/<area>/ consolidando secciones y acumulando métricas por ejecución.
  Usar cuando el usuario pida crear o actualizar un artefacto (PRD, FSD, ADR, C4), ejecutar un
  prompt versionado, o repetir la generación sin perder historial de métricas.
---

# Skill: Ejecutar última versión del prompt y generar artefacto

Ejecuta el prompt versionado más reciente de un tipo de documento y produce (o actualiza) el artefacto en `docs/<area>/`, **sin regenerar desde cero** y **sin borrar métricas** de ejecuciones anteriores.

---

## Cuándo activarlo

- El usuario pide **crear** o **actualizar** un artefacto: PRD, FSD, ADR, diagrama C4, etc.
- El usuario menciona ejecutar `PRD_v#`, `FSD_v#`, o el patrón *prompt → docs*.
- Hay que **re-ejecutar** el mismo prompt (misma versión) y conservar el historial de métricas.

---

## Reglas obligatorias (no negociables)

### Consolidación del artefacto

- **No generar de nuevo todo el contenido del artefacto**, para la nueva version [sufijo_v#.md] solo actualizar y consolida la misma salida y la salida anterior de cada seccion existente, si hay una nueva seccion agregalo al documento de salida

Comportamiento concreto:

1. **Leer antes de escribir**: prompt vigente + artefacto de salida actual (si existe) + artefacto de la versión anterior del mismo tipo (si existe, p. ej. `PRD_v1.md` al generar `PRD_v2.md`).
2. **Por cada sección** definida en el prompt (§ Output / secciones obligatorias):
   - Si la sección **ya existe** en el artefacto previo: **fusionar** — conservar lo válido, enriquecer con lo nuevo, resolver contradicciones a favor del brief/`docs/contexto.md` y del prompt.
   - Si la sección **es nueva** en el prompt y no está en el artefacto: **añadirla** al final del bloque de contenido (antes de `## Métricas`).
3. **Prohibido**: reemplazar el documento entero ignorando el contenido previo; eliminar secciones que siguen siendo válidas; duplicar secciones con el mismo título (unificar en una sola).

### Métricas acumulativas

- **No remover las metrias de las anteriores ejecuciones de la version actual**, agregar una nueva subseccion con los resultados de las nuevas metricas. El objetivo es que en la nueva version se encuentren todas las metricas de todas las ejecuciones.

Comportamiento concreto:

1. La sección `## Métricas` va **al final** del artefacto (como indica cada prompt `_v#`).
2. Dentro de `## Métricas`, cada ejecución es una subsección **inmutable** una vez escrita:

   ```markdown
   ## Métricas

   ### Ejecución 1 — 2026-05-22T14:30:00-04:00
   - **Prompt**: `PR-PRD-FTGO-001` / `v1`
   - **Run ID**: `20260522-143000`

   | Nombre de la métrica | Valor | Insights |
   | --- | ---: | --- |
   | Completitud del output (%) | 92 | … |

   ### Ejecución 2 — 2026-05-23T09:15:00-04:00
   …
   ```

3. **Nunca** editar ni borrar tablas/filas de ejecuciones pasadas; solo **añadir** `### Ejecución N — <timestamp ISO-8601>`.
4. Numerar ejecuciones de forma consecutiva (`Ejecución 1`, `Ejecución 2`, …).
5. Calcular métricas según `docs/METRICS.md` (fórmulas, umbrales y columnas aplicables por tipo de documento).

---

## Convenciones de rutas

| Concepto | Ruta | Ejemplo |
|----------|------|---------|
| Prompt ejecutable | `prompts/<TIPO>_v<N>.md` | `prompts/PRD_v1.md` |
| Artefacto generado | `docs/<area>/<TIPO>_v<N>.md` | `docs/prd/PRD_v1.md` |
| Brief / dominio | `docs/contexto.md` | — |
| Referencia de métricas | `docs/METRICS.md` | — |

### Mapeo tipo → carpeta de salida

| Tipo (prefijo del archivo) | Carpeta `docs/` | Ejemplo salida |
|----------------------------|-----------------|----------------|
| `PRD` | `prd` | `docs/prd/PRD_v1.md` |
| `FSD` | `fsd` | `docs/fsd/FSD_v1.md` |
| `ADR` | `adr` | `docs/adr/ADR_v1.md` |
| `C4_N1_N2` | `c4` | `docs/c4/C4_N1_N2_v1.md` |

> **Compatibilidad con este repositorio**: si no existe `prompts/`, buscar prompts en `docs/prompts/` con el mismo patrón `<TIPO>_v<N>.md`. La salida sigue siendo siempre `docs/<area>/<TIPO>_v<N>.md`.

---

## Resolver la última versión del prompt

1. Extraer `<TIPO>` del pedido del usuario (`PRD`, `FSD`, `ADR`, `C4_N1_N2`, …).
2. Listar archivos que coincidan con `prompts/<TIPO>_v*.md` (o `docs/prompts/<TIPO>_v*.md`).
3. Elegir el **mayor N** en el sufijo `_v<N>` (comparación numérica: `v2` > `v1`).
4. El artefacto de salida usa **el mismo N**: `docs/<area>/<TIPO>_v<N>.md`.

---

## Procedimiento (checklist)

```
- [ ] 1. Identificar TIPO (PRD, FSD, ADR, C4_N1_N2, …)
- [ ] 2. Localizar prompt más reciente: prompts/<TIPO>_v<N>.md
- [ ] 3. Definir ruta de salida: docs/<area>/<TIPO>_v<N>.md
- [ ] 4. Leer prompt completo + docs/contexto.md + entradas citadas en § Context
- [ ] 5. Si existe artefacto previo (misma vN o vN-1): cargarlo para consolidación
- [ ] 6. Ejecutar la tarea del prompt (Role, Task, Reasoning, Stop condition)
- [ ] 7. Consolidar sección por sección (regla de no regenerar todo)
- [ ] 8. Calcular métricas de esta ejecución (docs/METRICS.md)
- [ ] 9. Añadir subsección ### Ejecución K en ## Métricas (sin tocar ejecuciones anteriores)
- [ ] 10. Crear docs/<area>/ si no existe; escribir/actualizar el artefacto
- [ ] 11. Actualizar [`docs/PROMPT_MAPPING.md`](../../docs/PROMPT_MAPPING.md): Run ID, fecha, artefacto, autor, modelo, temperatura (§2 y §3)
- [ ] 12. Informar al usuario: prompt usado, ruta de salida, número de ejecución de métricas
```

---

## Ejemplos de uso

### Ejemplo 1 — Crear artefacto PRD (primera ejecución)

**Usuario:** «Crear artefacto PRD»

**Acciones:**

1. Último prompt: `prompts/PRD_v1.md` (o `docs/prompts/PRD_v1.md`).
2. Salida: `docs/prd/PRD_v1.md` (documento nuevo; secciones según § Output del prompt).
3. `## Métricas` con `### Ejecución 1 — <timestamp>` y tabla completa.

### Ejemplo 2 — Re-ejecutar el mismo prompt (segunda ejecución, misma versión)

**Usuario:** «Vuelve a ejecutar el PRD»

**Acciones:**

1. Mismo prompt `PRD_v1.md`, mismo archivo `docs/prd/PRD_v1.md`.
2. Consolidar cada sección existente con el contenido previo.
3. Añadir `### Ejecución 2 — …` en Métricas; **mantener** `### Ejecución 1` intacta.

### Ejemplo 3 — Otros tipos

| Pedido | Prompt (última v) | Salida |
|--------|-------------------|--------|
| Crear FSD | `prompts/FSD_v1.md` | `docs/fsd/FSD_v1.md` |
| Crear ADR | `prompts/ADR_v1.md` | `docs/adr/ADR_v1.md` |
| Crear C4 N1/N2 | `prompts/C4_N1_N2_v1.md` | `docs/c4/C4_N1_N2_v1.md` |

### Ejemplo 4 — Nueva versión del prompt (`v2`)

**Usuario:** «Ejecutar PRD con la nueva versión del prompt»

**Acciones:**

1. Prompt: `prompts/PRD_v2.md`.
2. Salida: `docs/prd/PRD_v2.md`.
3. Consolidar usando `docs/prd/PRD_v1.md` como salida anterior de referencia.
4. Métricas: empezar `### Ejecución 1` en el artefacto `PRD_v2.md` (historial de `PRD_v1.md` no se mueve; permanece en su archivo).

---

## Métricas por tipo de documento

Usar solo las filas aplicables según `docs/METRICS.md`:

| Métrica | PRD | FSD | C4 | ADR |
|---------|:---:|:---:|:--:|:---:|
| Completitud del output (%) | ✓ | ✓ | ✓ | ✓ |
| % Secciones cubiertas | ✓ | ✓ | ✓ | ✓ |
| Cobertura NFRs (%) | ✓ | — | — | — |
| UCs entregados (count) | — | ✓ | — | — |
| % UCs con Given/When/Then | — | ✓ | — | — |
| Trazabilidad (%) | ✓ | ✓ | ✓ | ✓ |
| Sintaxis Mermaid válida | — | — | ✓ | — |
| Reducción de iteraciones (%) | ✓ | ✓ | ✓ | ✓ |
| Tiempo hasta convergencia (min) | ✓ | ✓ | ✓ | ✓ |
| Ediciones humanas (count) | ✓ | ✓ | ✓ | ✓ |
| Inventado / Hallucination (%) | ✓ | ✓ | ✓ | ✓ |

Para cálculos heurísticos en Windows, ver ejemplos PowerShell en `docs/METRICS.md`.

---

## Anti-patrones

- Regenerar el PRD/FSD completo ignorando `docs/prd/PRD_v1.md` existente.
- Sobrescribir `## Métricas` con una sola tabla (pierde historial).
- Usar un prompt `v1` pero escribir en `PRD_v2.md` (desalineación versión prompt ↔ artefacto).
- Inventar dominio fuera de `docs/contexto.md` cuando el prompt lo prohíbe.
- Omitir la sección `## Métricas` al final del artefacto.

---

## Salida esperada al cerrar

Informar al usuario en español:

1. **Prompt ejecutado**: ruta y versión (`PRD_v2.md`).
2. **Artefacto**: ruta (`docs/prd/PRD_v2.md`).
3. **Consolidación**: qué secciones se fusionaron, cuáles se añadieron.
4. **Métricas**: número de ejecución registrada y 2–3 insights destacados de la tabla nueva.
