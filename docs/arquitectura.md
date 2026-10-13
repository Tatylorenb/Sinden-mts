# Arquitectura Técnica — SINDEN

Documento de referencia técnica para desarrolladores y para la auditoría del sistema. Describe cómo están organizadas las capas de la aplicación, los principales módulos del dominio y las decisiones de diseño que se tomaron durante la construcción del proyecto.

---

## 1. Vista general

SINDEN es una aplicación web monolítica construida sobre el marco Laravel 9, que sigue el patrón **MVC** con una capa adicional de **servicios de dominio** para encapsular las reglas de negocio más complejas. La aplicación organiza el acceso en cuatro perfiles de usuario (Administrador, Recepción, Contabilidad y Operario) y expone paneles diferenciados para cada uno.

```
Navegador (Blade + Tailwind + Alpine.js + Livewire)
        │
        ▼
Rutas (routes/web.php)
        │
        ▼
Controladores (app/Http/Controllers)
        │
        ▼
Servicios de dominio (app/Services)
        │
        ▼
Modelos Eloquent (app/Models)
        │
        ▼
MySQL / MariaDB
```

---

## 2. Capas de la aplicación

### 2.1 Capa de presentación

- Vistas en **Blade** organizadas por módulo (`resources/views/<modulo>/`).
- **Tailwind CSS** para los estilos y **Alpine.js** / **Livewire** para la interactividad del cliente.
- **DataTables** (vía Yajra) para las tablas interactivas con filtros, paginación y búsqueda.
- Librerías adicionales en `public/vendor/` (Bootstrap 4, jQuery, Select2, Animate.css) para componentes visuales específicos.

### 2.2 Capa de control (controladores)

Un controlador por módulo, agrupados por responsabilidad:

- **Controladores de recepción:** `OrdenController`, `ClienteController`, `CatalogoItemController`, `BosquejoMatrizController`, `ConsultaPrecioController`.
- **Controladores de operario:** `OperarioController`.
- **Controladores de contabilidad:** `ContabilidadController`.
- **Controladores de entregas y garantías:** `EntregaController`, `GarantiaController`.
- **Controladores administrativos:** `Admin\UserController`, `Admin\ConfiguracionController`, `Admin\TablaPreciosController`, `Admin\PanelController`.
- **Controladores transversales:** `ActividadController` (auditoría), `NotificacionController`, `OrdenPdfController`.

Los controladores se encargan únicamente de recibir la petición, validarla, delegar la lógica a los servicios y preparar la respuesta.

### 2.3 Capa de servicios de dominio

Los servicios encapsulan la lógica de negocio reutilizable:

| Servicio | Responsabilidad |
|---|---|
| `OrdenService` | Creación, actualización y copia de órdenes, con todas sus relaciones. |
| `OrdenEstadoService` | Recalcular los tres estados de una orden (trabajo, entrega, pago) cuando ocurre cualquier evento relevante. |
| `OperarioPiezaService` | Avance, transferencia y liberación de piezas por parte de los operarios. |
| `BloqueoService` | Bloqueo colaborativo de órdenes durante la edición simultánea. |
| `DashboardService` | Cálculo de métricas visibles en cada panel según el rol. |
| `NotificacionService` | Creación y entrega de notificaciones internas. |

### 2.4 Capa de datos (modelos)

26 modelos Eloquent agrupados por dominio:

- **Dominio operativo:** `Orden`, `OrdenPieza`, `OrdenPiezaItem`, `OrdenPiezaItemGrupo`, `OrdenPiezaObservacion`, `OrdenDocumento`, `OrdenFoto`, `OrdenComentario`, `AsignacionPieza`, `HistorialAvance`.
- **Catálogos:** `CatalogoItem`, `BosquejoMatriz`, `ClientesImport`.
- **Clientes:** `Cliente`.
- **Entregas y garantías:** `Entrega`, `EntregaPieza`, `DevolucionGarantia`.
- **Financiero:** `Pago`, `TipoPago`.
- **Configuración:** `ConfiguracionSistema`, `TablaPrecios`.
- **Transversales:** `User`, `RegistroActividad`, `Notificacion`, `Fichaje` (opcional).

---

## 3. Autenticación y autorización

- **Autenticación:** Laravel Breeze (login, registro, verificación de email y recuperación de contraseña).
- **Autorización:** Spatie Laravel-Permission, con cuatro roles base definidos en `RolesAndPermissionsSeeder`:
  - **Administrador** — todos los permisos.
  - **Recepción** — creación/edición de órdenes, gestión de clientes y catálogo.
  - **Contabilidad** — módulo financiero, aprobación de pagos.
  - **Operario** — panel de trabajo, avance de piezas, transferencias.
- Las rutas se protegen combinando los middlewares `auth`, `verified` y `role:...`.
- Para redirecciones post-login según rol, se usa `RoleRedirectController`.

---

## 4. Modelo de datos

### Entidades principales

