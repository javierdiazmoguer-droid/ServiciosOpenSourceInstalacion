# Instalación de Zammad en Ubuntu 24.04

## Requisitos previos
- Ubuntu 24.04 LTS
- Mínimo 6GB o 10GB RAM si quieres Elasticsearch en el mismo servidor
- 2 CPUs
- 20GB espacio en disco
- Privilegios de superusuario (sudo)

## Pasos de instalación

### 1. Actualizar el sistema
```bash
sudo apt update
sudo apt upgrade -y
```

### Configuración de firewall (opcional si nos falla el comando)

Si tu Ubuntu tiene UFW activo, asegúrate de permitir el tráfico HTTP y HTTPS para que la interfaz web de Zammad sea accesible:

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw reload
```

### 2. Instalar dependencias
```bash
sudo apt install wget apt-transport-https gnupg2 -y
```

### 3. Añadir repositorio de Zammad
```bash
wget -qO- https://dl.packager.io/srv/zammad/zammad/key \
  | gpg --dearmor \
  | sudo tee /usr/share/keyrings/zammad.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/zammad.gpg] https://dl.packager.io/srv/zammad/zammad/stable/ubuntu/24.04 main" | sudo tee /etc/apt/sources.list.d/zammad.list
```

### 4. Instalar Zammad
```bash
sudo apt update
sudo apt install zammad -y
```

### 5. Iniciar servicios
```bash
sudo systemctl start zammad
sudo systemctl enable zammad
```

### 6. Verificar instalación
```bash
sudo systemctl status zammad
ss -tulpn | grep 3000
```

### 7. Acceder a Zammad
- Abrir navegador web
- Acceder a: `(http://IP_DEL_SERVIDOR:3000)`
- Completar el asistente de configuración inicial

## Configuración inicial
1. Crear cuenta de administrador
2. Configurar zona horaria
3. Configurar idioma
4. Establecer ajustes de correo electrónico

## Notas importantes
- El puerto predeterminado es 3000
- Se recomienda configurar un proxy inverso (nginx/apache)
- Realizar copias de seguridad regulares
- Mantener el sistema actualizado

## Referencias
