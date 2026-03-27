
# REALITY_CHECK.md

## Contexto

El MVP del motor de reservas fue desarrollado y validado en un esquema de **2 micro-sprints (4 días)**, con un enfoque altamente pragmático basado en:

* Riesgo (Risk-Based Testing)
* Validación de invariantes de negocio
* Reducción de alcance funcional
* Entrega acelerada con calidad controlada

El desarrollo tomó aproximadamente **27 horas reales**:

* Micro-Sprint 1: 20h
* Micro-Sprint 2: 7h

---

## ¿Qué tareas subestimamos y por qué?

### 1. Concurrencia (HU3) — 🔴 Subestimación crítica

**Problema:**

* La implementación de `SELECT ... FOR UPDATE` resolvía el caso simple, pero no contemplaba completamente:

  * Locks prolongados
  * Deadlocks potenciales
  * Simulación realista de concurrencia

**Causa:**

* Falta de pruebas iniciales con múltiples hilos reales
* Subestimación del comportamiento de la base de datos bajo carga concurrente

**Impacto:**

* Ajustes tardíos en lógica transaccional
* Incremento en tiempo de pruebas QA

---

### 2. Idempotencia en pagos (HU5) — 🔴 Subestimación media

**Problema:**

* Manejo de `Idempotency-Key` requirió más lógica de persistencia de la esperada
* Edge cases:

  * Reintentos simultáneos
  * Reintentos después de expiración del hold

**Causa:**

* Se asumió un flujo lineal, pero el comportamiento real de red es no determinista

**Impacto:**

* Complejidad adicional en backend
* Necesidad de pruebas negativas adicionales

---

### 3. Worker de expiración (HU8) — 🟡 Subestimación moderada

**Problema:**

* Manejo de:

  * Idempotencia del worker
  * Reprocesamiento tras caídas
  * Consistencia eventual

**Causa:**

* Se subestimó la complejidad de procesos asincrónicos

**Impacto:**

* Ajustes en queries de disponibilidad (fallback sin worker)
* Incremento leve en esfuerzo QA

---

### 4. Sincronización Frontend–Backend (Timer) — 🟡

**Problema:**

* Desfase entre tiempo del cliente y servidor

**Causa:**

* Suposición inicial de confianza en reloj del frontend

**Impacto:**

* Se tuvo que rediseñar para usar `expires_at` del backend

---

## ¿El MVP es realmente valioso para el negocio?

### Objetivos cumplidos

* ✅ **Overbooking = 0** (garantizado mediante bloqueo pesimista)
* ✅ Flujo completo funcional:

  * Búsqueda → Hold → Pago → Confirmación
* ✅ Protección de inventario (holds de 10 min)
* ✅ Liberación automática de inventario
* ✅ Manejo de concurrencia real

### Valor entregado

* Permite validar el modelo de negocio en producción
* Reduce riesgo financiero por sobreventa
* Provee base sólida para escalar funcionalidades

 **Conclusión:**
El MVP es **funcional, coherente y listo para validación de negocio**, cumpliendo su propósito principal.

---

## ¿Cómo QA garantizó la calidad en tan poco tiempo?

Se aplicó la siguiente estrategia mencionado en [TEST_PLAN.md - 3. Estretegia de Pruebas](https://github.com/ItsgabrielJT/QA_HOTEL_MVP/blob/main/TEST_PLAN.md#3-estrategia-de-pruebas) junto a los entregables [TEST_PLAN.md - 9. Entregables](https://github.com/ItsgabrielJT/QA_HOTEL_MVP/blob/main/TEST_PLAN.md#9-entregables-de-prueba) para poder asegurar la calidad del mvp 

### Niveles de Prueba

| Nivel | Descripción | HUs Principales |
|---|---|---|
| **Unitario** | Lógica de negocio aislada (servicios, validadores) | HU11, HU6, HU7 |
| **Integración** | DB + Servicios + Endpoints de API | HU0, HU2, HU5, HU8 |
| **E2E** | Flujos completos: viajero → confirmación | HU3, HU6 |
| **Concurrencia** | Race conditions y bloqueos atómicos | HU3 |
| **Seguridad** | Inyección de entradas | HU2, HU3 |
| **Regresión** | Suite crítica tras cada release | HU2, HU3, HU5, HU6, HU7 |

### Tipos de Prueba

| Tipo | Herramienta Propuesta | Condición de Ejecución |
|---|---|---|
| Funcional / API | Karate | Siempre — obligatorio |
| Concurrencia | k6 + threading | HU3 — obligatorio |
| Seguridad (OWASP) | OWASP ZAP + pruebas manuales | HU3 — obligatorio |
| Performance / Load | k6 | Solo si hay SLAs definidos en spec |

### Entregables

| Entregable                    | Descripción                         |
| ----------------------------- | ----------------------------------- |
| **Scripts Automatizados**     | Serenity + Cucumber, Karate, k6, Pytest                  |
| **Reportes de Ejecución**     | Resultados por suite (JSON / HTML)  |
| **Repositorio de pruebas**    | Cada uno independiente     |
| **Ejecución manual**    | Pantallazo o video corto         |
| **Reporte de bugs**          | Tambien se reportan incidencias |