```
Cliente ─────┐
             │
             ▼
            Orden ──┬──> OrdenPieza ──┬──> OrdenPiezaItem (servicios/materiales)
             │      │                 ├──> OrdenPiezaItemGrupo
             │      │                 ├──> OrdenPiezaObservacion
             │      │                 ├──> AsignacionPieza (quien trabaja ahora)
             │      │                 └──> HistorialAvance (quien trabajo antes)
             │      ├──> OrdenDocumento (archivos adjuntos)
             │      ├──> OrdenFoto (fotos del taller)
             │      ├──> OrdenComentario
             │      ├──> Entrega ─────> EntregaPieza
             │      ├──> Pago
             │      └──> DevolucionGarantia
             │
             ▼
       ConfiguracionSistema  (parametros globales)
       TablaPrecios           (precios por categoria)
       CatalogoItem           (items reutilizables)
       BosquejoMatriz         (plantillas de bosquejo)
```

### Decisiones de diseño destacables

- **Trazabilidad multi-operario:** la asignación actual de una pieza se guarda en `AsignacionPieza`, y el histórico de porcentajes de avance queda en `HistorialAvance`. Así una pieza siempre tiene un responsable visible y conserva el rastro completo de quién la trabajó previamente.
- **Recálculo automático de estados:** los tres estados de una orden (trabajo, entrega, pago) no se asignan manualmente; los calcula `OrdenEstadoService` cada vez que un evento relevante cambia (una entrega, un pago, un avance). Esto evita inconsistencias cuando varios usuarios actúan en paralelo.
- **Auditoría inmutable:** cada operación relevante crea un registro en `RegistroActividad`. La tabla es apéndice-only; no se modifican ni se eliminan registros, lo que garantiza una pista de auditoría confiable.
- **Soft deletes:** las entidades que pueden desactivarse (`TipoPago`, `User.activo`) usan borrado lógico para preservar la integridad referencial de las órdenes históricas.

---

## 5. Flujos críticos

### 5.1 Creación de una orden (asistente guiado)

1. Recepción abre `/recepcion/ordenes/crear`.
2. Selecciona o crea un cliente.
3. Agrega las piezas y sus items (servicios, materiales, productos terminados).
4. Dibuja los bosquejos matriz en el canvas y captura la firma digital del cliente.
5. `OrdenController@guardar` persiste la orden como borrador.
6. Al confirmar, `OrdenController@generar` cambia el estado a oficial y genera el PDF imprimible.

### 5.2 Flujo del operario sobre una pieza

1. El operario busca la orden por número o accede a sus asignaciones.
2. En la vista `operario/trabajar.blade.php` registra el porcentaje de avance.
3. `OperarioPiezaService` actualiza `AsignacionPieza` y crea una entrada en `HistorialAvance`.
4. Si transfiere la pieza, el servicio actualiza la asignación al nuevo operario y conserva el historial.
5. Si libera a cola general, la pieza queda disponible para cualquier operario.
6. `OrdenEstadoService` recalcula el estado global de la orden tras cada cambio.

### 5.3 Registro y aprobación de pagos

1. Contabilidad consulta las órdenes con saldo pendiente.
2. Registra un abono o pago total asociado a la orden.
3. `OrdenEstadoService` recalcula el estado de pago.
4. Los pagos pendientes de aprobación se listan en una bandeja; el administrador o contabilidad los aprueba.
5. El historial financiero por cliente y por período queda disponible como reporte exportable.

### 5.4 Entregas parciales

1. Recepción abre el módulo de entregas de una orden.
2. Marca qué piezas se están entregando (puede ser una entrega parcial).
3. `Entrega` y `EntregaPieza` registran la operación con fecha y usuario.
4. `OrdenEstadoService` actualiza el estado de entrega (parcial o total).
5. Si todas las piezas están entregadas, la orden queda marcada como entregada.

---

## 6. Convenciones de código

- **PSR-12** para el estilo PHP.
- Nombres de clases, métodos y variables en **inglés o español consistente** según el contexto del dominio (los nombres del dominio del taller se mantienen en español: `Orden`, `Pieza`, `Entrega`).
- Las migraciones usan el convenio de Laravel y respetan el orden cronológico.
- Las vistas Blade se organizan por módulo y usan parciales (`_partials/` o `partials/`) para los componentes reutilizables.
- Los servicios no reciben la `Request`; reciben datos ya validados por el controlador o un Form Request.

---

## 7. Pruebas

El proyecto incluye el esqueleto de pruebas de Laravel en `tests/`:

- **Feature:** pruebas de los flujos de autenticación y de perfil provistos por Breeze, y pruebas de autorización por rol en `RoleAuthorizationTest`.
- **Unit:** reservado para pruebas de servicios aislados.

Las pruebas se ejecutan con:

```bash
php artisan test
```

En la fase de cierre se agregarán pruebas automatizadas de los flujos críticos (creación de orden, avance del operario, entrega, cobro) descritas en la sección 5.

---

## 8. Observaciones

- El sistema no expone API REST pública; toda la interacción es por el navegador, autenticada por sesión.
- Las exportaciones a Excel se generan en memoria y se entregan como descarga; para reportes grandes conviene revisar los límites de memoria de PHP.
- Las imágenes de bosquejos y las firmas digitales se guardan como archivos físicos en `public/uploads/`, cuya ruta queda referenciada en las tablas correspondientes.
