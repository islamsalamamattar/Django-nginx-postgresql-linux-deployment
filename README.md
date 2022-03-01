# Setting up Django with PostgreSQL, Nginx, and Gunicorn on Ubuntu 20.04
=========================================================================

Ubuntu 20.04 server configured with a basic firewall

# Creating non-root user with sudo privilege
adduser <username>
usermod -aG sudo <username>  # add user to sudo group
su - <username> #switch to user

sudo ls -la /root  #test sudo previlages 

# Installing the Necessary Packages

sudo apt update
sudo apt install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx curl

# PostgreSQL Database and User	

sudo -u postgres psql
	
CREATE DATABASE database;
CREATE USER admin WITH PASSWORD 'db@123';
ALTER ROLE admin SET client_encoding TO 'utf8';
ALTER ROLE admin SET default_transaction_isolation TO 'read committed';
ALTER ROLE admin SET timezone TO 'UTC';
GRANT ALL PRIVILEGES ON DATABASE database TO admin;
\q

# Python Virtual Environment

sudo -H pip3 install --upgrade pip
sudo -H pip3 install virtualenv

## create a virtual environment
cd ~
virtualenv <project_env>
source <project_env>/bin/activate

# install Django, Gunicorn, and psycopg2 (PostgreSQL adapter)
pip3 install django gunicorn psycopg2-binary



# Django project
intsall or copy Django project with dependencies in ~/<project_env>/<project_name>/

# Adjusting project settings

settings.py
'''python
import os
from pathlib import Path

# Build paths inside the project like this: BASE_DIR / 'subdir'.
BASE_DIR = Path(__file__).resolve().parent.parent

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = 'xxxxx'

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = False
ALLOWED_HOSTS = ['<server_ip_or_domain_name_1>',' <server_ip_or_domain_name_2>']


# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'core.urls'

TEMPLATE_DIR = os.path.join(BASE_DIR, "templates") 
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [TEMPLATE_DIR],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'core.wsgi.application'

# Database
# https://docs.djangoproject.com/en/3.2/ref/settings/#databases

DATABASES = {
    'default': {
    'ENGINE': 'django.db.backends.postgresql_psycopg2',
    'NAME': '<database_name>',
    'USER': '<database_user>',
    'PASSWORD': 'password123',
    'HOST': 'localhost',
    'PORT': '',
    }
}

# Password validation
# https://docs.djangoproject.com/en/3.2/ref/settings/#auth-password-validators

AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]

# Internationalization
# https://docs.djangoproject.com/en/3.2/topics/i18n/

LANGUAGE_CODE = 'en-us'
TIME_ZONE = 'UTC'
USE_I18N = True
USE_L10N = True
USE_TZ = True

# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/3.2/howto/static-files/

STATIC_URL = '/static/'
STATICFILES_DIRS = [os.path.join(BASE_DIR, "static"), '/home/<username>/<project_env>/lib/python3.8/site-packages/django/contrib/admin/static'
STATIC_ROOT = 'static_root/'
LOGIN_URL = 'accounts/login/'

# Default primary key field type
# https://docs.djangoproject.com/en/3.2/ref/settings/#default-auto-field

DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'

# CSRF HTTPS configurations
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
'''



# Enable firewall
sudo ufw allow 8000
sudo ufw allow ssh
sudo ufw enable
sudo ufw status

# create database tables, admin user, and start server
python3 manage.py makemigrations
python3 manage.py migrate
python3 manage.py createsuperuser

python3 manage.py runserver 0.0.0.0:8000
visit --->     http://<server_ip_or_domain_name>:8000/admin
--> press “Ctrl + C” from the terminal window to stop the server.

gunicorn --bind 0.0.0.0:8000 <project_name>.wsgi    #from project folder
visit --->     http://<server_ip_or_domain_name>:8000/admin
--> press “Ctrl + C” from the terminal window to stop the Gunicorn server.
	
deactivate    #Exiting virtual environment





# Gunicorn Socket and Service Files
sudo nano /etc/systemd/system/gunicorn.socket
# =========================== gunicor.socket ========================= #
[Unit]
Description=gunicorn socket
[Socket]
ListenStream=/run/gunicorn.sock
 
[Install]
WantedBy=sockets.target
# ============================= end gunicor.socket ============================== #

sudo nano /etc/systemd/system/gunicorn.service
# ============================= gunicorn.service ==================================== #
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target
 
[Service]
User=<user>
Group=<user>
WorkingDirectory=/home/<username>/<project_env>/<project_name>
ExecStart=/home/<username>/<project_env>/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          <project_name>.wsgi:application
 
[Install]
WantedBy=multi-user.target
# ========================== end gunicorn.service ================================ #

# Enabling Gunicorn socket
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket
sudo systemctl status gunicorn.socket
file /run/gunicorn.sock    #check the existence of the socket file
curl --unix-socket /run/gunicorn.sock localhost  # test
sudo journalctl -u gunicorn.socket   #checking detailed log

# gunicorn Socket activation
sudo systemctl status gunicorn
sudo systemctl daemon-reload
sudo systemctl restart gunicorn




# Configuring Nginx
sudo nano /etc/nginx/sites-available/<project_name>
'''bash
server {
 
listen 80;
listen [::]:80;
listen 443 ssl http2;
listen [::]:443 ssl http2;

ssl on;
ssl_certificate /etc/ssl/certs/cert.pem;
ssl_certificate_key /etc/ssl/private/key.pem;

root /var/www/html;
index index.php index.html index.htm;

server_name <server_ip_or_domain_name_1> <server_ip_or_domain_name_2>;

location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;  # <-
        proxy_set_header Host $http_host;
        proxy_redirect off;

        if (!-f $request_filename) {
            proxy_pass http://unix:/run/gunicorn.sock;
            break;
        }
    }

location = /favicon.ico { access_log off; log_not_found off; }
 
location /static/ {
root /home/<username>/<project_env>/<project_name>;
}
  
}
'''
