# Guía de Despliegue en Producción — SINDEN

Esta guía describe los pasos para desplegar el Sistema de Gestión de Órdenes de Trabajo en un servidor de producción. Para la instalación en ambiente de desarrollo, consulte el `README.md`.

---

## Requisitos del servidor

- Sistema operativo: Linux (Ubuntu 20.04+ recomendado) o Windows Server
- PHP 8.0 o superior con las siguientes extensiones habilitadas: `pdo_mysql`, `mbstring`, `openssl`, `tokenizer`, `xml`, `ctype`, `json`, `bcmath`, `fileinfo`, `gd`, `zip`
- Composer 2.x
- MySQL 5.7+ o MariaDB 10.4+
- Servidor web: Apache 2.4+ o Nginx 1.18+
- Node.js 16+ y npm (solo para compilar los assets en el despliegue inicial)
- Mínimo 1 GB de RAM y 2 GB de espacio en disco

---

## 1. Preparación del servidor

Instalar las dependencias del sistema (ejemplo para Ubuntu):

```bash
sudo apt update
sudo apt install -y php8.1 php8.1-cli php8.1-mysql php8.1-xml php8.1-mbstring \
    php8.1-curl php8.1-zip php8.1-gd php8.1-bcmath \
    composer mariadb-server nginx nodejs npm
```

Crear el usuario y la base de datos:

```sql
CREATE DATABASE sinden CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'sinden_app'@'localhost' IDENTIFIED BY 'una_clave_segura';
GRANT ALL PRIVILEGES ON sinden.* TO 'sinden_app'@'localhost';
FLUSH PRIVILEGES;
```

---

## 2. Clonar el proyecto

Ubicarse en la carpeta de aplicaciones web (por ejemplo `/var/www/`) y clonar el repositorio:

```bash
cd /var/www
sudo git clone https://github.com/Tatylorenb/Sinden-mts.git sinden
sudo chown -R www-data:www-data sinden
cd sinden
```

---

## 3. Instalar dependencias

Instalar únicamente las dependencias necesarias para producción:

```bash
composer install --no-dev --optimize-autoloader
npm install
npm run build
```

Una vez compilados los assets, se puede desinstalar Node.js del servidor si no se usará para otra cosa.

---

## 4. Configurar el entorno

Copiar y ajustar el archivo `.env`:

```bash
cp .env.example .env
```

Las variables más importantes a ajustar para producción:

```
APP_NAME=Sinden
APP_ENV=production
APP_DEBUG=false
APP_URL=https://sinden.sudominio.com

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=sinden
DB_USERNAME=sinden_app
DB_PASSWORD=una_clave_segura

LOG_CHANNEL=daily
LOG_LEVEL=warning
```

Generar la clave de la aplicación:

```bash
php artisan key:generate
```

---

## 5. Migraciones y datos iniciales

Ejecutar las migraciones y cargar los datos base (roles, permisos, tipos de pago, tabla de precios, cliente predeterminado):

```bash
php artisan migrate --force
php artisan db:seed --force
```

El flag `--force` es obligatorio en producción porque Laravel bloquea estas operaciones por seguridad.

---

## 6. Optimización para producción

Precargar la caché de configuración, rutas y vistas para mejorar el rendimiento:

```bash
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

Si en el futuro se modifica un archivo de configuración, de rutas o una vista, estas cachés deben regenerarse ejecutando los mismos comandos.

---

## 7. Permisos de carpetas

Las carpetas `storage/` y `bootstrap/cache/` deben ser escribibles por el servidor web:

```bash
sudo chown -R www-data:www-data storage bootstrap/cache
sudo chmod -R 775 storage bootstrap/cache
```

La carpeta `public/uploads/` también requiere permisos de escritura, ya que ahí se guardan los documentos, fotos y firmas de las órdenes:

```bash
sudo mkdir -p public/uploads
sudo chown -R www-data:www-data public/uploads
sudo chmod -R 775 public/uploads
```

---

## 8. Configuración del servidor web

### Opción A — Nginx

Crear el archivo `/etc/nginx/sites-available/sinden`:

```nginx
server {
    listen 80;
    server_name sinden.sudominio.com;
    root /var/www/sinden/public;

    index index.php;
    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    client_max_body_size 50M;
}
```

Habilitar el sitio y reiniciar Nginx:

```bash
sudo ln -s /etc/nginx/sites-available/sinden /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### Opción B — Apache

El proyecto incluye un archivo `.htaccess` en la carpeta `public/`. Es necesario asegurarse de que el módulo `rewrite` esté habilitado y que el `DocumentRoot` apunte a `public/`:

```apache
<VirtualHost *:80>
    ServerName sinden.sudominio.com
    DocumentRoot /var/www/sinden/public

    <Directory /var/www/sinden/public>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

---

## 9. HTTPS

Se recomienda habilitar HTTPS mediante Certbot (Let's Encrypt):

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d sinden.sudominio.com
```

---

## 10. Programador de tareas (opcional)

Si en el futuro se agregan comandos programados, registrar en el crontab del usuario `www-data`:

```bash
* * * * * cd /var/www/sinden && php artisan schedule:run >> /dev/null 2>&1
```

---

## 11. Verificación

Abrir en el navegador la URL configurada y verificar que:

1. La pantalla de bienvenida de Sinden carga correctamente.
2. Es posible iniciar sesión con un usuario válido.
3. Cada rol accede a su panel correspondiente.
4. No se muestran errores en el archivo `storage/logs/laravel.log`.

---

## Actualizaciones posteriores

Para actualizar el sistema a una nueva versión:

```bash
cd /var/www/sinden
php artisan down
git pull origin main
composer install --no-dev --optimize-autoloader
npm install && npm run build
php artisan migrate --force
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan up
```

---

## Respaldos recomendados

- **Base de datos:** respaldo diario con `mysqldump`
- **Archivos de usuarios:** respaldo semanal de la carpeta `public/uploads/`
- **Código:** se mantiene en el repositorio Git

Ejemplo de respaldo diario de la base de datos:

```bash
mysqldump -u sinden_app -p sinden > /var/backups/sinden_$(date +%Y%m%d).sql
```

---

## Contacto

Para dudas sobre el despliegue, contactar a: `tbonelo@ucompensar.edu.co`
