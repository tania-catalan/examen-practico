# Examen Práctico — Arquitectura de Microservicios FTGO

> **Maestría en Inteligencia Artificial · Módulo 4 — UMSS**  
> Laboratorio de documentación arquitectónica con **prompt engineering** aplicado al caso **FTGO** (Food To Go).

---

## Descripción del proyecto

Este repositorio contiene los **artefactos de arquitectura** generados mediante prompts estructurados para documentar la migración del monolito FTGO a microservicios, basándose en el libro *Microservices Patterns* de Chris Richardson (Manning, 2019).

El objetivo no es construir el sistema, sino **documentar la arquitectura objetivo** para que los equipos de desarrollo puedan ejecutar la migración incremental (**Strangler Fig**) durante 18–24 meses.

### ¿Qué es FTGO?

FTGO es un **marketplace de delivery de comida** que conecta consumidores, restaurantes y couriers. Opera como un monolito Java (WAR) con los síntomas clásicos del *infierno monolítico*: builds lentos, escalado conflictivo, falta de aislamiento de fallos y lock-in tecnológico.

---

## Estructura del repositorio

```
examen-practico/
│
├── docs/                          # Documentación del proyecto
│   ├── contexto.md                # Brief del caso FTGO (Anexo A) — fuente canónica del dominio
│   ├── PROMPT_MAPPING.md          # Registro de prompts ejecutados y artefactos generados
│   ├── METRICS.md                 # Definición de métricas de calidad por tipo de documento
│   │
│   ├── prd/                       # Product Requirements Document
│   │   ├── PRD_v1.md              # PRD — primera generación
│   │   └── PRD_v2.md              # PRD — consolidación v2 (con matriz de trazabilidad)
│   │
│   ├── fsd/                       # Functional Specification Document
│   │   └── FSD_v1.md              # FSD — 6 casos de uso con Given/When/Then
│   │
│   ├── adr/                       # Architecture Decision Records
│   │   ├── ADR-0001.md            # Estrategia de descomposición (Strangler Fig)
│   │   ├── ADR-0002.md            # Mecanismo IPC predominante
│   │   └── ADR-0003.md            # Estrategia de datos / consistencia del pedido
│   │
│   ├── c4/                        # Diagramas C4
│   │   ├── C4_N1_N2_v1.md         # Documento C4 Nivel 1 + Nivel 2
│   │   ├── c4_context.mmd         # Diagrama de contexto (Mermaid)
│   │   └── c4_container.mmd       # Diagrama de contenedores (Mermaid)
│   │
│   ├── prompts/                   # Prompts ejecutables (plantillas + versiones)
│   │   ├── PRD.md                 # Plantilla semilla PRD
│   │   ├── PRD_v1.md              # Prompt ejecutable PRD v1
│   │   ├── FSD.md                 # Plantilla semilla FSD
│   │   ├── FSD_v1.md              # Prompt ejecutable FSD v1
│   │   ├── ADR.md                 # Plantilla semilla ADR
│   │   ├── ADR_v1.md              # Prompt ejecutable ADR v1
│   │   ├── C4_N1_N2.md            # Plantilla semilla C4
│   │   ├── C4_N1_N2_v1.md         # Prompt ejecutable C4 v1
│   │   ├── PROMPT.md              # Guía de estructura de prompts
│   │   └── SKILL.md               # Prompt del skill de ejecución
│   │
│   └── diagrams/                  # (Reservado para diagramas adicionales)
│
├── prompts_mejorados/             # Prompts mejorados (huecos TODO rellenados)
│   ├── prd_mejorado.md            # PRD prompt con TODOs resueltos
│   ├── fsd_mejorado.md            # FSD prompt con TODOs resueltos
│   ├── adr_mejorado.md            # ADR prompt con TODOs resueltos
│   └── C4_mejorado.md             # C4 prompt con TODOs resueltos
│
├── agents/                        # Configuración de agentes IA
│   └── skills/
│       └── SKILL.md               # Skill: ejecutar-prompt-artefacto
│
└── .gitignore
```

---

## Artefactos generados

### 📋 PRD (Product Requirements Document)

Define el contexto, stakeholders, las 7 capacidades de negocio del Cap. 2 de Richardson, 8 NFRs con métricas y el alcance de la migración.

| Versión | Archivo | Contenido clave |
|---------|---------|-----------------|
| v1 | [`PRD_v1.md`](docs/prd/PRD_v1.md) | Primera generación: stakeholders, capacidades, NFRs |
| v2 | [`PRD_v2.md`](docs/prd/PRD_v2.md) | Consolidación con matriz de trazabilidad US → capacidad → NFR |

### 📝 FSD (Functional Specification Document)

Especificación funcional con **6 casos de uso** formalizados con bloques **Given/When/Then** (BDD), derivados de las 3 user stories semilla del brief.

| Versión | Archivo | UCs |
|---------|---------|-----|
| v1 | [`FSD_v1.md`](docs/fsd/FSD_v1.md) | 6 UCs: toma de pedido, tickets, courier, pago, tracking, reasignación |

### 🏗️ ADRs (Architecture Decision Records)

Tres decisiones arquitectónicas clave con ≥ 3 opciones evaluadas, trade-offs explícitos y consecuencias positivas/negativas.

