# Prompt derivado de Especificación – Diagramas C4 (Nivel 1 y Nivel 2)

> **Propósito**: Normalizar la generación de los diagramas C4 (Contexto y Contenedor) para el caso FTGO usando la sintaxis Mermaid C4, garantizando que el diseño sea coherente con las decisiones arquitectónicas del repositorio (PRD, ADRs).

---

## 0. Metadatos del prompt

| Campo | Valor |
|-------|-------|
| ID del prompt | `PR-C4-FTGO-001` |
| Título | Generación de diagramas C4 (Nivel 1 y Nivel 2) en Mermaid |
| Artefacto origen | PRD / ADRs / Brief del Anexo A |
| ID origen | `PRD-REQ-05 / ADR-0001` |
| Tipo de prompt | Generación / Transformación |
| Modelo recomendado | Sonnet / Opus |
| Temperatura | 0.2 |
| Versión | `v0.1-seed` |
| Fecha | 22/05/2026 |
| Autor(es) | Módulo 4 - UMSS |
| Estado | Borrador |

## 1. Anatomía del prompt (contenido principal)

### 1.1 Role

Eres un arquitecto de software experto en el modelo C4 de Simon Brown y en la sintaxis Mermaid para C4Context y C4Container. Conoces el caso FTGO del libro de Richardson y has documentado al menos 10 sistemas usando C4.

### 1.2 Task

Produce 2 diagramas Mermaid del caso FTGO:
1. `c4_context.mmd` — diagrama de contexto (nivel 1): FTGO como un solo sistema + personas + sistemas externos.
2. `c4_container.mmd` — diagrama de contenedores (nivel 2): los principales contenedores (microservicios, BDs, broker) de FTGO con sus tecnologías y protocolos.

### 1.3 Context

- Documentos fuente:
  - `docs/PRD.md` (stakeholders + capacidades + NFRs).
  - `docs/adr/0001-*.md` y `docs/adr/0002-*.md` (decisiones arquitectónicas que condicionan los containers).
  - El brief del Anexo A (sistemas externos, stakeholders).
- TODO 1 (Context — sistema y stakeholders del brief): enumera explícitamente el sistema principal y todas las personas/sistemas externos que deben aparecer en el diagrama de contexto.
  <!-- TODO: completar la lista, ej.
  - Person: Consumidor, Restaurante, Courier, Empleado FTGO (back office).
  - System: FTGO Platform (el sistema bajo diseño).
  - System_Ext: Stripe (pasarela de pago), Google Maps (geocoding + rutas), SendGrid/Twilio (notificaciones email/SMS), Sistema Legacy Monolito (durante la migración Strangler Fig). Sin esta lista el diagrama omite externos clave del brief. -->

### 1.4 Reasoning (chain‑of‑thought estructurado)

Sigue estos pasos en orden:
1. Nivel 1 (Context): dibuja FTGO como un único System rodeado de Person y System_Ext. Cada relación con un protocolo y un propósito (ej. "usa la app móvil para ordenar comida").
2. Nivel 2 (Container): dentro del System_Boundary de FTGO, dibuja los contenedores que se derivan del PRD y los ADRs (ej. Consumer Service, Order Service, Kitchen Service, Delivery Service, Billing Service, Notification Service, Mobile App, Web Admin, Order DB, Kafka Broker, etc.).
3. TODO 2 (Reasoning — criterio nivel 1 vs nivel 2): define aquí la regla para decidir qué pertenece al nivel 1 (Context) y qué al nivel 2 (Container).
4. En el Nivel 2, cada relación entre contenedores debe declarar tecnología + protocolo (ej. `<<JSON/HTTPS>>` o `<<async Kafka>>`).
5. Verifica que el Nivel 2 sea coherente con los ADRs (si elegiste async, el broker aparece; si elegiste DB-per-service, hay N BDs separadas).

### 1.5 Stop condition

Detente cuando:
- Existen los 2 archivos Mermaid completos.
- Nivel 1: >= 1 Person, >= 2 System_Ext, 1 System (FTGO).
- Nivel 2: >= 5 contenedores, todas las relaciones con tecnología/protocolo.
- TODO 3 (Stop condition — criterio de sintaxis válida): agrega aquí un criterio para garantizar sintaxis Mermaid válida.
No continues produciendo contenido más allá de estas condiciones.

### 1.6 Output

Formato: dos bloques de código Mermaid (uno por diagrama), cada uno destinado a un archivo `.mmd` separado.

### C4Context

```mermaid
C4Context
title FTGO – Diagrama de contexto (nivel 1)
Person(consumer, "Consumidor", "Ordena comida")
%% ... resto de actores y sistema ...
```

### C4Container

```mermaid
C4Container
title FTGO – Diagrama de contenedores (nivel 2)
System_Boundary(ftgo, "FTGO Platform") {
    Container(order_service, "Order Service", "Java/Spring Boot", "Order Taking + Order Fulfillment")
    ContainerDb(order_db, "Order DB", "PostgreSQL", "Datos de pedidos")
    %% ... resto de contenedores ...
}
%% ... relaciones con tecnología y protocolo ...
```

TODO 4 (Output — sintaxis Mermaid C4 detallada): especifica aquí los elementos sintácticos exactos de Mermaid C4 que se deben usar y un fragmento de referencia más extenso.
<!-- TODO: agregar referencia sintáctica. Ejemplo:
```mermaid
C4Container
title FTGO – Container Diagram
Person(consumer, "Consumidor", "Usuario móvil")
Person(restaurant, "Restaurante", "Operador de cocina")
System_Ext(stripe, "Stripe", "Pasarela de pago")
System_Boundary(ftgo, "FTGO Platform") {
    Container(mobile_app, "Mobile App", "React Native", "App del consumidor")
    Container(order_svc, "Order Service", "Java 17/Spring Boot", "Toma de pedidos")
    ContainerDb(order_db, "Order DB", "PostgreSQL 15", "Pedidos y items")
    ContainerQueue(kafka, "Event Broker", "Apache Kafka", "Bus de eventos")
}
Rel(consumer, mobile_app, "Usa", "iOS/Android")
Rel(mobile_app, order_svc, "Crea pedidos", "JSON/HTTPS, REST")
Rel(order_svc, order_db, "Lee/Escribe", "JDBC")
Rel(order_svc, kafka, "Publica OrderCreated", "Kafka protocol")
Rel(order_svc, stripe, "Cobra", "JSON/HTTPS")
```
Sin un fragmento de referencia los maestrantes producen sintaxis inválida que no renderiza. -->

## 2. Invariantes del prompt

- Ambos archivos deben ser sintaxis Mermaid C4 válida (C4Context, C4Container).
- Nivel 1 debe tener >= 1 Person y >= 2 System_Ext.
- Nivel 2 debe tener >= 5 contenedores.
- Cada relación de Nivel 2 debe declarar tecnología y protocolo.

## 3. *Failure modes* declarados

| Código | Descripción | Acción del consumidor |
|--------|-------------|------------------------|
| `E_MISSING_INPUTS` | faltan PRD/ADRs | abortar con error |
| `E_INVALID_MERMAID` | la sintaxis no renderiza | rechazar y reintentar |
| `E_LEVEL_MIXED` | el Nivel 1 contiene detalles internos de FTGO | rechazar y reintentar |
| `E_NO_TECH_PROTOCOL` | hay relaciones sin tecnología/protocolo | rechazar y reintentar |

## 4. Huecos TODO del prompt C4

| # | Ubicación | Qué falta |
|---|-----------|-----------|
| 1 | Context | Lista concreta de personas y sistemas externos del brief |
| 2 | Reasoning | Regla para decidir Nivel 1 vs Nivel 2 |
| 3 | Stop condition | Criterio de sintaxis Mermaid válida |
| 4 | Output | Fragmento sintáctico de referencia más completo |

Para considerarlo "mejorado": rellena >= 2 huecos + agrega 1 sección nueva + Changelog + comando en README + métrica con 3 corridas. Guarda en `prompts_mejorados/c4_mejorado.md`.