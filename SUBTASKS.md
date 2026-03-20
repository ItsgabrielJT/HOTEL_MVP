
## 🛠️ Desglose de Tasking por Historia de Usuario

### 1. Cimientos e Infraestructura
#### **HU0: Configuración del Ecosistema de Datos (5 SP)**
* **Dev (Backend):**
    * Configurar Docker Compose con PostgreSQL 15+.
    * Instalar SQLAlchemy/SQLModel y Alembic para migraciones.
    * Configurar el nivel de aislamiento de transacciones en la conexión.
    * Crear scripts de GitHub Actions para CI (Linter + Vitest).
* **QA:**
    * Crear test de integración de "stress de conexión" para validar que la BD soporta bloqueos de fila concurrentes.
    * Verificar pipeline de CI ante fallos de sintaxis.

#### **HU1: Seeder de Inventario Inicial (2 SP)**
* **Dev (Backend):**
    * Crear script de Node para poblar tablas `Hotel` y `Room` con datos realistas.
    * Implementar comando de consola (ej. `make seed`) para ejecución rápida.
* **QA:**
    * Validar integridad referencial tras el seeding (que no existan habitaciones sin hotel).
    
### 2. Ciclo de Búsqueda y Reserva
#### **HU2: Consulta de Disponibilidad Consistente (3 SP)**
* **Dev (Backend):**
    * Crear endpoint `GET /rooms/available` que reciba `checkin` y `checkout`.
    * Implementar lógica SQL (Subquery o Join) que filtre habitaciones que tengan `Reservations` o `Holds` activos en ese rango de fechas.
* **Dev (Frontend):**
    * Crear componente de Buscador con selectores de fecha (DatePickers).
    * Mapear resultados en tarjetas de habitaciones.
* **QA:**
    * **Unit:** Testear función de solapamiento de fechas (Overlap logic).
    * **Integration:** Mockear un Hold activo y verificar que la habitación desaparece de la lista.

#### **HU3: Bloqueo Atómico de Checkout (8 SP) - CRÍTICA**
* **Dev (Backend):**
    * Crear endpoint `POST /rooms/{id}/hold`.
    * Implementar bloque de transacción: `BEGIN -> SELECT FOR UPDATE -> INSERT hold -> COMMIT`.
    * Manejar excepción de "Row Locked" y retornar error 409 (Conflict).
* **Dev (Frontend):**
    * Implementar botón "Reservar" con estado de carga (Loading).
    * Manejo de errores específicos (Toast de "Alguien te ganó la habitación").

#### **HU4: Persistencia del Timer de Reserva (3 SP)**
* **Dev (Backend):**
    * Endpoint `GET /holds/{id}` que calcule `remaining_seconds` en tiempo real.
* **Dev (Frontend):**
    * Implementar Hook `useTimer` que consuma el tiempo del servidor.
    * Persistir `holdId` en `localStorage` para recuperar el estado tras F5.


### 3. Pagos e Integridad
#### **HU5: Procesamiento de Pago Idempotente (5 SP)**
* **Dev (Backend):**
    * Crear tabla `Payments` con campo `idempotency_key` único.
    * Middleware para interceptar el header `X-Idempotency-Key` y retornar respuesta cacheada si ya existe.
* **Dev (Frontend):**
    * Generar UUID en el cliente antes de disparar el request de pago.


#### **HU6: Confirmación Definitiva de Reserva (3 SP)**
* **Dev (Backend):**
    * Lógica de transición: `Hold (Pending) -> Reservation (Confirmed)`.
    * Utilizar librería de Hash o NanoID para el código de reserva.
* **Dev (Frontend):**
    * Pantalla de éxito con resumen de reserva.


    
### 4. Background Tasks (Workers)
#### **HU7: Liberación Proactiva por Fallo (2 SP)**
* **Dev (Backend):**
    * Si el simulador de pago retorna `Status: Declined`, ejecutar `DELETE` o `Update status: Released` al Hold.

    
    
#### **HU8: Expiración Automática - Worker (3 SP)**
* **Dev (Backend):**
    * Configurar tarea programada (ej. FastAPI BackgroundTasks o un script loop).
    * Consulta: `UPDATE holds SET status = 'EXPIRED' WHERE expires_at < NOW()`.


#### **HU9: Resolución de Carrera Pago-Expiración (5 SP)**
* **Dev (Backend):**
    * Implementar bloqueo pesimista en el Worker para que no pueda expirar un Hold que está siendo procesado por el endpoint de Pago.


### 5. Seguridad y Validaciones
#### **HU10: Protección Rate Limiting (5 SP)**
* **Dev (Backend):**
    * Instalar e integrar `slowapi` o Redis-limiter.
    * Configurar límite: "Max 3 holds por IP cada 5 minutos".

#### **HU11: Integridad de Fechas (3 SP)**
* **Dev (Backend):**
    * Validación en Pydantic schemas: `checkout > checkin` y `checkin >= today`.
* **Dev (Frontend):**
    * Deshabilitar fechas pasadas en el calendario.
