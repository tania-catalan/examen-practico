# Mapeo de prompts ejecutables y artefactos generados

Registro canónico de **prompts versionados**, **ejecuciones** y **artefactos** del laboratorio FTGO.  
Referenciado desde [`docs/prompts/PROMPT.md`](prompts/PROMPT.md) y el skill [`agents/skills/SKILL.md`](../agents/skills/SKILL.md).

**Convención de rutas**

| Elemento | Ruta en este repo |
| --- | --- |
| Prompt ejecutable | `docs/prompts/<TIPO>_v<N>.md` |
| Artefacto | `docs/<area>/…` (ver tabla por tipo) |
| Brief | [`docs/contexto.md`](contexto.md) |

---

## 1. Catálogo de prompts ejecutables (última versión por tipo)

| Tipo | Archivo prompt (última v) | ID prompt | Versión prompt | Fecha prompt | Autor prompt | Modelo recomendado | Temperatura | Carpeta salida |
| --- | --- | --- | --- | --- | --- | --- | ---: | --- |
| **PRD** | [`prompts/PRD_v1.md`](prompts/PRD_v1.md) | `PR-PRD-FTGO-001` | v1 | 22/05/2026 | Módulo 4 - UMSS | Sonnet / Opus | 0.2 | `docs/prd/` |
| **FSD** | [`prompts/FSD_v1.md`](prompts/FSD_v1.md) | `PR-FSD-FTGO-001` | v1 | 22/05/2026 | Módulo 4 - UMSS | Sonnet | 0.2 | `docs/fsd/` |
| **ADR** | [`prompts/ADR_v1.md`](prompts/ADR_v1.md) | `PR-ADR-FTGO-001` | v1 | 22/05/2026 | Módulo 4 - UMSS | Opus | 0.3 | `docs/adr/` |
| **C4** | [`prompts/C4_N1_N2_v2.md`](prompts/C4_N1_N2_v2.md) | `PR-C4-FTGO-001` | v2 | 23/05/2026 | Módulo 4 - UMSS | Sonnet / Opus | 0.2 | `docs/c4/` |

> **Plantillas no ejecutables** (semilla, sin sufijo `_vN`): `PRD.md`, `FSD.md`, `ADR.md`, `C4_N1_N2.md` en `docs/prompts/`.

---

## 2. Registro de ejecuciones y artefactos generados

Columna **Modelo (ejecución)**: valor real usado en la corrida. Si el artefacto no lo registra, se indica *según prompt* (heredado del catálogo §1).

| Run ID | Fecha ejecución (ISO) | ID prompt | v prompt | Parámetro / notas | Artefacto generado | Autor ejecución | Modelo (ejecución) | Temp. | Estado artefacto |
| --- | --- | --- | --- | --- | --- | --- | --- | ---: | --- |
| `20260523-120000Z` | 2026-05-23T12:00:00Z | `PR-PRD-FTGO-001` | v1 | Primera generación | [`prd/PRD_v1.md`](prd/PRD_v1.md) | Módulo 4 - UMSS | Según prompt (Sonnet/Opus) | 0.2 | Generado |
| `20260523-174500-0400` | 2026-05-23T17:45:00-04:00 | `PR-PRD-FTGO-001` | v1 | Consolidación v1 → **artefacto v2** | [`prd/PRD_v2.md`](prd/PRD_v2.md) | Módulo 4 - UMSS | Según prompt (Sonnet/Opus) | 0.2 | Generado |
| `20260523-174000-0400` | 2026-05-23T17:40:00-04:00 | `PR-FSD-FTGO-001` | v1 | Entrada: PRD_v1 + brief | [`fsd/FSD_v1.md`](fsd/FSD_v1.md) | Módulo 4 - UMSS | Según prompt (Sonnet) | 0.2 | Generado |
| `20260523-180000-0400-adr0001` | 2026-05-23T18:00:00-04:00 | `PR-ADR-FTGO-001` | v1 | *estrategia de descomposición* | [`adr/ADR-0001.md`](adr/ADR-0001.md) | Módulo 4 - UMSS | Según prompt (Opus) | 0.3 | Accepted |
| `20260523-180500-0400-adr0002` | 2026-05-23T18:05:00-04:00 | `PR-ADR-FTGO-001` | v1 | *mecanismo IPC predominante* | [`adr/ADR-0002.md`](adr/ADR-0002.md) | Módulo 4 - UMSS | Según prompt (Opus) | 0.3 | Accepted |
| `20260523-181000-0400-adr0003` | 2026-05-23T18:10:00-04:00 | `PR-ADR-FTGO-001` | v1 | *estrategia de datos / consistencia* | [`adr/ADR-0003.md`](adr/ADR-0003.md) | Módulo 4 - UMSS | Según prompt (Opus) | 0.3 | Accepted |
| `20260523-190000-0400-c4` | 2026-05-23T19:00:00-04:00 | `PR-C4-FTGO-001` | v2 | N1+N2; PRD_v2 + ADR 0001–0003 | [`c4/C4_N1_N2_v1.md`](c4/C4_N1_N2_v1.md) (+ `c4_context.mmd`, `c4_container.mmd`) | Módulo 4 - UMSS | Según prompt (Sonnet/Opus) | 0.2 | Generado |

