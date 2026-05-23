# PRD — FTGO (Food To Go)

**Versión del artefacto:** `v1`  
**Origen:** [Brief Anexo A](docs/contexto.md) · Richardson, *Microservices Patterns* (Manning, 2019), Caps. 1–2  
**Prompt:** `PR-PRD-FTGO-001` / `v1`  
**Estado:** Borrador para laboratorio (entrada a FSD y ADRs)

---

## 1. Contexto y objetivos

FTGO es un marketplace de delivery de comida que hoy opera como **monolito Java (WAR)** con los síntomas típicos del “infierno monolítico” descritos en el Cap. 1 del libro de Richardson: builds lentos, escalado conflictivo, acoplamiento y riesgo de fallos en cascada. La dirección decidió **migrar hacia microservicios** de forma incremental (**Strangler Fig**), manteniendo el negocio en producción mientras se documenta y despliega la arquitectura objetivo.

**Objetivo de este PRD:** fijar de forma ligera (2–4 páginas equivalentes) el **contexto de producto**, los **stakeholders**, las **siete capacidades de negocio** del Cap. 2, los **NFRs trazables al brief** y el **alcance del laboratorio**, de modo que equipos puedan derivar un **FSD** (extendiendo [US-01](docs/contexto.md#us-01-toma-de-pedido-por-el-consumidor), [US-02](docs/contexto.md#us-02-aceptación-de-tickets-por-el-restaurante), [US-03](docs/contexto.md#us-03-asignación-de-entrega-al-courier)) y al menos **dos ADRs** sin inventar dominio fuera del Anexo A.

**Criterio cuantitativo de completitud (laboratorio):** ≥ **5** NFRs con **métrica explícita** y **referencia [Brief §A.4]**, las **7** capacidades del Cap. 2 con **un párrafo** cada una, y **6** stakeholders alineados a la tabla A.2 (sin roles adicionales).

---

## 2. Stakeholders

| Rol | Necesidad principal | Origen |
| --- | --- | --- |
| **Consumidor** | UX rápida, visibilidad del estado del pedido y tracking en tiempo real | [Brief §A.2](docs/contexto.md#a2-stakeholders) |
| **Restaurante** | Tickets gestionables, control de carga de cocina y dashboard de pedidos | [Brief §A.2](docs/contexto.md#a2-stakeholders) |
| **Courier** | Asignaciones cercanas, rutas razonables y pagos confiables | [Brief §A.2](docs/contexto.md#a2-stakeholders) |
| **Empleado FTGO (back office)** | Visibilidad operativa, reportes y resolución de incidentes | [Brief §A.2](docs/contexto.md#a2-stakeholders) |
| **Equipo de arquitectura** | Trazabilidad, calidad arquitectónica y mantenibilidad durante la migración | [Brief §A.2](docs/contexto.md#a2-stakeholders) |
| **Sistemas externos** | Integraciones estables (pago, mapas, notificaciones) con SLAs predecibles | [Brief §A.2](docs/contexto.md#a2-stakeholders) |

Las **user stories semilla** [US-01](docs/contexto.md#us-01-toma-de-pedido-por-el-consumidor), [US-02](docs/contexto.md#us-02-aceptación-de-tickets-por-el-restaurante) y [US-03](docs/contexto.md#us-03-asignación-de-entrega-al-courier) anclan los flujos prioritarios de toma de pedido, cocina/tickets y última milla.

---

## 3. Capacidades de negocio (Cap. 2 — Richardson)

Las siguientes capacidades son **candidatos a límites de servicio**; el grado de descomposición final se decide en ADRs/C4, no en este PRD [Brief §A.3](docs/contexto.md#a3-capacidades-de-negocio-cap-2-del-libro).

1. **Consumer Management** — Responsable del registro, perfiles, direcciones y preferencias del consumidor; habilita el descubrimiento de restaurantes y el checkout coherente con [US-01](docs/contexto.md#us-01-toma-de-pedido-por-el-consumidor).

2. **Restaurant Management** — Gestiona restaurantes registrados, menús, horarios y disponibilidad; soporta la validación de “restaurante abierto / ítem disponible” antes de confirmar pedidos [US-01](docs/contexto.md#us-01-toma-de-pedido-por-el-consumidor).

3. **Order Taking** — Valida carrito, totales, dirección y método de pago; emite **número de pedido único** y confirmación al consumidor, alineado con la aceptación de [US-01](docs/contexto.md#us-01-toma-de-pedido-por-el-consumidor).

4. **Order Fulfillment / Kitchen** — Convierte pedidos confirmados en **tickets** para cocina, con estados de preparación visibles; habilita aceptación/rechazo y ETA por parte del restaurante [US-02](docs/contexto.md#us-02-aceptación-de-tickets-por-el-restaurante).

5. **Delivery** — Asignación de couriers, rutas y **tracking en tiempo real**; incluye oferta de pedidos “listos para retirar” con timeout de aceptación [US-03](docs/contexto.md#us-03-asignación-de-entrega-al-courier).

6. **Billing & Accounting** — Cobros, comisiones y payouts; debe convivir con la restricción de **pedidos posibles con pasarela caída** (cola/retry) según el brief [Brief §A.4](docs/contexto.md#a4-restricciones-técnicas-y-nfrs-base).

7. **Notifications** — Confirmaciones, alertas de estado y recibos por canales acordados (email/SMS/push), enlazados a Stripe, Google Maps, SendGrid/Twilio como sistemas externos [Brief §A.2](docs/contexto.md#a2-stakeholders).

---

## 4. Requisitos no funcionales (NFRs)

Cada NFR incluye **métrica** y **origen** en A.4 del brief.

| ID | NFR | Métrica (objetivo) | Origen |
| --- | --- | --- | --- |
| **NFR-01** | **Latencia percibida (consumidor)** | **< 200 ms p95** en acciones críticas de la app (navegación/checkout según FSD) | [Brief §A.4 — Latencia UX](docs/contexto.md#a4-restricciones-técnicas-y-nfrs-base) |
| **NFR-02** | **Disponibilidad del flujo de toma de pedidos** | **≥ 99,9 %** mensual | [Brief §A.4 — Disponibilidad](docs/contexto.md#a4-restricciones-técnicas-y-nfrs-base) |
| **NFR-03** | **Disponibilidad del tracking en tiempo real** | **≥ 99,5 %** mensual (puede degradar respecto al flujo principal) | [Brief §A.4 — Disponibilidad](docs/contexto.md#a4-restricciones-técnicas-y-nfrs-base) |
| **NFR-04** | **Resiliencia a fallos de pasarela de pago** | El sistema **debe poder tomar pedidos** si la pasarela está caída, con **cola de retry**; definir en FSD el estado “pago pendiente” vs cancelación | [Brief §A.4 — Tolerancia a fallos externos](docs/contexto.md#a4-restricciones-técnicas-y-nfrs-base) |
| **NFR-05** | **Patrón de carga** | Soportar **pico ~5×** en ventanas **12:00–14:00** y **19:00–22:00** (hora local) sin violar NFR-01/NFR-02 en el diseño objetivo | [Brief §A.4 — Carga](docs/contexto.md#a4-restricciones-técnicas-y-nfrs-base) |
| **NFR-06** | **Escalabilidad** | Componentes diseñados para **escalado horizontal** (Scale Cube: X/Y según ADR) | [Brief §A.4 — Escalabilidad horizontal](docs/contexto.md#a4-restricciones-técnicas-y-nfrs-base) |
| **NFR-07** | **Consistencia de datos** | **Consistencia fuerte** dentro del **aggregate del pedido**; **eventual** aceptada para reporting/analytics | [Brief §A.4 — Consistencia de datos](docs/contexto.md#a4-restricciones-técnicas-y-nfrs-base) |
| **NFR-08** | **Trazabilidad operativa** | **100 %** de acciones del consumidor con **correlation ID** y trazas distribuibles en diseño objetivo | [Brief §A.4 — Trazabilidad](docs/contexto.md#a4-restricciones-técnicas-y-nfrs-base) |

> **Nota:** PCI-DSS (delegado a Stripe) y GDPR/locales se mantienen como restricciones de cumplimiento [Brief §A.4 — Cumplimiento](docs/contexto.md#a4-restricciones-técnicas-y-nfrs-base); el detalle legal queda fuera del alcance de este PRD ligero.

---

## 5. Alcance

### 5.1 Dentro del alcance (laboratorio / producto)

- Documentar la **visión de migración incremental** (Strangler Fig) coexistiendo con el **monolito legacy** durante **18–24 meses** [Brief §A.4 — Migración incremental](docs/contexto.md#a4-restricciones-técnicas-y-nfrs-base).
- Cubrir las **7 capacidades** del Cap. 2 como marco de requisitos y límites candidatos [Brief §A.3](docs/contexto.md#a3-capacidades-de-negocio-cap-2-del-libro).
- Anclar requisitos a **[US-01](docs/contexto.md#us-01-toma-de-pedido-por-el-consumidor), [US-02](docs/contexto.md#us-02-aceptación-de-tickets-por-el-restaurante), [US-03](docs/contexto.md#us-03-asignación-de-entrega-al-courier)** y extensión en FSD a **≥ 5 UCs** según [Brief §A.5](docs/contexto.md#a5-user-stories-semilla).
- Preferencia declarada: **Java / Spring Boot** en el núcleo para reutilizar conocimiento del equipo [Brief §A.4 — Tecnología](docs/contexto.md#a4-restricciones-técnicas-y-nfrs-base).

### 5.2 Fuera del alcance (en esta fase del PRD)

- Diseño detallado de APIs, esquemas de BD, colas o despliegue (pertenece al **FSD** y a **ADRs**).
- Especificación exhaustiva de todos los flujos del marketplace (p. ej. marketing, CRM avanzado) no citados en el Anexo A.
- Implementación de código o pruebas automatizadas.

---

## Métricas

### Ejecución 1 — 2026-05-23T12:00:00Z

- **Prompt:** `PR-PRD-FTGO-001` / `v1`
- **Run ID:** `20260523-120000Z`

| Nombre de la métrica | Valor | Insights |
| --- | ---: | --- |
| Completitud del output (%) | 100 | Cinco secciones obligatorias del PRD + NFRs + alcance + métricas presentes. |
| % Secciones cubiertas | 100 | Misma base que completitud (secciones canónicas del prompt). |
| Cobertura NFRs (%) | 100 | 8 NFRs explícitos; los 8 citan [Brief §A.4] (sub-bullets del brief). |
| UCs entregados (count) | N/A | El PRD no define UCs; el FSD debe extender a ≥ 5 UCs [Brief §A.5]. |
| % UCs con Given/When/Then | N/A | Corresponde al FSD, no al PRD. |
| Sintaxis Mermaid válida | N/A | No aplica al PRD. |
| Trazabilidad (%) | 100 | Stakeholders y capacidades anclados a A.2/A.3; NFRs a A.4; stories a A.5. |
| Reducción de iteraciones (%) | 0 | Primera ejecución del artefacto; sin baseline de iteraciones previas. |
| Tiempo hasta convergencia (min) | — | No medido en pipeline; completitud verificable por checklist del prompt. |
| Ediciones humanas (count) | 0 | Artefacto generado; sin commits posteriores. |
| Inventado / Hallucination (%) | 0 | Sin stakeholders, capacidades ni NFRs fuera del Anexo A. |
