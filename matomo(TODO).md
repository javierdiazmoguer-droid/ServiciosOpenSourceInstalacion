# Instalación completa de Matomo en Ubuntu 24.04 (guía paso a paso)

Resumen: esta guía instala Matomo usando Nginx + PHP-FPM + MariaDB en Ubuntu 24.04. Ajustar valores (dominio, usuario, versiones PHP) según el entorno.

## 1) Prerrequisitos
- Usuario con privilegios sudo.
- Dominio apuntando al servidor (opcional para HTTPS).
- Puerto 80/443 accesibles (UFW/Cloud rules).
```bash
sudo apt install -y chrony
sudo systemctl enable chrony --now
```

## 2) Actualizar sistema
```bash
sudo apt update && sudo apt upgrade -y
```

## 3) Instalar webserver, PHP y extensiones requeridas
Instalar la versión de PHP disponible en repositorios (o específ. cambiando `php` por `php8.x`).

```bash
sudo apt install -y nginx mariadb-server php8.2 php8.2-fpm php8.2-mysql php8.2-xml php8.2-cli php8.2-gd php8.2-curl php8.2-mbstring php8.2-zip php8.2-intl php8.2-bcmath php8.2-opcache unzip wget

```

Verificar socket de PHP-FPM (necesario para la configuración de Nginx):
```bash
ls /run/php       # verá algo como php8.2-fpm.sock
```
Apuntar al socket correcto en la configuración de Nginx (ver sección de Nginx).

## 4) Configurar MariaDB (crear base de datos y usuario)
Acceder a MariaDB y crear DB y usuario seguros:
```bash
#Ejecutar el script de seguridad
Ejecutar el script de seguridad:

sudo mysql -u root

# dentro del cliente MariaDB:
CREATE DATABASE matomo CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
CREATE USER 'matomo'@'localhost' IDENTIFIED BY 'Matomo223344_*';
GRANT ALL PRIVILEGES ON matomo.* TO 'matomo'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
Reemplazar `ContraseñaSegura` por una contraseña fuerte.

## 5) Descargar e instalar Matomo
```bash
cd /var/www
sudo wget https://downloads.matomo.org/matomo-latest.zip -O matomo.zip
sudo unzip matomo.zip -d matomo
sudo chown -R www-data:www-data matomo
sudo chmod -R 755 matomo
rm matomo.zip
```

## 6) Configurar Nginx (ejemplo de servidor)
Crear archivo de sitio, reemplazar `example.com` y `php_socket` por los valores reales (e.g., `/run/php/php8.2-fpm.sock`).

```bash
sudo tee /etc/nginx/sites-available/matomo.conf > /dev/null <<'EOF'
server {
    listen 80;
    server_name example.com;

    root /var/www/matomo;
    index index.php index.html;

    access_log /var/log/nginx/matomo.access.log;
    error_log /var/log/nginx/matomo.error.log;

    client_max_body_size 20M;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        # Reemplazar php_socket por el socket detectado (/run/php/php8.x-fpm.sock)
        fastcgi_pass unix:/run/php/php_socket;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires max;
        log_not_found off;
    }
}
EOF

sudo ln -s /etc/nginx/sites-available/matomo.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

Asegurarse de sustituir `fastcgi_pass unix:/run/php/php_socket;` por el socket correcto (ej. `unix:/run/php/php8.2-fpm.sock`) o por `127.0.0.1:9000` si se usa TCP.

## 7) Ajustes PHP (opcional pero recomendado)
Editar valores de php.ini / fpm según necesidades, por ejemplo aumentar memory_limit y max_execution_time:
```bash
# encontrar php.ini
php -i | grep "Loaded Configuration File"
sudo nano /etc/php/8.x/fpm/php.ini   # ajustar según la versión instalada
# Cambios recomendados:
# memory_limit = 512M
# max_execution_time = 300
# upload_max_filesize = 20M
# post_max_size = 20M
```
Luego reiniciar PHP-FPM:
```bash
sudo systemctl restart php8.2-fpm   # ajustar versión
```

## 8) Ejecutar el instalador web de Matomo
Abrir en navegador: http://example.com (reemplazar por su dominio o IP).
- Seguir el asistente web: verificará requisitos, proporcionará datos de DB (host: localhost, bd: matomo, usuario, contraseña) y creará el usuario administrador.
- Finalizar instalación y anotar URL de panel.

## 9) Tarea programada (archivado crons)
Matomo requiere ejecución periódica de archiving para reportes (si no usa Matomo Cloud). Crear cron para usuario www-data:

```bash
sudo crontab -u www-data -e
```
Añadir línea (ajustar URL):
```
*/5 * * * * /usr/bin/php /var/www/matomo/console core:archive --url=https://example.com >/dev/null 2>&1
```
Este ejemplo corre cada 5 minutos.

## 10) Habilitar HTTPS con Certbot (opcional recomendado)
```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d example.com
```

## 11) Firewall
Si usa UFW:
```bash
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

## 12) Optimización de base de datos
Editar configuración de MariaDB:

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
Reiniciar:

```bash
sudo systemctl restart mariadb
```

## 13) Verificación y mantenimiento
- Acceder al panel de Matomo y comprobar recepción de datos.
- Logs: /var/log/nginx/, /var/www/matomo/tmp/logs/
- Actualizar Matomo periódicamente desde la interfaz o manualmente.
- Hacer copias de seguridad de /var/www/matomo y de la base de datos.

Notas rápidas:
- Reemplazar `example.com`, `ContraseñaSegura`, `php_socket` por valores reales.
- Si usa Apache: adaptar configuración a Apache + PHP-FPM o mod_php.
- Si hay errores 502, verificar que PHP-FPM esté corriendo y que el socket/mapeo en Nginx coincida.

Fin.