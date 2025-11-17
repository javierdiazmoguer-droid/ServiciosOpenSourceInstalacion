# Instalación de Rocket.Chat en Ubuntu 24.04

## Requisitos previos
- Ubuntu 22.04 (jammy jellyfish)
- Acceso root o sudo
- CPU: 4 núcleos
- Mínimo 2GB RAM / Recomendado 8GB
- Espacio recomendado: 30GB (20GB Rocket Chat + 10GB MongoDB)

## Pasos de instalación

### 1. Actualizar el sistema
```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Instalar dependencias
```bash
sudo apt install -y curl wget git build-essential
```

### 3. Instalar Node.js
```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
```

### 3. Instalar Deno
```bash
curl -fsSL https://deno.land/install.sh | sh
# Versiones de Deno compatible >=1.37.1 y <2.0.0
```


### 5. Instalar MongoDB
```bash
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo gpg --dearmor -o /usr/share/keyrings/mongodb-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/mongodb-archive-keyring.gpg] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list 
sudo apt update 
sudo apt install -y mongodb-org
sudo systemctl start mongodb
sudo systemctl enable mongodb
sudo systemctl status mongod 
```

### 6. Descargar Rocket.Chat
```bash
sudo mkdir /opt/Rocket.Chat
sudo curl -L https://releases.rocket.chat/latest/download -O rocket.chat.tar.gz
sudo tar -xvzf rocket.chat.tgz -C /opt/Rocket.Chat --strip-components=1
```

### 7. Instalar dependencias de Node
```bash
cd /opt/rocket.chat/programs/server
sudo npm install
```

### 8. Configurar Rocket.Chat
```bash
sudo useradd -r -m -U -d /opt/rocketchat -s /usr/sbin/nologin rocketchat
cd /opt/rocket.chat
sudo chown -R rocketchat:rocketchat /opt/rocket.chat
```

### 9. Crear servicio systemd
```bash
sudo nano /etc/systemd/system/rocketchat.service
```

Añadir contenido:
```ini
[Unit]
Description=Rocket.Chat server
After=network.target mongod.target

[Service]
Type=simple
User=rocketchat
Group=rocketchat
WorkingDirectory=/opt/Rocket.Chat
ExecStart=/usr/bin/node main.js
Restart=always
Environment=ROOT_URL=http://TU_IP:3000
Environment=MONGO_URL=mongodb://localhost:27017/rocketchat
Environment=PORT=3000

[Install]
WantedBy=multi-user.target
```

### 9. Iniciar servicio
```bash
sudo systemctl daemon-reload
sudo systemctl start rocketchat
sudo systemctl enable rocketchat
sudo systemctl status rocketchat
```

### 10. Instalar Nginx
```bash
sudo apt install -y nginx
sudo ufw allow 'Nginx Full'
sudo ufw reload
```

### 11. Configurar Nginx como proxy inverso
```bash
sudo nano /etc/nginx/sites-available/rocketchat
```
```nginx
server {
    listen 80;
    server_name TU_DOMINIO_O_IP;

    location / {
        proxy_pass http://localhost:3000/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 12. habilitar el sitio
```bash
sudo ln -s /etc/nginx/sites-available/rocketchat /etc/nginx/sites-enabled/rocketchat
sudo nginx -t
sudo systemctl restart nginx
```

Acceder a `http://localhost:3000`
