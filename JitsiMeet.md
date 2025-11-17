# Jitsi-Meet-Installation

Este repositorio servirá para documentar la instalación y puesta en funcionamiento de Jitsi Server en una red local.

## Requerimientos iniciales

* **Ubuntu Server 24.04**
* **RAM:** 8GB o más.
* **CPU:** Para un servidor básico, 4 núcleos sería suficiente.
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

Add 192.168.0.1 meet.sansebastian.org & save & exit
```
> **OJO**: Hay que tener en cuenta que la dirección asociada al hostname no puede ser de loopback para que el servidor resuelva las solicitudes.

## Instalación de OpenJDK17
```bash
wget https://download.java.net/java/GA/jdk17.0.2/dfd4a8d0985749f896bed50d7138ee7f/8/GPL/openjdk-17.0.2_linux-x64_bin.tar.gz
tar -xvzf openjdk-17.0.2_linux-x64_bin.tar.gz

sudo mv jdk-17.0.2 /opt/

sudo nano /etc/profile.d/jdk17.sh
export JAVA_HOME=/opt/jdk-17.0.2
export PATH=$JAVA_HOME/bin:$PATH

source /etc/profile.d/jdk17.sh
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

configuración de jitsi-videobridge2
meet.sansebastia.org

Nos pide Configuración de jitsi-meet-web-config
Elegimos Crear un certificado autofirmado
```

```bash
sudo systemctl restart prosody
sudo systemctl restart jicofo
sudo systemctl restart jitsi-videobridge2
```
