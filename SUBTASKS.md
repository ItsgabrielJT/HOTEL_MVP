
## 🛠️ Desglose de Tasking por Historia de Usuario

### 1. Cimientos e Infraestructura
#### **HU0: Configuración del Ecosistema de Datos (5 SP)**
* **Dev (Backend):**
    * Configurar Docker Compose con PostgreSQL 15+.
    * Instalar SQLAlchemy/SQLModel y Alembic para migraciones.
    * Configurar el nivel de aislamiento de transacciones en la conexión.
    * Crear scripts de GitHub Actions para CI (Linter + Vitest).


#### **HU1: Seeder de Inventario Inicial (2 SP)**
* **Dev (Backend):**
    * Crear script de Node para poblar tablas `Hotel` y `Room` con datos realistas.
    * Implementar comando de consola (ej. `make seed`) para ejecución rápida.
    
    ### 2. Ciclo de Búsqueda y Reserva
#### **HU2: Consulta de Disponibilidad Consistente (3 SP)**
* **Dev (Backend):**
    * Crear endpoint `GET /rooms/available` que reciba `checkin` y `checkout`.
    * Implementar lógica SQL (Subquery o Join) que filtre habitaciones que tengan `Reservations` o `Holds` activos en ese rango de fechas.
* **Dev (Frontend):**
    * Crear componente de Buscador con selectores de fecha (DatePickers).
    * Mapear resultados en tarjetas de habitaciones.


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
* **Dev (Frontend):**
    * Pantalla de éxito con resumen de reserva.


#### **HU7: Generación de Código Único (2 SP)**
* **Dev (Backend):**
    * Utilizar librería de Hash o NanoID para el código de reserva.

    
### 4. Background Tasks (Workers)
#### **HU8: Liberación Proactiva por Fallo (2 SP)**
* **Dev (Backend):**
    * Si el simulador de pago retorna `Status: Declined`, ejecutar `DELETE` o `Update status: Released` al Hold.

    
    
#### **HU9: Expiración Automática - Worker (3 SP)**
* **Dev (Backend):**
    * Configurar tarea programada (ej. FastAPI BackgroundTasks o un script loop).
    * Consulta: `UPDATE holds SET status = 'EXPIRED' WHERE expires_at < NOW()`.