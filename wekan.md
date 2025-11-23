
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
sudo apt install -y curl wget git build-essential gnupg2 ca-certificates lsb-release snapd
```

## Paso 3: Instalar Node.js y npm
### Instalación de Node.js (OPCIONAL — sólo si vas a compilar Wekan manualmente)
### Si vas a instalar Wekan mediante Snap, puedes omitir este paso.
### Para instalar Node.js 20:

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
node -v
npm -v
```

## Paso 4: Instalar MongoDB
### Añadir clave GPG y repo oficial de MongoDB (Ubuntu 24.04 "noble")

```bash
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc \
  | gpg --dearmor \
  | sudo tee /usr/share/keyrings/mongodb-archive-keyring.gpg > /dev/null

echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-archive-keyring.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/7.0 multiverse" \
  | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

sudo apt update
sudo apt install -y mongodb-org

sudo systemctl enable --now mongod
sudo systemctl status mongod --no-pager
```

## Paso 5: Descargar e Instalar Wekan
### Instalación recomendada de Wekan (Snap)
### Snap instala WeKan como un servicio gestionado y con actualizaciones automáticas.

```bash
sudo snap install wekan --channel=latest/candidate
```

## Paso 6: Configurar Wekan
### Configurar URL y puerto (ejemplo)

```bash
sudo snap set wekan root-url="http://TU_IP_O_DOMINIO"
```

### Para servir en puerto 80 (HTTP) (si quieres usar 80 en lugar de 3000)

```bash
sudo snap set wekan port='80'
```

## Paso 7: Iniciar Wekan
### Comprobar estado del servicio snap

```bash
sudo snap services wekan
```

Acceder en: `http://localhost:3000`

## Paso 8: Verificación y firewall

### Verificar que Wekan está corriendo (snap o manual)
```bash
ss -tulpn | grep -E '3000|80|443' || sudo snap services wekan
```

### Si usas UFW y ejecutas Wekan en 3000:
```bash
sudo ufw allow 3000/tcp
```

### Si usas proxy inverso (Nginx) o snap configurado para puerto 80/443:
```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

# Acceso:
### Si accedes desde la misma máquina: http://localhost:3000 (o http://localhost si configuraste puerto 80)
### Desde otra máquina: http://IP_DEL_SERVIDOR:3000  (o http://IP_DEL_SERVIDOR si usas puerto 80)
