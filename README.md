```markdown
# Setting up Django with PostgreSQL, Nginx, and Gunicorn on Ubuntu 20.04
=========================================================================

Ubuntu 20.04 server configured with a basic firewall.

## Creating non-root user with sudo privilege

```bash
adduser admin
usermod -aG sudo admin # add user to sudo group
su - admin #switch to user

sudo ls -la /root  #test sudo privileges
```

## Installing the Necessary Packages

```bash
sudo apt update
sudo apt upgrade
sudo apt install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx curl
```

## PostgreSQL Database and User

```bash
sudo -u postgres psql

> CREATE DATABASE database;
> CREATE USER admin WITH PASSWORD 'db@123';
> ALTER ROLE admin SET client_encoding TO 'utf8';
> ALTER ROLE admin SET default_transaction_isolation TO 'read committed';
> ALTER ROLE admin SET timezone TO 'UTC';
> GRANT ALL PRIVILEGES ON DATABASE database TO admin;
> \q
```

## Python Virtual Environment

```bash
sudo -H pip3 install --upgrade pip
sudo -H pip3 install virtualenv

# create a virtual environment
cd ~
virtualenv <project_env>
source <project_env>/bin/activate

# install Django, Gunicorn, and psycopg2 (PostgreSQL adapter)
pip3 install django gunicorn psycopg2-binary
```

## Django project

Install or copy Django project with dependencies in `~/<project_env>/<project_name>/`

## Adjusting project settings

### settings.py

```python
# (Content of settings.py)
```

## Enable firewall

```bash
sudo ufw allow 8000
sudo ufw allow ssh
sudo ufw enable
sudo ufw status
```

## Create database tables, admin user, and start server

```bash
python3 manage.py makemigrations
python3 manage.py migrate
python3 manage.py createsuperuser

python3 manage.py runserver 0.0.0.0:8000
# visit ---> http://<server_ip_or_domain_name>:8000/admin
# press “Ctrl + C” from the terminal window to stop the server.

gunicorn --bind 0.0.0.0:8000 <project_name>.wsgi    # from the project folder
# visit ---> http://<server_ip_or_domain_name>:8000/admin
# press “Ctrl + C” from the terminal window to stop the Gunicorn server.

deactivate    # Exiting virtual environment
```

## Gunicorn Socket and Service Files

```bash
# (Content of gunicorn.socket and gunicorn.service)
```

### Enabling Gunicorn socket

```bash
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket
sudo systemctl status gunicorn.socket
file /run/gunicorn.sock    # check the existence of the socket file
curl --unix-socket /run/gunicorn.sock localhost  # test
sudo journalctl -u gunicorn.socket   # checking detailed log
```

### Gunicorn Socket activation

```bash
sudo systemctl status gunicorn
sudo systemctl daemon-reload
sudo systemctl restart gunicorn
```

## Configuring Nginx

```bash
# (Content of Nginx configuration file)
```
```

Make sure to replace placeholders like `<project_env>`, `<project_name>`, `<database_name>`, `<database_user>`, `<server_ip_or_domain_name_1>`, `<server_ip_or_domain_name_2>` with your actual values.
