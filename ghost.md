# Guía de instalación de Ghost en Ubuntu 24.04

## Resumen

Esta guía describe los pasos completos para instalar Ghost (plataforma de blogs) en un servidor con Ubuntu 24.04. Cubre la instalación de dependencias, Node.js, Ghost-CLI, configuración de Nginx como proxy inverso y permisos. Está pensada para una instalación en producción básica.

Requisitos mínimos:
- Ubuntu 24.04 con acceso sudo
- 1 CPU, 1 GB RAM (2 GB recomendados para producción)
- Dominio público apuntando al servidor (opcional pero recomendado)

---

## 1. Actualizar el sistema

Abrir una terminal y ejecutar:

```bash
sudo apt update && sudo apt upgrade -y
```

## 2. Crear usuario dedicado (opcional pero recomendado)

Es buena práctica ejecutar Ghost con un usuario no root. Se crea el usuario `ghost` y se le da ownership del directorio de trabajo:

```bash
sudo adduser --system --quiet --group --shell /bin/false ghost
sudo mkdir -p /var/www/ghost
sudo chown ghost:ghost /var/www/ghost
sudo chmod 775 /var/www/ghost
```

## 3. Instalar dependencias del sistema

Instalar `curl`, `nginx`, `mysql-server` (o usar SQLite para pruebas), y herramientas básicas:

```bash
sudo apt install -y nginx curl git build-essential
sudo apt install -y mysql-server # opcional: para usar MySQL
```

Si vas a usar MySQL/MariaDB, asegúrate de configurarlo y crear la base de datos/usuario para Ghost (se muestra más abajo si eliges MySQL).

## 4. Instalar Node.js (versión recomendada)

Ghost requiere una versión compatible de Node.js. Aquí instalamos la última disponble:

```bash
# Node.js versión recomendada por Ghost (LTS compatible)
NODE_MAJOR=20
curl -fsSL https://deb.nodesource.com/setup_$NODE_MAJOR.x | sudo -E bash -
sudo apt install -y nodejs

# Comprobar versiones
node -v
npm -v
```

## 5. Instalar Ghost CLI

Ghost CLI facilita la instalación y administración de Ghost.

```bash
sudo npm install -g ghost-cli@latest --unsafe-perm
```

Verifica la instalación:

```bash
ghost --version
```

## 6. Preparar el directorio de instalación

Instala Ghost dentro de `/var/www/ghost` ejecutando como el usuario `ghost` (o como tu usuario con permisos sobre el directorio).

```bash
sudo chown -R ghost:ghost /var/www/ghost
sudo -u ghost -H bash
cd /var/www/ghost
```

Alternativamente, si no creaste el usuario `ghost`:

```bash
cd /var/www/ghost
sudo chown $USER:$USER /var/www/ghost
```

## 7. Instalar Ghost

Dentro del directorio `/var/www/ghost` ejecuta:

```bash
ghost install
```

El instalador interactivo hará lo siguiente por defecto:
- descargará la versión estable de Ghost
- instalará dependencias npm localmente
- configurará systemd para ejecutar Ghost como servicio
- configurará Nginx como proxy inverso (si está instalado)
- generará certificados SSL con Let's Encrypt (si el dominio apunta al servidor)

Durante el proceso se te pedirá:
- URL del sitio (ej. https://tu_dominio.com)
- configuración de correo (puedes dejar en blanco y configurarlo después)

Si prefieres no usar la instalación interactiva completa, puedes instalar manualmente con `ghost install --local` o seguir la documentación oficial.

## 8. (Opcional) Configurar MySQL

Si vas a usar MySQL, crea la base de datos y usuario:

```bash
sudo mysql -u root -p

CREATE DATABASE ghostdb
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_general_ci;
CREATE USER 'ghostuser'@'localhost' IDENTIFIED BY 'tu_password_segura';
GRANT ALL PRIVILEGES ON ghostdb.* TO 'ghostuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

Durante `ghost install` puedes indicar que usarás MySQL y proporcionar las credenciales.

## 9. Configurar Nginx como proxy inverso (si no lo hizo Ghost CLI)

Si prefieres configurar Nginx manualmente o Ghost CLI no lo configuró, crea un archivo de sitio:

```bash
sudo nano /etc/nginx/sites-available/ghost
```

Ejemplo de configuración mínima:

```nginx
server {
    listen 80;
    server_name tu_dominio.com www.tu_dominio.com;

    location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header Host $http_host;

    proxy_redirect off;
    proxy_pass http://127.0.0.1:2368;
}

    access_log /var/log/nginx/ghost_access.log;
    error_log  /var/log/nginx/ghost_error.log;
}
```

Habilitar y comprobar la configuración:

```bash
sudo ln -s /etc/nginx/sites-available/ghost /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

## 10. Configurar SSL con Let's Encrypt (opcional pero recomendado)

Si el dominio apunta al servidor, puedes obtener certificados con Certbot o dejar que Ghost CLI lo haga automáticamente.

Usando Certbot:

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx --redirect -d tu_dominio.com -d www.tu_dominio.com
```

## 11. Administrar el servicio Ghost

Ghost se instala como servicio systemd. Comandos útiles:

```bash
sudo systemctl status ghost_*  # ver estado del servicio Ghost (nombre puede variar)
ghost stop
ghost start
ghost restart
ghost log
```

Ejecuta estos comandos dentro del directorio de instalación (`/var/www/ghost`).

## 12. Comprobaciones finales

- Abre `http://tu_dominio.com` o `https://tu_dominio.com` en el navegador.
- Accede al panel de administración en `https://tu_dominio.com/ghost` para crear la cuenta inicial.
- Revisa los logs si algo falla: `ghost log` y `sudo journalctl -u ghost_yourdomain`.

## 13. Buenas prácticas y notas

- Haz copias de seguridad periódicas de la base de datos y del contenido (`/var/www/ghost/content`).
- Mantén Node.js y Ghost actualizados dentro de las versiones soportadas.
- Usa un firewall (ufw) para limitar accesos y abrir puertos 80/443 sólo.

Ejemplo rápido de firewall:

```bash
sudo apt install -y ufw
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

## Recursos

- Documentación oficial de Ghost: https://ghost.org/docs/
- Ghost-CLI: https://ghost.org/docs/ghost-cli/

---

Si quieres, puedo:
- añadir la configuración completa de Nginx con seguridad reforzada;
- preparar un script automático de instalación para Ubuntu 24.04;
- incluir pasos para migrar desde otra plataforma.

