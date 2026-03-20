
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