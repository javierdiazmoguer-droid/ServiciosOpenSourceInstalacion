# Instalación de Redmine en Ubuntu 24.04

## Requisitos previos
- Ubuntu 24.04 LTS
- Acceso root o sudo
- Conexión a Internet

## 1. Actualizar el sistema
```bash
sudo apt update
sudo apt upgrade -y
```

## 2. Instalar dependencias
```bash
sudo apt install -y ruby ruby-dev build-essential libmysqlclient-dev imagemagick libmagickwand-dev
sudo apt install -y mysql-server mysql-client
```

## 3. Configurar MySQL
```bash
sudo mysql_secure_installation
mysql -u root -p
CREATE DATABASE redmine CHARACTER SET utf8mb4;
CREATE USER 'redmine'@'localhost' IDENTIFIED BY 'tu_contraseña';
GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## 4. Instalar Redmine
```bash
cd /opt
sudo wget https://www.redmine.org/releases/redmine-5.1.1.tar.gz
sudo tar xvf redmine-5.1.1.tar.gz
sudo mv redmine-5.1.1 redmine
sudo chown -R $USER:$USER redmine
```

## 5. Configurar Redmine
```bash
cd redmine
cp config/database.yml.example config/database.yml
```

Editar config/database.yml:
```yaml
production:
    adapter: mysql2
    database: redmine
    host: localhost
    username: redmine
    password: "tu_contraseña"
```

## 6. Instalar gemas y configurar la base de datos
```bash
gem install bundler
sudo apt install -y libssl-dev zlib1g-dev libpq-dev
bundle install --without development test
bundle exec rake generate_secret_token
RAILS_ENV=production bundle exec rake db:migrate
RAILS_ENV=production bundle exec rake redmine:load_default_data
```

## 7. Configurar Passenger y Nginx
```bash
sudo apt install -y libnginx-mod-http-passenger
gem install passenger
sudo nano /etc/nginx/sites-available/redmine.conf
<VirtualHost *:80>
    ServerName redmine.local
    DocumentRoot /opt/redmine/public
    <Directory /opt/redmine/public>
        AllowOverride all
        Options -MultiViews
        Require all granted
    </Directory>
</VirtualHost>
```

## 8. Iniciar servicios
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

## 9. Acceder a Redmine
- Abrir navegador web
- Acceder a http://localhost
- Usuario por defecto: admin
- Contraseña por defecto: admin

## Notas de seguridad
- Cambiar la contraseña del administrador inmediatamente
- Configurar SSL/TLS para conexiones seguras
- Mantener el sistema actualizado regularmente