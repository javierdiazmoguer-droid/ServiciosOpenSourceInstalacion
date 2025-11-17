# Guía: Instalar Taiga en Ubuntu Server 24.04

Resumen rápido: instalaremos Taiga (backend Django + frontend dist), PostgreSQL, Redis, Gunicorn, Nginx y Certbot. Asumo un servidor limpio y un dominio apuntando al servidor (ej. taiga.example.com).

Requisitos (sudo):
- Ubuntu Server 24.04
- Usuario con sudo
- Dominio apuntando al servidor

1) Preparar el servidor
```
sudo apt update && sudo apt upgrade -y ✔️(Validado por Coder Nacho)
sudo apt install -y git curl build-essential python3 python3-venv python3-pip \
    libpq-dev postgresql postgresql-contrib redis-server nginx certbot python3-certbot-nginx ✔️ (Validado por Coder Nacho)
```

2) Crear usuario de sistema para Taiga
```
sudo adduser --system --group --home /home/taiga taiga
sudo mkdir -p /home/taiga
sudo chown taiga:taiga /home/taiga
```

3) PostgreSQL: crear base de datos y usuario
```
sudo -u postgres psql
# en psql:
CREATE USER taiga WITH PASSWORD 'tu_password_segura';
CREATE DATABASE taiga OWNER taiga;
\q
```
(Reemplaza la contraseña; si usas socket auth puedes ajustar.)

4) Redis
Redis ya estará instalado por apt; la configuración por defecto suele funcionar (localhost:6379).

5) Clonar el backend y preparar entorno Python
```
sudo -u taiga -H bash -lc '
cd ~
git clone https://github.com/taigaio/taiga-back.git
cd taiga-back
python3 -m venv .venv
source .venv/bin/activate
pip install -U pip setuptools wheel
pip install -r requirements.txt
'
```

6) Configuración básica del backend
Crear `settings/local.py` (ruta: /home/taiga/taiga-back/settings/local.py). Ejemplo mínimo:
```python
from .common import *

DEBUG = False
SECRET_KEY = "pone_una_secret_key_segura_aqui"
ALLOWED_HOSTS = ["taiga.example.com"]

DATABASES = {
        "default": {
                "ENGINE": "django.db.backends.postgresql",
                "NAME": "taiga",
                "USER": "taiga",
                "PASSWORD": "tu_password_segura",
                "HOST": "localhost",
                "PORT": "",
        }
}

# Redis / Celery
CELERY_BROKER_URL = "redis://localhost:6379/0"
CELERY_RESULT_BACKEND = "redis://localhost:6379/1"

MEDIA_URL = "/media/"
STATIC_ROOT = "/home/taiga/taiga-back/static"
MEDIA_ROOT = "/home/taiga/taiga-back/media"
```
Ajusta nombres y rutas según prefieras.

7) Migraciones y superusuario
```
sudo -u taiga -H bash -lc '
cd ~/taiga-back
source .venv/bin/activate
python manage.py migrate --noinput
python manage.py collectstatic --noinput
python manage.py createsuperuser
'
```

8) Frontend (taiga-front-dist)
```
sudo -u taiga -H bash -lc '
cd ~
git clone https://github.com/taigaio/taiga-front-dist.git
cd taiga-front-dist
# editar dist/conf.json:
#   "api": "https://taiga.example.com/api/v1/"
#   "eventsUrl": "wss://taiga.example.com/events"
#   "publicRegisterEnabled": true/false según prefieras
# Una vez editado, dejar la carpeta lista para servir vía nginx
'
```
El front-dist ya está compilado; Nginx servirá los archivos estáticos de esta carpeta.

9) Systemd: Gunicorn (backend) y Celery
Crear directorio socket y logs, cambiar propietario:
```
sudo mkdir -p /run/taiga
sudo mkdir -p /var/log/taiga
sudo chown taiga:taiga /run/taiga /var/log/taiga
```

Ejemplo de servicio Gunicorn: `/etc/systemd/system/taiga.service`
```
[Unit]
Description=Taiga (Gunicorn)
After=network.target

[Service]
User=taiga
Group=taiga
WorkingDirectory=/home/taiga/taiga-back
Environment="PATH=/home/taiga/taiga-back/.venv/bin"
ExecStart=/home/taiga/taiga-back/.venv/bin/gunicorn --workers 3 --bind unix:/run/taiga/taiga.sock taiga.wsgi:application
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Ejemplos de Celery worker y beat (opcional) `taiga-celery.service` y `taiga-celerybeat.service`:
- worker: ExecStart=... celery -A taiga.celery worker --loglevel=INFO
- beat: ExecStart=... celery -A taiga.celery beat --loglevel=INFO

Habilitar y arrancar:
```
sudo systemctl daemon-reload
sudo systemctl enable --now taiga.service
# y si creaste servicios celery:
sudo systemctl enable --now taiga-celery.service taiga-celerybeat.service
```

10) Nginx: proxy y servir front / media / static
Ejemplo de bloque de servidor `/etc/nginx/sites-available/taiga`:
```
server {
        listen 80;
        server_name taiga.example.com;

        root /home/taiga/taiga-front-dist/dist;

        access_log /var/log/nginx/taiga.access.log;
        error_log /var/log/nginx/taiga.error.log;

        location / {
                try_files $uri $uri/ /index.html;
        }

        location /api {
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_pass http://unix:/run/taiga/taiga.sock:;
        }

        location /static {
                alias /home/taiga/taiga-back/static;
        }

        location /media {
                alias /home/taiga/taiga-back/media;
        }

        location /events {
                proxy_pass http://127.0.0.1:8888; # si usas events via TCP/otro servicio, ajusta
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
        }
}
```
Habilitar y probar:
```
sudo ln -s /etc/nginx/sites-available/taiga /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

11) HTTPS con Certbot
```
sudo certbot --nginx -d taiga.example.com
# sigue las instrucciones para obtener cert
```

12) Comprobaciones finales
- Visita https://taiga.example.com
- Inicia sesión con el superusuario creado
- Revisa logs si algo falla:
    - /var/log/taiga (si configuraste)
    - journalctl -u taiga.service -f
    - /var/log/nginx/taiga.error.log

Notas y buenas prácticas
- Mantén SECRET_KEY en variable de entorno o gestor de secretos para producción.
- Haz copias de seguridad periódicas de la base de datos y media.
- Ajusta workers de Gunicorn según CPU/RAM.
- Considera usar un entorno virtualizado/contenerizado (Docker) si prefieres despliegues reproducibles.

Si quieres, te puedo generar:
- el archivo local.py completo con placeholders,
- el systemd unit file(s) listos para pegar,
- o la configuración completa de Nginx adaptada a tu dominio y rutas concretas.
