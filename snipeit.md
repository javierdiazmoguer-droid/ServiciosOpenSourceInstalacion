# Documentación: Instalación de Snipe-IT en Ubuntu Server 24.04

## Resumen

- Esta guía explica los pasos básicos para desplegar Snipe-IT (Laravel) en Ubuntu Server 24.04.
- Incluye: requisitos, instalación de dependencias (Apache/Nginx, PHP, MariaDB, Composer, Redis opcional), configuración de la aplicación, permisos, tareas programadas y supervisor para colas.

## Requisitos previos

- Sistema: Ubuntu Server 24.04 actualizado.
- Usuario con privilegios sudo.
- Puerto 80/443 abiertos (o configurar firewall/ufw).
- Se recomienda PHP 8.2 (compatible con Snipe-IT actual).

## Paso 1 — Actualizar sistema

```bash
sudo apt update && sudo apt upgrade -y
```

## Paso 2 — Instalar servidor web, base de datos y utilidades

**Apache:**
```bash
sudo apt install -y apache2
```

**O Nginx:**
```bash
sudo apt install -y nginx
```

**MariaDB:**
```bash
sudo apt install -y mariadb-server
sudo systemctl enable --now mariadb
sudo mysql_secure_installation
```

**Firewall (ufw):**
```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Apache Full'  # o 'Nginx Full'
sudo ufw enable
```

## Paso 3 — Instalar PHP y extensiones necesarias

**Apache**
```bash
sudo apt install -y php8.2 libapache2-mod-php8.2 php8.2-cli php8.2-mysql php8.2-xml \
    php8.2-mbstring php8.2-zip php8.2-curl php8.2-gd php8.2-intl \
    php8.2-bcmath php8.2-opcache

```


**Nginx**
```bash
sudo apt install -y php8.2 php8.2-fpm php8.2-cli php8.2-mysql php8.2-xml \
    php8.2-mbstring php8.2-zip php8.2-curl php8.2-gd php8.2-intl \
    php8.2-bcmath php8.2-opcache
```

Ajustar `php.ini` según necesidades (memory_limit, opcache, upload_max_filesize, etc.)

## Paso 4 — Instalar Composer y Git

```bash
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
sudo apt install -y git unzip
```

## Paso 5 — Crear base de datos y usuario

```bash
sudo mysql -u root -p
```

Ejecutar en MySQL:
```sql
CREATE DATABASE snipeit CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'snipeuser'@'localhost' IDENTIFIED BY 'TU_CONTRASENA_SEGURA';
GRANT ALL PRIVILEGES ON snipeit.* TO 'snipeuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## Paso 6 — Descargar Snipe-IT y preparar la aplicación

```bash
sudo git clone https://github.com/snipe/snipe-it.git /var/www/snipe-it
cd /var/www/snipe-it
sudo -u www-data composer install --no-dev --prefer-dist -o
sudo -u www-data cp .env.example .env
sudo chown -R www-data:www-data /var/www/snipe-it
sudo chmod -R 755 /var/www/snipe-it
```

Editar `.env`:
```
APP_ENV=production
APP_DEBUG=false
APP_URL=http://tu-dominio-o-ip
DB_DATABASE=snipeit
DB_USERNAME=snipeuser
DB_PASSWORD=TU_CONTRASENA_SEGURA
MAIL_DRIVER=smtp
REDIS_HOST=127.0.0.1
```

Generar clave:
```bash
sudo -u www-data php artisan key:generate
```

## Paso 7 — Migraciones y datos iniciales

```bash
sudo -u www-data php artisan migrate --seed --force
```

## Paso 8 — Permisos

```bash
sudo chown -R www-data:www-data /var/www/snipe-it
sudo find /var/www/snipe-it -type f -exec chmod 644 {} \;
sudo find /var/www/snipe-it -type d -exec chmod 755 {} \;
sudo chmod -R ug+rwx storage bootstrap/cache
```

## Paso 9 — Configurar servidor web

### Apache

```apache
<VirtualHost *:80>
    ServerName dominio.com/ip
    DocumentRoot /var/www/snipe-it/public
    <Directory /var/www/snipe-it/public>
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/snipeit_error.log
    CustomLog ${APACHE_LOG_DIR}/snipeit_access.log combined
</VirtualHost>
```

```bash
sudo a2ensite snipeit.conf
sudo a2enmod rewrite
sudo systemctl reload apache2
```

### Nginx

```nginx
server {
    listen 80;
    server_name tu-dominio;
    root /var/www/snipe-it/public;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/snipeit /etc/nginx/sites-enabled/
sudo systemctl reload nginx
```

## Paso 10 — SSL (recomendado)

```bash
sudo apt install -y certbot python3-certbot-apache
sudo certbot --apache -d tu-dominio
```

## Paso 11 — Tareas programadas y colas

**Cron scheduler:**
```bash
* * * * * php /var/www/snipe-it/artisan schedule:run >> /dev/null 2>&1
```

**Supervisor para queue worker:**
```bash
sudo apt install -y supervisor
```

Crear `/etc/supervisor/conf.d/snipeit-worker.conf`:
```ini
[program:snipeit-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/snipe-it/artisan queue:work --sleep=3 --tries=3 --timeout=90
autostart=true
autorestart=true
user=www-data
numprocs=1
redirect_stderr=true
stdout_logfile=/var/log/snipeit-worker.log
```

```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start snipeit-worker:*
```

## Paso 12 — Probar la instalación

Abrir en navegador: `http(s)://tu-dominio` o IP y seguir la interfaz web de Snipe-IT.

## Buenas prácticas y seguridad

- Usar contraseñas fuertes y limitar acceso a la base de datos.
- Instalar actualizaciones de seguridad regularmente.
- Revisar permisos y asegurar que archivos sensibles no sean públicos.
- Configurar backups regulares de la base de datos y `/storage`.

## Resolución de problemas comunes

- **Error 500:** Revisar logs en `/var/log/apache2/error.log`, `/var/log/nginx/error.log` o `/var/www/snipe-it/storage/logs`.
- **Problemas de permisos:** Asegurar que `www-data` sea propietario y que `storage/bootstrap/cache` sean escribibles.
- **Extensiones PHP faltantes:** Comprobar con `php -m` y `phpinfo()`.

## Recursos útiles

- [Documentación oficial Snipe-IT](https://snipeitapp.com/)
- Documentación de Laravel para comandos artisan y manejo de colas/cron.

## Nota final

Ajusta variables y comandos según tus políticas de seguridad y requisitos de producción (usuarios, backups, monitorización, entorno de correo, configuraciones de Redis/SMTP).