
# Instalación de WordPress en Ubuntu Server 24.04

## Requisitos previos

- Ubuntu Server 24.04 LTS
- Acceso root o usuario con permisos sudo
- Conexión a internet

## 1. Actualizar el sistema

```bash
sudo apt update
sudo apt upgrade -y
```

## 2. Instalar LAMP Stack

### Apache
```bash
sudo apt install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2
```

### MySQL
```bash
sudo apt install mysql-server -y
sudo mysql_secure_installation
```

### PHP
```bash
sudo apt install php php-mysql libapache2-mod-php php-cli php-curl php-gd php-mbstring php-xml php-xmlrpc -y
```

## 3. Crear base de datos para WordPress

```bash
sudo mysql -u root -p
```

```sql
CREATE DATABASE wordpress;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'contraseña_segura';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## 4. Descargar WordPress

```bash
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress /var/www/html/
sudo chown -R www-data:www-data /var/www/html/wordpress
```

## 5. Configurar WordPress

```bash
cd /var/www/html/wordpress
sudo cp wp-config-sample.php wp-config.php
sudo nano wp-config.php
```

Editar con los datos de la base de datos creada.

## 6. Completar instalación

Acceder a: `http://tu-ip/wordpress`
