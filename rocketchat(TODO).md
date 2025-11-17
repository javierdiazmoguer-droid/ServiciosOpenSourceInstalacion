# Instalación de Rocket.Chat en Ubuntu 24.04

## Requisitos previos
- Ubuntu 22.04 (jammy jellyfish)
- Acceso root o sudo
- Mínimo 2GB RAM
- 10GB espacio en disco

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

### 4. Instalar MongoDB
```bash
sudo apt install -y mongodb-server
sudo systemctl start mongodb
sudo systemctl enable mongodb
```

### 5. Descargar Rocket.Chat
```bash
cd /opt
sudo wget https://releases.rocket.chat/latest/download/tar.gz -O rocket.chat.tar.gz
sudo tar -xzf rocket.chat.tar.gz
sudo mv bundle rocket.chat
```

### 6. Instalar dependencias de Node
```bash
cd /opt/rocket.chat/programs/server
sudo npm install
```

### 7. Configurar Rocket.Chat
```bash
cd /opt/rocket.chat
sudo chown -R rocketchat:rocketchat /opt/rocket.chat
```

### 8. Crear servicio systemd
```bash
sudo nano /etc/systemd/system/rocketchat.service
```

Añadir contenido:
```ini
[Unit]
Description=Rocket.Chat
After=network.target mongodb.service

[Service]
ExecStart=/usr/bin/node /opt/rocket.chat/main.js
Restart=always
User=rocketchat
Environment="PORT=3000" "MONGO_URL=mongodb://localhost:27017/rocketchat"

[Install]
WantedBy=multi-user.target
```

### 9. Iniciar servicio
```bash
sudo systemctl start rocketchat
sudo systemctl enable rocketchat
```

Acceder a `http://localhost:3000`
