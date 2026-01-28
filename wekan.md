
# Instalación de Wekan en Ubuntu Server 24.04

## Requisitos Previos

- Ubuntu Server 24.04 LTS
- Usuario con permisos sudo
- Conexión a internet

## Paso 1: Actualizar el Sistema

```bash
sudo apt update
sudo apt upgrade -y
```

## Paso 2: Instalar Dependencias

```bash
sudo apt install -y curl wget git build-essential
```

## Paso 3: Instalar Node.js y npm

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
```

## Paso 4: Instalar MongoDB

```bash
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | sudo gpg --dearmor -o /usr/share/keyrings/mongodb-archive-keyring.gpg
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-archive-keyring.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt update
sudo apt install -y mongodb-org
sudo systemctl start mongod
sudo systemctl enable mongod
```

## Paso 5: Descargar e Instalar Wekan

```bash
cd /opt
sudo wget https://releases.wekan.team/wekan-latest.zip
sudo apt install -y unzip
sudo unzip wekan-latest.zip -d wekan
cd wekan
sudo npm install --production
```

## Paso 6: Configurar Wekan

Crear archivo `.env`:

```bash
sudo nano .env
```

Agregar:

```MONGO_URL=mongodb://localhost:27017/wekan
MONGO_OPLOG_URL=mongodb://localhost:27017/local?replicaSet=rs0
ROOT_URL=http://localhost:8080
PORT=8080
MAIL_URL=smtp://localhost
```
## Paso 7: ACTIVAR REPLICASET

```bash
sudo systemctl stop mongod
sudo mongod --dbpath /var/lib/mongodb --replSet rs0 --bind_ip_all &
sleep 5
mongosh --eval "rs.initiate()"
sudo systemctl start mongod
```

## Paso 8: Iniciar Wekan

```bash
sudo nano /etc/systemd/system/wekan.service

Contenido:
[Unit]
Description=Wekan Server
After=network.target mongod.service

[Service]
Type=simple
User=wekan
EnvironmentFile=/opt/wekan/.env
ExecStart=/usr/bin/node /opt/wekan/main.js
Restart=always

[Install]
WantedBy=multi-user.target

Habilitar:
sudo systemctl daemon-reload
sudo systemctl enable --now wekan
```
## Paso 9: Configuración mínima de Nginx
```bash
server {
    listen 80;
    server_name tu_dominio.com;

    location / {
        proxy_pass http://127.0.0.1:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
Acceder en: `http://localhost:8080`
