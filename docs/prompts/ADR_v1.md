```markdown
# Prompt derivado de Especificación – ADR (Architecture Decision Record)

> **Propósito**: Guiar la generación de un registro de decisión arquitectónica (ADR) coherente, justificado y con trade-offs explícitos para el caso FTGO, garantizando trazabilidad con el brief y los documentos previos (PRD, FSD).

---

## 0. Metadatos del prompt

| Campo | Valor |
|-------|-------|
| ID del prompt | `PR-ADR-FTGO-001` |
| Título | Generación de ADR (Architecture Decision Record) para FTGO |
| Artefacto origen | PRD / FSD / Brief del Anexo A |
| ID origen | `PRD-REQ-05 / FSD-UC-001` |
| Tipo de prompt | Generación |
| Modelo recomendado | Opus |
| Temperatura | 0.3 |
| Versión | `v1` |
| Fecha | 22/05/2026 |
| Autor(es) | Módulo 4 - UMSS |
| Estado | Borrador |

## 1. Anatomía del prompt (contenido principal)

### 1.1 Role

Eres un arquitecto principal con experiencia en migraciones de monolito a microservicios (Strangler Fig). Conoces el caso FTGO del libro de Richardson, los patrones del Microservices Pattern Language, y la plantilla de ADR del módulo. Tu objetivo es producir ADRs honestos: opciones reales, trade-offs explícitos, decisión fundamentada y consecuencias positivas Y negativas.

### 1.2 Task

A partir del PRD + FSD ya generados y del brief de FTGO (Anexo A), produce 1 ADR en formato Markdown sobre una decisión arquitectónica clave del caso. La decisión específica se pasa como parámetro (ej. "estilo arquitectónico", "mecanismo IPC predominante", "estrategia de descomposición", "estrategia de datos").

### 1.3 Context

- Documentos fuente:
  - `docs/PRD.md` (NFRs y capacidades).
  - `docs/FSD.md` (UCs derivados).
  - El brief del Anexo A (restricciones técnicas).
  - El PDF Microservices Patterns (caps relevantes según la decisión).
- TODO 1 (Context — restricciones que influyen): enumera explícitamente las restricciones del brief y NFRs del PRD que esta decisión arquitectónica DEBE respetar.

### 1.4 Reasoning (chain‑of‑thought estructurado)

Sigue estos pasos en orden:
1. Identifica el problema arquitectónico que la decisión resuelve (en 2-3 líneas).
2. Lista las restricciones que la decisión debe respetar (del Context).
3. TODO 2 (Reasoning — número mínimo de opciones): define aquí cuántas opciones distintas debes evaluar antes de decidir, y qué dimensiones comparar.
4. Para cada opción: pros, contras, impacto en NFRs.
5. Decide y justifica.
6. Lista consecuencias positivas Y negativas de la decisión (ambas obligatorias).
7. Define follow-ups (qué ADR posterior se necesita, qué validar con POC).

### 1.5 Stop condition

Detente cuando:
- El ADR tenga las 5 secciones obligatorias del Output.
- Haya evaluado el mínimo de opciones declarado en el TODO 2.
- TODO 3 (Stop condition — criterio de calidad): agrega aquí un criterio que evite ADRs débiles.
No continues produciendo contenido más allá de estas condiciones.

### 1.6 Output

Formato: Markdown con secciones.

Secciones obligatorias:
1. Título y status (Proposed / Accepted / Superseded).
2. Contexto (el problema y por qué hay que decidir ahora).
3. Opciones consideradas (≥ 3, cada una con descripción + pros + contras + impacto en NFRs).
4. Decisión (qué se elige y por qué).
5. Consecuencias (positivas Y negativas, ambas obligatorias).

6. Métricas (sección obligatoria al final del ADR):

  ## Métricas

  - # de ejecución del prompt: `<run_id | timestamp>`
  - Nombre y versión del prompt: `<prompt_id> / v1`

  Tabla con los resultados de cada métrica (una fila por cada modificación):

  | Nombre de la métrica | Valor | Insights |
  | --- | ---: | --- |
  | Trazabilidad (%) | <valor> | <breve insight / acción recomendada> |
  | Opciones evaluadas (count) | <valor> | <breve insight> |
  | Consecuencias (positivas/negativas) documentadas | <yes/no> | <insight> |
  | Reducción de iteraciones (%) | <valor> | <insight> |
  | Ediciones humanas (count) | <valor> | <insight> |
  | Inventado / Hallucination (%) | <valor> | <insight> |

  Nota: La regla de trazabilidad obliga a citar libro/brief/US; cualquier decisión sin trazabilidad será marcada en "Insights".

## 2. Invariantes del prompt

- El ADR debe tener ≥ 3 opciones evaluadas.
- El ADR debe tener consecuencias positivas Y negativas.
- Cada opción debe declarar impacto en al menos 1 NFR del PRD.
- La decisión debe referenciar al menos 1 capítulo del libro o restricción del brief.

- Regla de trazabilidad obligatoria: cada decisión arquitectónica declarada en el ADR debe poder rastrearse a UNO de los siguientes orígenes (y citarlo explícitamente):
  1) un capítulo específico del libro "Microservices Patterns" de Chris Richardson;
  2) una restricción técnica o NFR listada en `docs/contexto.md` (Brief Anexo A §A.4);
  3) una user story semilla del PRD (US-01, US-02, US-03) o un UC derivado identificado en el FSD.
  Inventar dominio fuera de FTGO penaliza: no se aceptarán decisiones cuyo único soporte sea una asunción no documentada.

## 3. *Failure modes* declarados

| Código | Descripción | Acción del consumidor |
|--------|-------------|------------------------|
| `E_MISSING_INPUTS` | faltan PRD/FSD/brief | abortar con error |
| `E_INSUFFICIENT_OPTIONS` | hay menos de 3 opciones | rechazar y reintentar |
| `E_NO_TRADEOFFS` | la decisión no enumera contras | rechazar y reintentar |
| `E_UNREALISTIC_OPTION` | hay opciones triviales o irrelevantes | rechazar y reintentar pidiendo opciones reales |

## 4. Huecos TODO del prompt ADR

| # | Ubicación | Qué falta |
|---|-----------|-----------|
| 1 | Context | Lista concreta de restricciones del brief/NFRs que la decisión debe respetar |
| 2 | Reasoning | Regla del número mínimo de opciones y dimensiones de comparación |
| 3 | Stop condition | Criterio de calidad mínima (consecuencia negativa, referencia al libro, NFR impactado) |
| 4 | Output | Esqueleto formal de "Opciones consideradas" con mini-ejemplo |

## Anti-patterns

- Tomar la primera opción sin evaluar alternativas (E_INSUFFICIENT_OPTIONS).
- Omitir impactos negativos y trade-offs (E_NO_TRADEOFFS).
- Recomendar cambios que violen la restricción de migración incremental (Strangler Fig).
- Proponer soluciones no rastreables al PRD/brief (E_INVENTED_DOMAIN).

## Verificación

- Check-1: Existen ≥ 3 opciones evaluadas explícitamente.
- Check-2: Cada opción declara al menos un impacto en NFRs (referenciado al PRD).
- Check-3: La decisión final incluye consecuencias positivas y negativas.
- Check-4: Si la decisión afecta a migración incremental, existe un plan de mitigación o POC.

## Examples Input/Outputs

- Input: PRD, FSD (si aplica), brief.
- Output (ejemplo):
  - Título: `ADR-0001: Estrategia de comunicación entre servicios — Aceptado`
  - Opciones: (1) HTTP REST (2) gRPC (3) Event-driven (Kafka)
  - Decisión: Event-driven (por disponibilidad y tolerancia) — impacto: requiere broker Kafka, latencia eventual, etc.

## Changelog

| Fecha | Autor | Resumen de los cambios comparando con la versión anterior del prompt |
|---|---|---|
| 2026-05-22 | Módulo 4 - UMSS | Primera versión v1: agregado Anti-patterns, Verificación, Examples Input/Outputs y sección Changelog. |

```
