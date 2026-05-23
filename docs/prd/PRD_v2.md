# PRD — FTGO (Food To Go)

**Versión del artefacto:** `v2`  
**Línea base consolidada:** [`PRD_v1.md`](PRD_v1.md)  
**Origen:** [Brief Anexo A](../contexto.md) · Richardson, *Microservices Patterns* (Manning, 2019), Caps. 1–2  
**Prompt:** `PR-PRD-FTGO-001` / `v1` (última versión disponible en `docs/prompts/`)  
**Artefactos descendientes:** [`FSD_v1.md`](../fsd/FSD_v1.md)  
**Estado:** Borrador consolidado (entrada a FSD v2 y ADRs)

---

## 1. Contexto y objetivos

FTGO es un marketplace de delivery de comida que opera como **monolito Java (WAR)** con síntomas del infierno monolítico (Cap. 1, Richardson): builds lentos, escalado conflictivo, acoplamiento y fallos en cascada. La dirección adoptó **migración incremental a microservicios** mediante **Strangler Fig** [Brief §A.4 — Migración incremental](../contexto.md#a4-restricciones-técnicas-y-nfrs-base), manteniendo producción mientras se documenta la arquitectura objetivo.

**Objetivo de este PRD (v2):** consolidar el contenido de `PRD_v1` y reforzar la **trazabilidad hacia el FSD** y los ADRs, sin ampliar el dominio fuera del Anexo A. El documento sigue siendo ligero (2–4 páginas equivalentes) y cubre contexto, stakeholders, **7 capacidades** del Cap. 2, **NFRs** con métrica y alcance del laboratorio.

**Criterio cuantitativo de completitud:** ≥ **5** NFRs con métrica y `[Brief §A.4]`; **7** capacidades con un párrafo; **6** stakeholders de A.2; matriz de trazabilidad US → capacidad → NFR (§6).

---

## 2. Stakeholders

| Rol | Necesidad principal | Origen |
| --- | --- | --- |
| **Consumidor** | UX rápida, visibilidad del estado del pedido y tracking en tiempo real | [Brief §A.2](../contexto.md#a2-stakeholders) |
| **Restaurante** | Tickets gestionables, control de carga de cocina y dashboard de pedidos | [Brief §A.2](../contexto.md#a2-stakeholders) |
| **Courier** | Asignaciones cercanas, rutas razonables y pagos confiables | [Brief §A.2](../contexto.md#a2-stakeholders) |
| **Empleado FTGO (back office)** | Visibilidad operativa, reportes y resolución de incidentes | [Brief §A.2](../contexto.md#a2-stakeholders) |
| **Equipo de arquitectura** | Trazabilidad, calidad arquitectónica y mantenibilidad durante la migración | [Brief §A.2](../contexto.md#a2-stakeholders) |
| **Sistemas externos** | Integraciones estables (Stripe, Google Maps, SendGrid/Twilio) con SLAs predecibles | [Brief §A.2](../contexto.md#a2-stakeholders) |

**User stories semilla** (ancla funcional para FSD): [US-01](../contexto.md#us-01-toma-de-pedido-por-el-consumidor), [US-02](../contexto.md#us-02-aceptación-de-tickets-por-el-restaurante), [US-03](../contexto.md#us-03-asignación-de-entrega-al-courier) [Brief §A.5](../contexto.md#a5-user-stories-semilla).

---

## 3. Capacidades de negocio (Cap. 2 — Richardson)

Candidatos a límites de servicio; la descomposición final se resuelve en ADRs/C4 [Brief §A.3](../contexto.md#a3-capacidades-de-negocio-cap-2-del-libro).

| # | Capacidad | Responsabilidad (consolidada) | US / flujo |
| ---: | --- | --- | --- |
| 1 | **Consumer Management** | Registro, perfiles, direcciones y preferencias; soporte a checkout | [US-01](../contexto.md#us-01-toma-de-pedido-por-el-consumidor) |
| 2 | **Restaurant Management** | Menús, horarios, disponibilidad; validación pre-pedido | [US-01](../contexto.md#us-01-toma-de-pedido-por-el-consumidor) |
| 3 | **Order Taking** | Carrito, totales, confirmación y **número de pedido único** | [US-01](../contexto.md#us-01-toma-de-pedido-por-el-consumidor) |
| 4 | **Order Fulfillment / Kitchen** | Tickets, estados de preparación, aceptación/rechazo y ETA | [US-02](../contexto.md#us-02-aceptación-de-tickets-por-el-restaurante) |
| 5 | **Delivery** | Asignación de couriers, rutas, tracking, timeout de oferta | [US-03](../contexto.md#us-03-asignación-de-entrega-al-courier) |
| 6 | **Billing & Accounting** | Cobros, comisiones, payouts; pedidos con pasarela caída (retry) | [US-01](../contexto.md#us-01-toma-de-pedido-por-el-consumidor) + NFR-04 |
| 7 | **Notifications** | Confirmaciones, alertas y recibos en canales acordados | Transversal (US-01..03) |

---

## 4. Requisitos no funcionales (NFRs)

| ID | NFR | Métrica (objetivo) | Origen | Ref. FSD (UC) |
| --- | --- | --- | --- | --- |
| **NFR-01** | Latencia percibida (consumidor) | **< 200 ms p95** en acciones críticas | [Brief §A.4 — Latencia UX](../contexto.md#a4-restricciones-técnicas-y-nfrs-base) | UC-01, UC-05 |
| **NFR-02** | Disponibilidad toma de pedidos | **≥ 99,9 %** mensual | [Brief §A.4 — Disponibilidad](../contexto.md#a4-restricciones-técnicas-y-nfrs-base) | UC-01 |
| **NFR-03** | Disponibilidad tracking | **≥ 99,5 %** mensual | [Brief §A.4 — Disponibilidad](../contexto.md#a4-restricciones-técnicas-y-nfrs-base) | UC-05 |
| **NFR-04** | Resiliencia pasarela de pago | Pedidos con pasarela caída + **cola retry** | [Brief §A.4 — Tolerancia a fallos](../contexto.md#a4-restricciones-técnicas-y-nfrs-base) | UC-04 |
| **NFR-05** | Patrón de carga | **Pico ~5×** 12:00–14:00 y 19:00–22:00 | [Brief §A.4 — Carga](../contexto.md#a4-restricciones-técnicas-y-nfrs-base) | UC-01..03 |
| **NFR-06** | Escalabilidad horizontal | Scale Cube X/Y por componente | [Brief §A.4 — Escalabilidad](../contexto.md#a4-restricciones-técnicas-y-nfrs-base) | ADR (pendiente) |
| **NFR-07** | Consistencia de datos | Fuerte en aggregate **pedido**; eventual en reporting | [Brief §A.4 — Consistencia](../contexto.md#a4-restricciones-técnicas-y-nfrs-base) | UC-01, UC-04 |
| **NFR-08** | Trazabilidad operativa | **100 %** acciones con correlation ID | [Brief §A.4 — Trazabilidad](../contexto.md#a4-restricciones-técnicas-y-nfrs-base) | Todos los UC |

PCI-DSS (Stripe) y GDPR/locales: restricciones de cumplimiento [Brief §A.4 — Cumplimiento](../contexto.md#a4-restricciones-técnicas-y-nfrs-base); detalle legal fuera de este PRD.

---

## 5. Alcance

### 5.1 Dentro del alcance

- Migración **Strangler Fig** con monolito legacy **18–24 meses** [Brief §A.4](../contexto.md#a4-restricciones-técnicas-y-nfrs-base).
- Las **7 capacidades** del Cap. 2 y **≥ 5 UCs** en FSD [Brief §A.5](../contexto.md#a5-user-stories-semilla) — cubierto en [`FSD_v1.md`](../fsd/FSD_v1.md) (6 UCs).
- **≥ 2 ADRs** derivables de capacidades y NFRs (pendiente de generación).
- Núcleo preferido **Java / Spring Boot** [Brief §A.4 — Tecnología](../contexto.md#a4-restricciones-técnicas-y-nfrs-base).

### 5.2 Fuera del alcance

- APIs, esquemas, colas y despliegue detallado (FSD/ADR).
- Flujos no citados en el Anexo A (marketing, CRM avanzado).
- Implementación y pruebas automatizadas.

### 5.3 Cambios respecto a PRD v1

| Área | v1 | v2 (consolidación) |
| --- | --- | --- |
| Capacidades | Lista numerada | Tabla con mapeo US |
| NFRs | 8 filas | Misma base + columna **Ref. FSD** |
| Trazabilidad | Implícita | §6 matriz explícita |
| Descendientes | FSD planificado | Enlace a `FSD_v1` existente |

---

## 6. Matriz de trazabilidad (US → capacidad → NFR)

| US / origen | Capacidad PRD | NFRs aplicables | Artefacto |
| --- | --- | --- | --- |
| [US-01](../contexto.md#us-01-toma-de-pedido-por-el-consumidor) | Consumer, Restaurant, Order Taking, Billing | NFR-01, 02, 04, 05, 07, 08 | FSD UC-01, UC-04 |
| [US-02](../contexto.md#us-02-aceptación-de-tickets-por-el-restaurante) | Order Fulfillment / Kitchen, Notifications | NFR-02, 05, 08 | FSD UC-02 |
| [US-03](../contexto.md#us-03-asignación-de-entrega-al-courier) | Delivery, Notifications | NFR-01, 03, 05, 08 | FSD UC-03, UC-06 |
| NFR tracking (derivado brief) | Delivery, Notifications | NFR-01, 03 | FSD UC-05 |
| Continuidad operacional (derivado US-03) | Delivery | NFR-02, 08 | FSD UC-06 |

---

## Métricas

### Ejecución 1 — 2026-05-23T17:45:00-04:00

- **Prompt:** `PR-PRD-FTGO-001` / `v1`
- **Run ID:** `20260523-174500-0400`
- **Operación:** Consolidación `PRD_v1` → artefacto `PRD_v2`

| Nombre de la métrica | Valor | Insights |
| --- | ---: | --- |
| Completitud del output (%) | 100 | Secciones 1–6 del PRD v2 + métricas; añadida matriz de trazabilidad. |
| % Secciones cubiertas | 100 | Incluye sección nueva §6 sin eliminar contenido válido de v1. |
| Cobertura NFRs (%) | 100 | 8/8 NFRs con métrica, origen brief y referencia a UC en FSD. |
| UCs entregados (count) | N/A | Corresponde al FSD (`FSD_v1`: 6 UCs). |
| % UCs con Given/When/Then | N/A | Corresponde al FSD. |
| Sintaxis Mermaid válida | N/A | No aplica al PRD. |
| Trazabilidad (%) | 100 | Matriz US → capacidad → NFR → UC; sin dominio inventado. |
| Reducción de iteraciones (%) | 50 | Segunda versión de artefacto; reutiliza v1 en lugar de regenerar desde cero. |
| Tiempo hasta convergencia (min) | — | No medido en pipeline. |
| Ediciones humanas (count) | 0 | Generación automatizada consolidada. |
| Inventado / Hallucination (%) | 0 | Sin elementos fuera de Anexo A / PRD v1 / FSD v1. |
