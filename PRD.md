# PRD: 🏨 Travel: Motor de Reservas de Hotel (MVP NestJS + React)

## 1. Visión

Este MVP implementa un motor de reservas de hotel centrado en la lógica de negocio crítica: verificar disponibilidad en tiempo real, bloquear una habitación específica por 10 minutos durante el check-out y confirmar la reserva solo tras un pago simulado exitoso.

El sistema prioriza la consistencia del inventario con un manejo de concurrencia simple, evitando dobles reservas mediante bloqueo pesimista (transacción + `SELECT ... FOR UPDATE`) y una expiración automática de bloqueos (un estimado de 10 minutos). El frontend (React) expone un flujo de búsqueda → selección → check-out con contador regresivo visible y notificaciones de estado.

No se incluye autenticación ni administración avanzada; los datos iniciales (hotel/habitaciones/tarifas básicas) se cargan vía seeder.


## 2. Alcance (IN/OUT)

### 2.1 Business goals

- Reducir el riesgo de sobreventa (double booking) a ~0 en el flujo MVP.
- Incrementar conversión al proteger inventario durante el pago (hold de 10 minutos).
- Liberar automáticamente inventario bloqueado para evitar pérdida de ventas.

### 2.2 User goals

- Permitir al viajero seleccionar una habitación y completar el pago con la tranquilidad de que la habitación permanecerá bloqueada durante 10 minutos.
- Dar visibilidad clara del tiempo restante de bloqueo en el check-out.
- Permitir al administrador del hotel recuperar inventario automáticamente cuando el pago no se completa.

### 2.3 Non-goals

- Cancelaciones, cambios de fecha, reembolsos o políticas de penalidad.
- Registro/login, perfiles, programas de lealtad.
- Multidivisa o integración con pasarelas reales.
- Panel de administración, reportes históricos avanzados.

## 3. User personas

### 3.1 Key user types

- Viajero (huésped)
- Administrador del hotel

### 3.2 Basic persona details

- **Viajero (huésped)**: busca una habitación disponible, la bloquea durante el check-out y completa el pago dentro de un límite de tiempo, con confirmación inmediata.
- **Administrador del hotel**: necesita que el inventario no quede “secuestrado” por bloqueos abandonados; requiere liberación automática y consistencia del estado.

### 3.3 Role-based access

- **Viajero (sin login)**: acceso público a búsqueda, selección, check-out, pago simulado y confirmación.
- **Administrador del hotel (sin login en MVP)**: necesidades cubiertas indirectamente por la lógica automática de expiración; no hay UI ni endpoints protegidos de administración.

## 4. Functional requirements

- **Buscador de disponibilidad atómico** (Priority: P0)
  - Consultar disponibilidad por rango de fechas para habitaciones específicas.
  - La disponibilidad debe descontar:
    - Reservas confirmadas.
    - Bloqueos temporales activos (holds) no expirados.
  - Si una habitación está bloqueada/confirmada para cualquier fecha del rango solicitado, debe aparecer como no disponible para ese rango.

- **Bloqueo pesimista de check-out (10 minutos)** (Priority: P0)
  - Al seleccionar una habitación y rango de fechas, el sistema debe intentar crear un hold.
  - El hold debe crear un “bloqueo temporal” con `expires_at = now() + 10 minutos`.
  - El proceso de creación del hold debe ser atómico y seguro ante concurrencia.
  - En la capa de persistencia, se debe usar transacción y `SELECT ... FOR UPDATE` (o equivalente) para evitar que dos holds/reservas se creen simultáneamente para la misma habitación/rango.