**Pendiente de ejecución:** ninguno (catálogo base completo).

---

## 3. Matriz prompt (versión) → artefactos

### PRD (`PR-PRD-FTGO-001`)

| Versión prompt | Artefactos generados | Ejecuciones (Run ID) |
| --- | --- | --- |
| v1 | `PRD_v1.md`, `PRD_v2.md` | `20260523-120000Z`, `20260523-174500-0400` |

### FSD (`PR-FSD-FTGO-001`)

| Versión prompt | Artefactos generados | Ejecuciones (Run ID) |
| --- | --- | --- |
| v1 | `FSD_v1.md` | `20260523-174000-0400` |

### ADR (`PR-ADR-FTGO-001`)

Un mismo prompt v1; cada decisión se materializa en un archivo `ADR-NNNN.md` (parámetro de la Task).

| Versión prompt | Parámetro | Artefacto | Run ID |
| --- | --- | --- | --- |
| v1 | descomposición / Strangler Fig | `ADR-0001.md` | `20260523-180000-0400-adr0001` |
| v1 | IPC predominante | `ADR-0002.md` | `20260523-180500-0400-adr0002` |
| v1 | datos / consistencia Pedido | `ADR-0003.md` | `20260523-181000-0400-adr0003` |

### C4 (`PR-C4-FTGO-001`)

| Versión prompt | Artefactos generados | Ejecuciones (Run ID) |
| --- | --- | --- |
| v1 | — | — (supersedido por prompt v2) |
| v2 | `C4_N1_N2_v1.md`, `c4_context.mmd`, `c4_container.mmd` | `20260523-190000-0400-c4` |

---

## 4. Resumen por artefacto (metadatos en cabecera)

| Artefacto | v artefacto | Prompt / v | Fecha última ejecución | Autor | Modelo | Temp. |
| --- | --- | --- | --- | --- | --- | ---: |
| [`PRD_v1.md`](prd/PRD_v1.md) | v1 | `PR-PRD-FTGO-001` / v1 | 2026-05-23T12:00:00Z | Módulo 4 - UMSS | Sonnet / Opus | 0.2 |
| [`PRD_v2.md`](prd/PRD_v2.md) | v2 | `PR-PRD-FTGO-001` / v1 | 2026-05-23T17:45:00-04:00 | Módulo 4 - UMSS | Sonnet / Opus | 0.2 |
| [`FSD_v1.md`](fsd/FSD_v1.md) | v1 | `PR-FSD-FTGO-001` / v1 | 2026-05-23T17:40:00-04:00 | Módulo 4 - UMSS | Sonnet | 0.2 |
| [`ADR-0001.md`](adr/ADR-0001.md) | — | `PR-ADR-FTGO-001` / v1 | 2026-05-23T18:00:00-04:00 | Módulo 4 - UMSS | Opus | 0.3 |
| [`ADR-0002.md`](adr/ADR-0002.md) | — | `PR-ADR-FTGO-001` / v1 | 2026-05-23T18:05:00-04:00 | Módulo 4 - UMSS | Opus | 0.3 |
| [`ADR-0003.md`](adr/ADR-0003.md) | — | `PR-ADR-FTGO-001` / v1 | 2026-05-23T18:10:00-04:00 | Módulo 4 - UMSS | Opus | 0.3 |
| [`C4_N1_N2_v1.md`](c4/C4_N1_N2_v1.md) | v1 | `PR-C4-FTGO-001` / v2 | 2026-05-23T19:00:00-04:00 | Módulo 4 - UMSS | Sonnet / Opus | 0.2 |

---

## 5. Cómo actualizar este registro

Al cerrar una ejecución con el skill **ejecutar-prompt-artefacto**:

1. Añadir fila en **§2 Registro de ejecuciones** (Run ID, fecha, artefacto, parámetro si es ADR).
2. Actualizar **§3 Matriz** y **§4 Resumen** si aplica.
3. Si existe nueva versión del prompt (`_v2`, `_v3`, …), actualizar **§1 Catálogo**.
4. Opcional: registrar **modelo real** de la corrida en la columna *Modelo (ejecución)* si la telemetría lo expone.

**Plantilla de fila (§2)**

```markdown
| `<run_id>` | `<ISO-8601>` | `<PR-XXX-FTGO-001>` | vN | `<notas>` | `[<area>/<file>.md](<area>/<file>.md)` | `<autor>` | `<modelo>` | 0.x | Generado |
```

---

## 6. Historial de cambios del mapeo

| Fecha | Autor | Cambio |
| --- | --- | --- |
| 2026-05-23 | Módulo 4 - UMSS | Creación inicial: PRD v1/v2, FSD v1, ADR 0001–0003; C4 pendiente. |
| 2026-05-23 | Módulo 4 - UMSS | Ejecución C4 con prompt v2: `C4_N1_N2_v1.md` + diagramas `.mmd`. |
