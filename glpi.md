/**
# Guía: Instalación completa de GLPI en Ubuntu 24.04

Esta documentación describe los pasos para instalar y configurar GLPI (Gestión Libre de Parques Informáticos) en Ubuntu 24.04 en un servidor LAMP/LEMP básico. Ajusta nombres de paquetes, versiones y dominios según tu entorno.

## Resumen de componentes
- Sistema operativo: Ubuntu 24.04
- Servidor web: Apache2 (opcional: Nginx + php-fpm)
- Base de datos: MariaDB / MySQL
- PHP: versión por defecto de la distribución (asegúrate de cumplir los requisitos de la versión de GLPI)
- GLPI: última versión estable desde releases oficiales

---

## 1) Preparación del sistema
1. Actualizar paquetes:
    ```
    sudo apt update && sudo apt upgrade -y
    ```

2. Instalar utilidades básicas:
    ```
    sudo apt install -y wget curl unzip tar gnupg lsb-release software-properties-common
    ```

---

## 2) Instalar Apache, MariaDB y PHP (LAMP)
1. Instalar servidor web y MariaDB:
    ```
    sudo apt install -y apache2 mariadb-server
    ```

2. Instalar PHP y extensiones requeridas (ajusta si necesitas php-fpm en vez de libapache2-mod-php):
    ```
    sudo apt install -y php php-cli libapache2-mod-php php-mysql php-xml php-gd php-curl php-zip php-mbstring php-intl php-bcmath php-ldap php-json php-fileinfo
    ```

3. Reiniciar Apache tras la instalación:
    ```
    sudo systemctl restart apache2
    sudo a2enmod php*
    ```

---

## 3) Asegurar MariaDB y crear base de datos GLPI
1. Ejecutar script seguro:
    ```
    sudo mysql_secure_installation
    ```

2. Crear la base de datos y el usuario para GLPI:
    ```
    sudo mysql -u root -p
    -- En el prompt de MySQL:
    CREATE DATABASE glpi CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
    CREATE USER 'glpiuser'@'localhost' IDENTIFIED BY 'TuContraseñaSegura';
    GRANT ALL PRIVILEGES ON glpi.* TO 'glpiuser'@'localhost';
    FLUSH PRIVILEGES;
    EXIT;
    ```

Nota: usa contraseñas seguras y, si procede, restringe acceso por host.

---

## 4) Descargar e instalar GLPI
1. Descargar la última versión estable (reemplaza la URL por la versión actual):
    ```
    cd /tmp
    wget https://github.com/glpi-project/glpi/releases/download/X.Y.Z/glpi-X.Y.Z.tgz
    tar -xzf glpi-X.Y.Z.tgz
    sudo mv glpi /var/www/html/glpi
    ```

2. Ajustar permisos:
    ```
    sudo chown -R www-data:www-data /var/www/html/glpi
    sudo find /var/www/html/glpi -type d -exec chmod 750 {} \;
    sudo find /var/www/html/glpi -type f -exec chmod 640 {} \;
    ```

---

## 5) Configurar Apache (virtual host)
1. Crear archivo de sitio (ejemplo /etc/apache2/sites-available/glpi.conf):
    - DocumentRoot: /var/www/html/glpi
    - Habilitar DirectoryIndex y AllowOverride All si usas .htaccess

    Ejemplo mínimo de configuración:
    ```
    <VirtualHost *:80>
         ServerName ejemplo.com
         DocumentRoot /var/www/html/glpi
         <Directory /var/www/html/glpi>
              DirectoryIndex index.php
              AllowOverride All
              Require all granted
         </Directory>
         ErrorLog ${APACHE_LOG_DIR}/glpi_error.log
         CustomLog ${APACHE_LOG_DIR}/glpi_access.log combined
    </VirtualHost>
    ```

2. Habilitar el sitio y módulos necesarios:
    ```
    sudo a2dissite 000-default.conf
    sudo a2ensite glpi.conf
    sudo a2enmod rewrite
    sudo systemctl reload apache2
    sudo systemctl restart apache2
    ```

---

## 6) Ajustes PHP recomendados
Editar php.ini (ruta p. ej. /etc/php/*/apache2/php.ini) y ajustar:
- memory_limit = 256M (o más)
- upload_max_filesize = 50M
- post_max_size = 50M
- max_execution_time = 300
- date.timezone = "Europe/Madrid" (ajusta a tu zona)

Reiniciar Apache tras cambios: