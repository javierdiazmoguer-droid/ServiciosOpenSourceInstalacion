# Jitsi-Meet-Installation

Este repositorio servirá para documentar la instalación y puesta en funcionamiento de Jitsi Server en una red local.

## Requerimientos iniciales

* **Ubuntu Server 24.04**
* **RAM:** more than 8GB
* **CPU:** Para un servidor básico, 4 núcleos sería suficiente
* **Espacio en disco:** al menos 20GB de espacio disponble en discos SSD.

## Pasos de preparación de requisitos
```bash
sudo apt update
sudo apt upgrade

sudo apt install apt-transport-https

sudo apt-add-repository universe
sudo apt update

sudo hostnamectl set-hostname meet.sansebastian.org
sudo nano /etc/hosts
Add 127.0.1.1 meet.sansebastian.org & save & exit
```

## Añadir repositorios
```bash
sudo curl -sL https://prosody.im/files/prosody-debian-packages.key -o /usr/share/keyrings/prosody-debian-packages.key

echo "deb [signed-by=/usr/share/keyrings/prosody-debian-packages.key] http://packages.prosody.im/debian $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/prosody-debian-packages.list

sudo apt install lua5.2

curl -sL https://download.jitsi.org/jitsi-key.gpg.key | sudo sh -c 'gpg --dearmor > /usr/share/keyrings/jitsi-keyring.gpg'

echo "deb [signed-by=/usr/share/keyrings/jitsi-keyring.gpg] https://download.jitsi.org stable/" | sudo tee /etc/apt/sources.list.d/jitsi-stable.list

sudo apt update

```

## Reglas de Firewall

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 10000/udp
sudo ufw allow 22/tcp
sudo ufw allow 3478/udp
sudo ufw allow 5349/tcp
sudo ufw enable


```
## Instalar Jitsi meet

```bash
sudo apt install jitsi-meet

## Configuración Jitsi meet

Nos pide el nombre de equipo:
meet.sansebastia.org

Nos pide Configuración de jitsi-meet-web-config
Elegimos Le