| ADR | Archivo | Decisión |
|-----|---------|----------|
| ADR-0001 | [`ADR-0001.md`](docs/adr/ADR-0001.md) | Estrategia de descomposición (Strangler Fig) |
| ADR-0002 | [`ADR-0002.md`](docs/adr/ADR-0002.md) | Mecanismo IPC predominante |
| ADR-0003 | [`ADR-0003.md`](docs/adr/ADR-0003.md) | Estrategia de datos y consistencia del pedido |

### 📐 Diagramas C4 (Nivel 1 y Nivel 2)

Diagramas en **Mermaid** siguiendo el modelo C4 de Simon Brown.

| Nivel | Archivo | Contenido |
|-------|---------|-----------|
| Narrativa | [`C4_N1_N2_v1.md`](docs/c4/C4_N1_N2_v1.md) | Documento descriptivo de ambos niveles |
| N1 — Contexto | [`c4_context.mmd`](docs/c4/c4_context.mmd) | FTGO + personas + sistemas externos |
| N2 — Contenedores | [`c4_container.mmd`](docs/c4/c4_container.mmd) | Microservicios, BDs, broker, protocolos |

---

## Prompts y prompt engineering

### Flujo de trabajo

```
Plantilla semilla (PRD.md)
        │
        ▼
Prompt ejecutable (PRD_v1.md)   ──►   Artefacto generado (docs/prd/PRD_v1.md)
        │                                       │
        ▼                                       ▼
Prompt mejorado (prd_mejorado.md)       Consolidación (docs/prd/PRD_v2.md)
```

### Catálogo de prompts

| Tipo | Prompt ejecutable | Modelo | Temp. | Salida |
|------|-------------------|--------|------:|--------|
| **PRD** | [`PRD_v1.md`](docs/prompts/PRD_v1.md) | Sonnet / Opus | 0.2 | `docs/prd/` |
| **FSD** | [`FSD_v1.md`](docs/prompts/FSD_v1.md) | Sonnet | 0.2 | `docs/fsd/` |
| **ADR** | [`ADR_v1.md`](docs/prompts/ADR_v1.md) | Opus | 0.3 | `docs/adr/` |
| **C4** | [`C4_N1_N2_v1.md`](docs/prompts/C4_N1_N2_v1.md) | Sonnet / Opus | 0.2 | `docs/c4/` |

### Prompts mejorados

Los prompts mejorados en [`prompts_mejorados/`](prompts_mejorados/) resuelven los **huecos TODO** de las plantillas semilla, incluyendo:

- Listas explícitas de contexto (stakeholders, UCs, restricciones)
- Reglas de granularidad y criterios cuantitativos
- Esqueletos formales de output con mini-ejemplos
- Failure modes y secciones de Changelog

---

## Skill del agente: `ejecutar-prompt-artefacto`

El skill [`SKILL.md`](agents/skills/SKILL.md) automatiza la ejecución de prompts versionados con estas reglas:

1. **Consolidación incremental** — No regenera todo; fusiona secciones existentes con nuevo contenido.
2. **Métricas acumulativas** — Cada ejecución agrega una subsección inmutable sin borrar las anteriores.
3. **Versionado automático** — Resuelve el último `_vN` del prompt y alinea el artefacto de salida.
4. **Actualización del registro** — Mantiene [`PROMPT_MAPPING.md`](docs/PROMPT_MAPPING.md) sincronizado.

---

## Métricas de calidad

Definidas en [`METRICS.md`](docs/METRICS.md), se evalúan por ejecución y por tipo de documento:

| Métrica | PRD | FSD | C4 | ADR |
|---------|:---:|:---:|:--:|:---:|
| Completitud del output (%) | ✓ | ✓ | ✓ | ✓ |
| Cobertura NFRs (%) | ✓ | — | — | — |
| UCs entregados | — | ✓ | — | — |
| % UCs con Given/When/Then | — | ✓ | — | — |
| Trazabilidad (%) | ✓ | ✓ | ✓ | ✓ |
| Sintaxis Mermaid válida | — | — | ✓ | — |
| Inventado / Hallucination (%) | ✓ | ✓ | ✓ | ✓ |

**Umbrales sugeridos:** completitud ≥ 90%, trazabilidad = 100% (ADR) / ≥ 90% (PRD/FSD), invención ≤ 5%.

---

## Restricciones del laboratorio

- **Fuente única del dominio**: Brief del Anexo A ([`contexto.md`](docs/contexto.md)) + libro de Richardson.
- **Trazabilidad obligatoria**: cada decisión cita su origen (`[Brief §A.X]`, `Cap. Y`, `US-NN`).
- **Migración incremental**: el monolito sigue vivo; se aplica **Strangler Fig** durante 18–24 meses.
- **Tecnología preferida**: Java / Spring Boot en el core; libertad en servicios satélite.
- **Granularidad apropiada**: documentos ligeros, no exhaustivos.

---

## Referencias

- 📖 Richardson, C. (2019). *Microservices Patterns*. Manning Publications.
- 🔗 [Repositorio oficial FTGO](https://github.com/microservices-patterns/ftgo-application)
- 🌐 [Microservices Pattern Language](https://microservices.io/)

---

## Autora

**Tania Catalán** — Maestría en Inteligencia Artificial, Módulo 4 — UMSS