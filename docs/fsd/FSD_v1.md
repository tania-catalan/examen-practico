# FSD — FTGO (Functional Specification Document)

**Versión del artefacto:** `v1`  
**Origen:** `docs/prd/PRD_v1.md` + [Brief Anexo A](docs/contexto.md)  
**Prompt:** `PR-FSD-FTGO-001` / `v1`  
**Estado:** Borrador para laboratorio

---

## 1. Introducción

Este FSD define los casos de uso funcionales mínimos para FTGO en el contexto de migración incremental Strangler Fig, tomando como base el PRD v1 y las user stories semilla [US-01](docs/contexto.md#us-01-toma-de-pedido-por-el-consumidor), [US-02](docs/contexto.md#us-02-aceptación-de-tickets-por-el-restaurante) y [US-03](docs/contexto.md#us-03-asignación-de-entrega-al-courier). Se incluyen 6 UCs completos con trazabilidad a capacidades de negocio y criterios Given/When/Then.

## 2. Tabla de UCs

| ID | Título | Actor primario | Capacidad PRD | Origen |
| --- | --- | --- | --- | --- |
| UC-01 | Tomar pedido del consumidor | Consumidor | Order Taking | US-01 [Brief §A.5](docs/contexto.md#a5-user-stories-semilla) |
| UC-02 | Aceptar o rechazar ticket en restaurante | Restaurante | Order Fulfillment / Kitchen | US-02 [Brief §A.5](docs/contexto.md#a5-user-stories-semilla) |
| UC-03 | Asignar entrega a courier | Courier | Delivery | US-03 [Brief §A.5](docs/contexto.md#a5-user-stories-semilla) |
| UC-04 | Procesar pago en checkout | Consumidor | Billing & Accounting | Derivado de US-01 + NFR tolerancia a fallos [Brief §A.4](docs/contexto.md#a4-restricciones-técnicas-y-nfrs-base) |
| UC-05 | Tracking en tiempo real del pedido | Consumidor | Delivery + Notifications | Derivado de NFR latencia/disponibilidad [Brief §A.4](docs/contexto.md#a4-restricciones-técnicas-y-nfrs-base) |
| UC-06 | Reasignación automática por rechazo/timeout | Sistema FTGO | Delivery | Derivado de US-03 + continuidad operacional [Brief §A.5](docs/contexto.md#a5-user-stories-semilla) |

## 3. Detalle de Casos de Uso

### UC-01: Tomar pedido del consumidor

| Campo | Valor |
| --- | --- |
| Actor primario | Consumidor |
| Actores secundarios | Sistema FTGO, Restaurante, Pasarela de pago |
| Capacidad PRD | Order Taking |
| Origen | US-01 [Brief §A.5](docs/contexto.md#a5-user-stories-semilla) |

**Precondiciones**
- Consumidor autenticado y con dirección de entrega válida.
- Restaurante en estado abierto y con menú disponible.

**Flujo principal**
1. El consumidor selecciona restaurante y visualiza menú.
2. Agrega o quita ítems del carrito.
3. Confirma dirección y método de pago.
4. El sistema valida disponibilidad y calcula total.
5. El sistema crea pedido con identificador único y estado `PENDING_CONFIRMATION`.
6. Se envía confirmación al consumidor.

**Flujos alternativos**
- A1: Ítem sin stock ? se informa al consumidor y se solicita ajustar carrito.
- A2: Restaurante cerrado ? se bloquea confirmación y se sugiere otro horario.

**Postcondiciones**
- Pedido registrado y trazable con correlation ID.

**Given/When/Then**
- **Given** el consumidor tiene un carrito con al menos un ítem disponible.
- **When** confirma el pedido con dirección y método de pago.
- **Then** el sistema crea un pedido con número único y devuelve confirmación.

---

### UC-02: Aceptar o rechazar ticket en restaurante

| Campo | Valor |
| --- | --- |
| Actor primario | Restaurante |
| Actores secundarios | Sistema FTGO, Consumidor |
| Capacidad PRD | Order Fulfillment / Kitchen |
| Origen | US-02 [Brief §A.5](docs/contexto.md#a5-user-stories-semilla) |

**Precondiciones**
- Existe pedido confirmado pendiente de revisión por cocina.
- Restaurante conectado a su dashboard.

**Flujo principal**
1. El dashboard muestra ticket entrante.
2. El restaurante revisa carga y tiempo estimado.
3. Acepta el ticket e indica ETA.
4. El sistema actualiza estado a `ACCEPTED_BY_RESTAURANT`.
5. Se notifica al consumidor.

**Flujos alternativos**
- A1: Restaurante rechaza con motivo ? pedido pasa a `REJECTED_BY_RESTAURANT`.
- A2: No hay respuesta en ventana operativa ? se aplica política de vencimiento y notificación.

**Postcondiciones**
- Pedido aceptado con ETA o rechazado con motivo registrado.

**Given/When/Then**
- **Given** el restaurante recibe un ticket nuevo.
- **When** acepta el ticket y define tiempo de preparación.
- **Then** el sistema actualiza estado del pedido y notifica al consumidor.

---

### UC-03: Asignar entrega a courier

| Campo | Valor |
| --- | --- |
| Actor primario | Courier |
| Actores secundarios | Sistema FTGO, Restaurante, Consumidor |
| Capacidad PRD | Delivery |
| Origen | US-03 [Brief §A.5](docs/contexto.md#a5-user-stories-semilla) |

**Precondiciones**
- Pedido en estado `READY_FOR_PICKUP`.
- Al menos un courier disponible en radio configurado.

**Flujo principal**
1. El sistema identifica couriers cercanos.
2. Publica oferta de entrega al courier candidato.
3. El courier acepta dentro del timeout.
4. El sistema asigna la entrega.
5. Se muestra ruta al restaurante y luego al consumidor.

**Flujos alternativos**
- A1: Courier rechaza ? pasa al siguiente candidato.
- A2: Timeout de respuesta (30 s) ? reasignación automática.

**Postcondiciones**
- Entrega asignada a courier con ruta activa.

**Given/When/Then**
- **Given** existe un pedido listo para retiro y un courier disponible.
- **When** el courier acepta la oferta antes del timeout.
- **Then** el sistema asigna el pedido y habilita la ruta de entrega.

---

### UC-04: Procesar pago en checkout

| Campo | Valor |
| --- | --- |
| Actor primario | Consumidor |
| Actores secundarios | Sistema FTGO, Pasarela Stripe |
| Capacidad PRD | Billing & Accounting |
| Origen | Derivado de US-01 + NFR tolerancia a fallos [Brief §A.4](docs/contexto.md#a4-restricciones-técnicas-y-nfrs-base) |

**Precondiciones**
- Pedido creado en checkout con total calculado.
- Método de pago seleccionado.

**Flujo principal**
1. El sistema envía intento de cobro a la pasarela.
2. La pasarela responde aprobación.
3. El sistema marca pedido como `PAID`.
4. Se emite confirmación y recibo.

**Flujos alternativos**
- A1: Pasarela caída/timeout ? pedido continúa como `PAYMENT_PENDING` con cola de reintento.
- A2: Pago rechazado por emisor ? se solicita nuevo método de pago.

**Postcondiciones**
- Pago confirmado o registrado para retry sin perder pedido.

**Given/When/Then**
- **Given** hay un pedido válido y un método de pago registrado.
- **When** el sistema intenta autorizar el cobro.
- **Then** marca el pedido como pagado o como pendiente de reintento según la respuesta.

---

### UC-05: Tracking en tiempo real del pedido

| Campo | Valor |
| --- | --- |
| Actor primario | Consumidor |
| Actores secundarios | Sistema FTGO, Courier |
| Capacidad PRD | Delivery + Notifications |
| Origen | Derivado de NFR latencia/disponibilidad [Brief §A.4](docs/contexto.md#a4-restricciones-técnicas-y-nfrs-base) |

**Precondiciones**
- Pedido asignado a courier.
- Dispositivo del consumidor con sesión activa.

**Flujo principal**
1. El sistema recibe eventos de ubicación del courier.
2. Actualiza estado y ETA del pedido.
3. El consumidor visualiza progreso en la app.
4. Se envían notificaciones en hitos relevantes.

**Flujos alternativos**
- A1: Degradación temporal de mapas ? mostrar estado textual y último punto válido.
- A2: Pérdida de conectividad del courier ? conservar último estado con alerta de actualización diferida.

**Postcondiciones**
- Consumidor mantiene visibilidad continua del estado del pedido.

**Given/When/Then**
- **Given** el pedido está en reparto y existen eventos de ubicación.
- **When** el courier cambia de estado o posición relevante.
- **Then** el consumidor ve la actualización y ETA en la app.

---

### UC-06: Reasignación automática por rechazo o timeout

| Campo | Valor |
| --- | --- |
| Actor primario | Sistema FTGO |
| Actores secundarios | Courier, Consumidor, Back office |
| Capacidad PRD | Delivery |
| Origen | Derivado de US-03 + continuidad operacional [Brief §A.5](docs/contexto.md#a5-user-stories-semilla) |

**Precondiciones**
- Pedido requiere courier y no quedó asignado tras intento previo.

**Flujo principal**
1. El sistema detecta rechazo o timeout de courier.
2. Recalcula candidatos por proximidad y disponibilidad.
3. Publica nueva oferta de entrega.
4. Un nuevo courier acepta.
5. Se actualiza estado y se notifica al consumidor.

**Flujos alternativos**
- A1: Sin couriers disponibles en radio inicial ? expandir radio y elevar alerta operativa.
- A2: Se supera número de intentos ? escalar a back office para gestión manual.

**Postcondiciones**
- Pedido reasignado o escalado sin pérdida de trazabilidad.

**Given/When/Then**
- **Given** un pedido pierde su asignación por rechazo o timeout.
- **When** el sistema ejecuta la política de reasignación.
- **Then** encuentra nuevo courier o escala el incidente con visibilidad al consumidor.

---

## Métricas

### Ejecución 1 — 2026-05-23T17:40:00-04:00

- **Prompt:** `PR-FSD-FTGO-001` / `v1`
- **Run ID:** `20260523-174000-0400`

| Nombre de la métrica | Valor | Insights |
| --- | ---: | --- |
| UCs entregados (count) | 6 | Se cubren las 3 US semilla y 3 UCs derivados trazables al brief/PRD. |
| % UCs con Given/When/Then | 100 | Los 6 UCs contienen bloque formal Given/When/Then. |
| Trazabilidad (%) | 100 | Cada UC mapea a capacidad PRD y cita origen (US o §A.4/A.5). |
| Cobertura NFRs (%) | 100 | UCs cubren latencia, disponibilidad y tolerancia a fallos relevantes del PRD. |
| Reducción de iteraciones (%) | 0 | Primera corrida del artefacto FSD v1. |
| Ediciones humanas (count) | 0 | Sin ajustes manuales posteriores en esta ejecución. |
| Inventado / Hallucination (%) | 0 | No se introducen actores ni capacidades fuera del Anexo A/PRD. |
