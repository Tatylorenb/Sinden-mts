# SINDEN — Sistema Web de Gestión de Órdenes de Trabajo

Sistema web desarrollado en Laravel 9 para la gestión integral de órdenes de trabajo de un taller: creación de órdenes, ejecución multi-operario, control de entregas, cobros, garantías y auditoría de operaciones.

Proyecto desarrollado por **MyTechSolutions S.A.S.** para **SINDEN S.A.S.** en el marco de la práctica empresarial de la Facultad de Ingeniería de la Fundación Universitaria Compensar.

---

## Características principales

- **Cuatro perfiles de acceso** con permisos diferenciados: Administrador, Recepción, Contabilidad y Operario.
- **Módulo de órdenes** con asistente guiado, bosquejos matriz (editor de dibujo), captura de firma digital y asociación de piezas.
- **Flujo operativo** con registro de avance multi-operario, transferencia de piezas y liberación a una cola general compartida.
- **Módulo financiero** con pagos pendientes, aprobación, historial, reporte de ventas por categoría y recálculo automático del saldo de cada orden.
- **Módulo de entregas** parciales y totales con recálculo automático del estado de la orden.
- **Módulo de garantías** para seguimiento de devoluciones.
- **Bitácora de auditoría inmutable** que registra quién hizo cada operación y cuándo, con filtros por usuario, módulo y fecha.
- **Reportes exportables** a Excel (11 reportes operativos y financieros) y generación de PDF para las órdenes.
- **Importación masiva** de clientes y catálogo de ítems desde Excel.

---

## Requisitos técnicos

- PHP 8.0 o superior
- Composer 2.x
- MySQL 5.7+ o MariaDB 10.4+
- Node.js 16+ y npm (para compilar los assets)
- Extensiones de PHP: `pdo_mysql`, `mbstring`, `openssl`, `tokenizer`, `xml`, `ctype`, `json`, `bcmath`, `fileinfo`, `gd`

---

## Instalación

### 1. Clonar el repositorio

```bash
git clone https://github.com/Tatylorenb/Sinden-mts.git
cd Sinden-mts
```

### 2. Instalar dependencias de PHP

```bash
composer install
```

### 3. Instalar dependencias de Node.js y compilar los assets

```bash
npm install
npm run build
```

### 4. Configurar el archivo de entorno

Copie el archivo `.env.example` como `.env` y ajuste las variables de conexión a la base de datos:

```bash
cp .env.example .env
```

Edite el archivo `.env` con los datos de su base de datos:

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=sinden
DB_USERNAME=root
DB_PASSWORD=
```

### 5. Generar la clave de la aplicación

```bash
php artisan key:generate
```

### 6. Crear la base de datos

Desde su cliente de MySQL/MariaDB, cree la base de datos:

```sql
CREATE DATABASE sinden CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### 7. Ejecutar las migraciones y los seeders

```bash
php artisan migrate --seed
```

Esto creará todas las tablas del sistema y cargará los datos iniciales: roles, permisos, tipos de pago, tabla de precios base y el cliente predeterminado.

### 8. (Opcional) Crear un usuario administrador de prueba

```bash
php artisan db:seed --class=AdminPruebaSeeder
```

Esto crea un usuario administrador con las siguientes credenciales:

- **Correo:** `admin.prueba@sinden.com.co`
- **Contraseña:** `Sinden2026*`

### 9. Levantar el servidor de desarrollo

```bash
php artisan serve
```

La aplicación quedará disponible en [http://localhost:8000](http://localhost:8000).

---

## Usuarios por defecto

El sistema maneja cuatro roles, cada uno con un panel y permisos propios:

| Rol | Acceso |
|---|---|
| **Administrador** | Acceso completo a todos los módulos, incluyendo configuración del sistema, gestión de usuarios y auditoría global. |
| **Recepción** | Creación de órdenes, gestión de clientes y catálogo, consulta de precios, impresión de órdenes. |
| **Contabilidad** | Panel financiero, registro y aprobación de pagos, saldos pendientes, reportes de recaudo e historial. |
| **Operario** | Panel del operario, trabajo sobre piezas asignadas, registro de avance, transferencia a otro operario o cola general. |

---

## Estructura del proyecto

```
sinden-mts/
├── app/
│   ├── Http/
│   │   ├── Controllers/      Controladores por módulo
│   │   └── Requests/         Validaciones de formulario
│   ├── Models/               Modelos Eloquent (26 modelos del dominio)
│   ├── Services/             Capa de servicios (dominio y reglas de negocio)
│   ├── Exports/              Clases de exportación a Excel
│   └── Policies/             Políticas de autorización
├── database/
│   ├── migrations/           Migraciones de la base de datos
│   └── seeders/              Semillas de datos iniciales
├── resources/
│   └── views/                Vistas Blade organizadas por módulo
├── routes/
│   └── web.php               Rutas de la aplicación
└── public/
    ├── js/                   Scripts del lado del cliente (dibujo, firma)
    └── vendor/               Librerías de interfaz (Bootstrap, jQuery, Select2)
```

---

## Tecnologías utilizadas

- **Backend:** Laravel 9 (PHP 8), Eloquent ORM
- **Frontend:** Blade, Tailwind CSS, Alpine.js, Livewire, Bootstrap 4, jQuery, Select2
- **Base de datos:** MySQL / MariaDB
- **Autenticación:** Laravel Breeze
- **Permisos:** Spatie Laravel-Permission
- **Reportes:** Maatwebsite Excel, DomPDF
- **Tablas interactivas:** Yajra DataTables
- **Control de versiones:** Git / GitHub

---

## Comandos útiles

| Acción | Comando |
|---|---|
| Limpiar la caché de configuración | `php artisan config:clear` |
| Limpiar la caché de rutas | `php artisan route:clear` |
| Limpiar la caché de vistas | `php artisan view:clear` |
| Listar las rutas disponibles | `php artisan route:list` |
| Ejecutar las pruebas | `php artisan test` |
| Reiniciar la base de datos | `php artisan migrate:fresh --seed` |

---

## Soporte y contacto

Para preguntas o incidencias relacionadas con el proyecto, contactar a:

- **Practicante / desarrolladora:** Tatiana Lorena Bonelo Manrique — `tbonelo@ucompensar.edu.co`
- **Empresa receptora:** MyTechSolutions S.A.S.
- **Empresa beneficiaria:** SINDEN S.A.S.

---

## Licencia

Este proyecto es de uso interno de SINDEN S.A.S. Todos los derechos reservados.
