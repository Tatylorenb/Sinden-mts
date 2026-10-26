# Historial de Versiones — SINDEN

Este documento registra las versiones oficiales del Sistema Web de Gestión de Órdenes de Trabajo para SINDEN S.A.S., siguiendo el formato de [Keep a Changelog](https://keepachangelog.com/es-ES/) y [Versionado Semántico](https://semver.org/lang/es/).

---

## [1.0.0] — 2026-10-26

**Primera versión estable del sistema.** Entregada al cliente al cierre de la práctica empresarial.

### Agregado — Autenticación y seguridad

- Autenticación con Laravel Breeze (login, recuperación de contraseña, verificación de correo).
- Sistema de roles y permisos con Spatie Laravel-Permission.
- Cuatro perfiles base: Administrador, Recepción, Contabilidad y Operario.
- Panel diferenciado por rol con redirección automática al iniciar sesión.

### Agregado — Módulos operativos

- **Órdenes de trabajo:** asistente de creación paso a paso con bosquejos matriz (editor de dibujo integrado), captura de firma digital del cliente y asociación de piezas con items del catálogo.
- **Flujo del operario:** registro de avance multi-operario, transferencia de piezas y liberación a cola general compartida.
- **Entregas:** registro de entregas parciales y totales con recálculo automático del estado de la orden.
- **Garantías:** gestión de devoluciones por garantía y seguimiento de reparaciones.
- **Catálogo de ítems:** servicios, materiales y productos terminados con importación/exportación en Excel.
- **Clientes:** CRUD completo con búsqueda avanzada, importación masiva desde Excel y exportación.
- **Configuración del sistema:** parámetros globales, imagen de fondo, logotipos y colores.
- **Tabla de precios:** definición de precios por categoría controlada por el Administrador.

### Agregado — Módulo financiero

- Registro de pagos con múltiples métodos (efectivo, transferencia, consignación, otros).
- Flujo de aprobación de pagos registrados por Recepción.
- Recálculo automático de saldos pendientes por orden.
- Historial financiero por cliente y por período.
- Reporte de ventas por categoría de ítem.

### Agregado — Transversales

- Bitácora de auditoría inmutable con registro de usuario, acción y timestamp en cada operación relevante.
- Sistema de notificaciones internas.
- Once exportaciones a Excel para los distintos reportes operativos y financieros.
- Generación de PDF individual y masivo de las órdenes.
- Bloqueo colaborativo de órdenes durante la edición simultánea.

### Agregado — Documentación

- `README.md` con descripción del proyecto, requisitos e instrucciones de instalación.
- `DEPLOY.md` con guía detallada de despliegue en producción (Nginx y Apache).
- `docs/arquitectura.md` con la documentación técnica del sistema.
- Manual de usuario por perfil (Administrador, Recepción, Contabilidad, Operario).

### Técnico

- Base del sistema: Laravel 9 (PHP 8.0+), MySQL/MariaDB 10.4+.
- Frontend: Blade, Tailwind CSS, Alpine.js, Livewire.
- Librerías de interfaz: Bootstrap 4, jQuery 3.2.1, Select2, Animate.css.
- Reportes: Maatwebsite Excel, DomPDF, Yajra DataTables.

---

## Notas de cierre

Esta versión 1.0.0 corresponde al entregable final de la práctica empresarial realizada en MyTechSolutions S.A.S. para SINDEN S.A.S. durante el período del 18 de agosto al 26 de octubre de 2026.

El sistema queda desplegado, documentado y listo para su operación cotidiana en el taller. Las versiones posteriores dependerán de la retroalimentación de los usuarios finales y de las nuevas necesidades que surjan con el uso en el día a día.

**Agradecimientos** a Amparo Ríos (jefe inmediato en MyTechSolutions), al equipo de SINDEN S.A.S. por la retroalimentación durante el proceso y al tutor académico John Fredy Céspedes Beltrán por el acompañamiento.
