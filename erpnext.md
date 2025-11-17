# Instalación completa de ERPNext en Ubuntu 24.04

Guía concisa paso a paso para instalar ERPNext (Frappe) en Ubuntu 24.04. Ajusta versiones (Frappe/ERPNext, Node, MariaDB) según la compatibilidad que necesites.

> Requisitos mínimos sugeridos
- CPU 2 cores+, 4+ GB RAM (8 GB recomendado), 20+ GB disco.
- Usuario con privilegios sudo.
- Conexión a Internet.

---

## 1. Preparación del sistema
Actualizar paquetes y configurar locales:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y software-properties-common curl git
sudo locale-gen en_US.UTF-8
```

Crear un usuario dedicado (recomendado):
```bash
sudo adduser frappe
usermod -aG sudo frappe
```

(Trabaja como ese usuario para la mayoría de comandos)
```bash
sudo su - frappe
```
(Entramos en el home del usuario)
```bash
cd /home/frappe
```

---

## 2. Dependencias del sistema
Instalar paquetes básicos:
```bash
# desde cuenta root o con sudo
sudo apt install -y python3 python3-dev python3-pip python3-venv build-essential \
sudo apt-get install redis-server
redis-server mariadb-server libmariadb-dev-compat libmariadb-dev \
nodejs npm yarn gettext git curl
```

Node.js: usar la versión recomendada por la versión de Frappe (ej. Node 18):
```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm install -g yarn
```

Instalar wkhtmltopdf (versión con Qt patch, necesaria para impresión/PDF):
- Descargar el binario recomendado (p. ej. 0.12.6 con patch) desde la web oficial de wkhtmltopdf o repositorios de Frappe.
- Instalarlo y dar permisos ejecutables. Ejemplo (ajusta URL a la versión compatible):
```bash
# como root
wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6-1/wkhtmltox_0.12.6-1.focal_amd64.deb
sudo apt install -y ./wkhtmltox_0.12.6-1.focal_amd64.deb
```

---

## 3. Configurar MariaDB (MySQL) para Frappe
Editar configuración (ejemplo en /etc/mysql/mariadb.conf.d/50-server.cnf o /etc/mysql/my.cnf):
Añadir/ajustar en [mysqld]:
```
max_connections = 800
innodb_file_per_table = 1
innodb_buffer_pool_size = 1G        # ajustar según RAM
innodb_log_file_size = 256M
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
sql_mode = NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
```
Reiniciar MariaDB:
```bash
sudo systemctl restart mariadb
```

Crear usuario y contraseña MariaDB para Frappe (opcional — bench creará DBs individuales):
```bash
sudo mysql -uroot -p
# dentro del cliente:
CREATE USER 'frappe'@'localhost' IDENTIFIED BY 'tu_password_segura';
GRANT ALL PRIVILEGES ON *.* TO 'frappe'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```

---

## 4. Instalar Bench CLI y preparar entorno Frappe
Instalar bench (recomendado con pip):
```bash
pip3 install --user frappe-bench
export PATH=$PATH:~/.local/bin
```

Crear directorio de bench y inicializar entorno:
```bash
bench init frappe-bench --frappe-branch version-14
cd frappe-bench
```
Nota: Reemplaza `version-14` por la rama/versión compatible que prefieras (ver compatibilidades ERPNext/Frappe).

---

## 5. Instalar ERPNext
Obtener la app ERPNext y crear un sitio:
```bash
bench get-app erpnext --branch version-14
bench new-site mysite.local
# te pedirá MySQL root, contraseña de administrador del sitio, etc.
```

Instalar ERPNext en el sitio creado:
```bash
bench --site mysite.local install-app erpnext
```

---

## 6. Configuración para producción (Nginx, systemd, SSL)
Configurar producción (esto crea systemd services y Nginx config):
```bash
sudo bench setup production frappe
# o desde la cuenta 'frappe':
sudo -i -u frappe bash -c "cd ~/frappe-bench && bench setup production $(whoami)"
```
Este comando:
- Genera configuración de systemd para procesos (worker, scheduler, web).
- Crea bloque de Nginx y habilita el sitio.
- Reinicia servicios.

Habilitar HTTPS con Let's Encrypt (si el dominio apunta al servidor):
```bash
sudo bench setup nginx
sudo ln -s ~/frappe-bench/config/nginx.conf /etc/nginx/sites-enabled/frappe-bench.conf
sudo systemctl reload nginx
sudo bench --site mysite.local set-maintenance-mode on
sudo certbot --nginx -d tu_dominio
sudo bench --site mysite.local set-maintenance-mode off
```
(O usar `bench setup lets-encrypt` si disponible en tu versión de bench.)

---

## 7. Comandos útiles
- Iniciar/Detener servicios systemd:
    sudo systemctl status [frappe, nginx, redis-server, mariadb]
- Logs:
    cd ~/frappe-bench && bench --site mysite.local log
- Acceder interfaz web: http://tu_dominio o http://IP:8000 (si no estás con nginx)

---

## 8. Post-instalación y seguridad
- Cambia contraseñas por defecto (MySQL, usuario administrador).
- Configura firewall (ufw):
```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
```
- Backups regulares: usar `bench backup` o script automatizado.
- Monitorización y mantenimiento de MariaDB y Redis.

---

## 9. Resolución de problemas comunes
- Errores de versión de Node, Python o MariaDB: verifica compatibilidades de la rama Frappe que instalaste.
- wkhtmltopdf genera PDFs corruptos: usar binario parcheado recomendado.
- Permisos: asegurarse que los archivos de frappe-bench pertenezcan al usuario `frappe`.

---

Fuentes y referencias rápidas
- Documentación oficial Frappe/ERPNext (consultar para compatibilidades exactas de versiones).
- Páginas de descarga de wkhtmltopdf y NodeSource para Node.js.

Fin. Ajusta ramas/versión según la release de ERPNext que desees instalar y revisa la documentación oficial para cambios puntuales en comandos.