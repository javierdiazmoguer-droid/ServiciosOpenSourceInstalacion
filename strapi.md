# Instalación de Strapi en Ubuntu 24.04

Resumen rápido: guía para instalar Strapi (v4+) en Ubuntu 24.04, desde un entorno de desarrollo (SQLite / quickstart) hasta una instalación de producción con PostgreSQL, PM2 y Nginx.

## Requisitos
- Ubuntu 24.04
- Usuario con sudo
- Puerto 1337 libre (por defecto)
- Node.js (solo compatible LTS 20/22)
- npm (v9+) o Yarn
- (Producción) PostgreSQL, Nginx, Certbot, PM2

---

## 1) Actualizar sistema e instalar utilidades
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl git build-essential python3 python3-pip
```

## 2) Instalar Node.js (recomendado: nvm)
Instalar nvm y Node LTS (ej. 18):
```bash
curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash
# Cargar nvm (o cerrar/abrir terminal)
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

nvm install 18
nvm use 18
node -v
npm -v
```

(Opcional) instalar Yarn:
```bash
npm install -g yarn
```

## 3) Crear un proyecto Strapi (modo rápido / desarrollo)
Quickstart (usa SQLite y arranca inmediatamente):
```bash
npx create-strapi-app@latest mi-strapi --quickstart
# o con yarn
# yarn create strapi-app mi-strapi --quickstart
```
Acceder: http://localhost:1337/admin

## 4) Configurar Strapi con PostgreSQL (entorno de producción recomendado)
Instalar PostgreSQL:
```bash
sudo apt install -y postgresql postgresql-contrib libpq-dev
sudo -u postgres psql -c "CREATE USER strapi_user WITH PASSWORD 'tu_contraseña';"
sudo -u postgres psql -c "CREATE DATABASE strapi_db OWNER strapi_user;"
```

Crear proyecto (interactivo) o crear y luego configurar la base de datos:
- Al crear con `npx create-strapi-app@latest mi-strapi`, seleccionar "custom" y elegir PostgreSQL, o
- Crear con `--no-run` y editar configuración.

Ejemplo de variables de entorno (archivo .env en la raíz del proyecto):
```
NODE_ENV=production
DATABASE_CLIENT=postgres
DATABASE_HOST=127.0.0.1
DATABASE_PORT=5432
DATABASE_NAME=strapi_db
DATABASE_USERNAME=strapi_user
DATABASE_PASSWORD=tu_contraseña
DATABASE_SSL=false
```

Si usas `DATABASE_URL`:
```
DATABASE_URL=postgres://strapi_user:tu_contraseña@127.0.0.1:5432/strapi_db
```

Luego instalar dependencias y construir:
```bash
cd mi-strapi
npm install
npm run build
```

Arrancar en modo producción:
```bash
NODE_ENV=production npm run start
# o con yarn
# NODE_ENV=production yarn start
```

## 5) Ejecutar como servicio (PM2)
Instalar PM2 y registrar la app:
```bash
npm install -g pm2
cd ~/mi-strapi
NODE_ENV=production npm run build
pm2 start npm --name "strapi" -- start
pm2 save
pm2 startup systemd
# ejecutar el comando que muestre pm2 startup para habilitar el servicio
```

Ver logs:
```bash
pm2 logs strapi
```

## 6) Configurar Nginx como reverse proxy (HTTP/HTTPS)
Instalar nginx:
```bash
sudo apt install -y nginx
```
Ejemplo de bloque server (crear /etc/nginx/sites-available/mi-strapi):
```
server {
    listen 80;
    server_name example.com; # cambiar por tu dominio

    client_max_body_size 100M;

    location / {
        proxy_pass http://127.0.0.1:1337;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
Habilitar y reiniciar:
```bash
sudo ln -s /etc/nginx/sites-available/mi-strapi /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

Habilitar HTTPS con Certbot:
```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d example.com
```

## 7) Comandos útiles
- Desarrollo: `npm run develop`
- Build producción: `npm run build`
- Start producción: `NODE_ENV=production npm run start`
- Logs: `pm2 logs strapi` o revisar `journalctl`/systemd si configuraste servicio

## 8) Seguridad y recomendaciones
- No correr Strapi como root.
- Usar base de datos dedicada y credenciales seguras.
- Hacer backups regulares de la base de datos.
- Habilitar HTTPS en producción.
- Mantener Node y dependencias actualizadas, probar en staging antes de prod.

---

Si necesitas un ejemplo de archivo de configuración `config/database.js` para Strapi o el bloque completo de systemd para el servicio, indícalo y lo agrego.