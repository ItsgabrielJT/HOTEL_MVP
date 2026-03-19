# 🏨 Backlog Final Consolidado: Travel Hotel

### HU0: Configuración del Ecosistema de Datos

**Como** equipo de ingeniería, **quiero** configurar un entorno de persistencia con transacciones ACID y CI/CD, **para** garantizar que el motor de reservas opere con consistencia y seguridad.


### HU1: Seeder de Inventario Inicial

**Como** equipo de desarrollo, **quiero** contar con una carga automática de hoteles y habitaciones, **para** realizar pruebas funcionales sin depender de ingresos manuales.

### HU2: Consulta de Disponibilidad Consistente

**Como** viajero, **quiero** ver solo las habitaciones que no tienen reservas ni bloqueos activos, **para** tomar una decisión basada en la disponibilidad real del hotel.


### HU3: Bloqueo Atómico de Checkout (Hold)

**Como** viajero, **quiero** que la habitación se aparte exclusivamente para mí por 10 minutos al seleccionarla, **para** completar mis datos de pago sin riesgo de que alguien más la reserve.

   
### HU4: Persistencia del Timer de Reserva

**Como** viajero, **quiero** que el tiempo restante de mi bloqueo se mantenga si refresco la página, **para** no perder mi turno por un error del navegador.

### HU5: Procesamiento de Pago Idempotente

**Como** viajero, **quiero** que mi pago se procese una sola vez ante reintentos de red, **para** evitar cargos duplicados en mi cuenta.

### HU6: Confirmación Definitiva de Reserva

**Como** viajero, **quiero** que mi reserva pase de "Bloqueada" a "Confirmada" tras el pago, **para** recibir mi garantía de estancia.


### HU7: Liberación Proactiva por Fallo de Pago

**Como** sistema, **quiero** liberar la habitación de inmediato si el pago es rechazado, **para** que el hotel no pierda oportunidades de venta con otros clientes.

### HU8: Expiración Automática de Bloqueos (Worker)

**Como** administrador, **quiero** que el sistema libere automáticamente los bloqueos que superen los 10 minutos, **para** evitar que el inventario quede retenido por carritos abandonados.

### HU9: Resolución de Carrera Pago-Expiración

**Como** sistema, **quiero** priorizar un pago exitoso que llega en el último segundo frente a la limpieza del worker, **para** no cancelar una venta legítima por milisegundos de desfase.