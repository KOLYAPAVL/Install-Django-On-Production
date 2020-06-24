# Django (Debiah) install on production server
This my tasklist about how to install django on production server and connect libraries for correct working it.

## Requirements

```
1) You should get root permisions for your users (def. root)
2) You should have at least some skills to use the console and vim | nano | another :)
3) You should be able to use htop

But if you can't something in this list you can try to make it in first time
```

## Install Packages / Creating user

Updating data

```
sudo apt-get update
sudo apt-get upgrade
```

Creating user WWW (all settings will do with this user)

```
adduser www
usermod -aG sudo www
getent group sudo
```

Installing Default Packages

```
sudo apt-get install -y vim mosh tmux htop git curl wget unzip zip gcc build-essential make
sudo apt-get install -y zsh tree redis-server nginx zlib1g-dev libbz2-dev libreadline-dev llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev liblzma-dev python3-dev python-imaging python3-lxml libxslt-dev python-libxml2 python-libxslt1 libffi-dev libssl-dev python-dev gnumeric libsqlite3-dev libpq-dev libxml2-dev libxslt1-dev libjpeg-dev libfreetype6-dev libcurl4-openssl-dev supervisor
```

Installing oh-my-zsh

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

vim ~/.zshrc
   alias cls="clear"
   
chsh -s $(which zsh) 
```

## Install Python
Installing Python (I'm install Python3.7, if you want anouther version change 3.7 on your version)

```
mkdir ~/code
wget https://www.python.org/ftp/python/3.7.3/Python-3.7.3.tgz
tar xvf Python-3.7.*
cd Python-3.7.3 
mkdir ~/.python
./configure --enable-optimizations --prefix=/home/www/.python
make -j8
sudo make altinstall
```

After installing Python you should add command python3.7 (I'm install Python3.7, if you want anouther version change 3.7 on your version)

```
sudo rm -rf Python-3.7.3.tgz Python-3.7.3 
vim ~/.zshrc
   export PATH=$PATH:/home/www/.python/bin
. ~/.zshrc
python3.7
```

Now we should install PIP

```
sudo /home/www/.python/bin/python3.7 -m pip install -U pip
```

## Adding project to your server

Creating directory and virtual env
```
1) My project will be in /home/www/projects/project - (pls. remember this path)
   mkdir projects
   cd projects
   mkdir project
   cd project

2) In project directory I'm creating virtual env, you can do this like this:
   python3.7 -m venv env
    
   And activate it
   source env/bin/activate
3) The next you should pull your project (my project name = pr)

============================================================================================
PROJECT_DIRECTORY = /home/www/projects/project
DJANGO = /home/www/projects/project/pr
ENV = /home/www/projects/project/env
============================================================================================
p.s. You can use another paths if you want it
```


## First start Django

```
python3.7 manage.py makemigrations
python3.7 manage.py migrate
python3.7 manage.py runserver 0.0.0.0:8000
```

The next you should open your browser and write in search: yourdomain:8000
And it working... You can see your django project.
Ok, First step was ended. We have django on our server, but this is start.
At the next we should install gunicorn for starting django.



## Installing PostgreSQL

Install PostgreSQL 11 and configure locales.

```
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
RELEASE=$(lsb_release -cs)
echo "deb http://apt.postgresql.org/pub/repos/apt/ ${RELEASE}"-pgdg main | sudo tee  /etc/apt/sources.list.d/pgdg.list
sudo apt update
sudo apt -y install postgresql-11
sudo localedef ru_RU.UTF-8 -i ru_RU -fUTF-8 
export LANGUAGE=ru_RU.UTF-8
export LANG=ru_RU.UTF-8
export LC_ALL=ru_RU.UTF-8
sudo locale-gen ru_RU.UTF-8
sudo dpkg-reconfigure locales
```

Change postges password, create clear database named dbms_db

```
sudo passwd postgres
su - postgres
export PATH=$PATH:/usr/lib/postgresql/11/bin
createdb --encoding UNICODE dbms_db --username postgres
exit
```

Create dbms db user and grand privileges to him
```
sudo -u postgres psql
postgres=# ...
create user dbms with password 'some_password';
ALTER USER dbms CREATEDB;
grant all privileges on database dbms_db to dbms;
\c dbms_db
GRANT ALL ON ALL TABLES IN SCHEMA public to dbms;
GRANT ALL ON ALL SEQUENCES IN SCHEMA public to dbms;
GRANT ALL ON ALL FUNCTIONS IN SCHEMA public to dbms;
CREATE EXTENSION pg_trgm;
ALTER EXTENSION pg_trgm SET SCHEMA public;
UPDATE pg_opclass SET opcdefault = true WHERE opcname='gin_trgm_ops';
\q
exit
```

Run SQL dump, if you have
```
psql -h localhost dbms_db dbms < dump.sql
```

Add PostgreSQL to Django
```
pip install psycopg2

#settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'DB_NAME' ,
        'USER' : 'DB_USER',
        'PASSWORD' : 'DB_PASSWORD',
        'HOST' : '127.0.0.1',
        'PORT' : '5432',
    }
}

python3.7 manage.py makemigrations
python3.7 manage.py migrate
```

## Gunicorn

Download gunicorn

```
pip install gunicorn
```

Creating in DJANGO gunicorn config
```
#workers=CPU Count * 2 + 1
#gunicorn_config.py
command = '/home/www/projects/project/env/bin/gunicorn'
pythonpath = '/home/www/projects/project/pr'
bind = '127.0.0.1:8001'
workers = 3
user = 'www'
limit_request_fields = 32000
limit_request_field_size = 0
raw_env = 'DJANGO_SETTINGS_MODULE=pr.settings'
```

Creating gunicornStarter in PROJECT

```
mkdir bin
sudo vim start_gunicorn.sh
#start_gunicorn.sh
#!/bin/bash
source /home/www/projects/project/env/bin/activate
exec gunicorn  -c "/home/www/projects/project/pr/gunicorn_config.py" pr.wsgi
```

Starting Proccess

```
chmod +x bin/start_gunicorn.sh
. ./bin/start_gunicorn.sh
```

And you will see if success
```
[2020-06-24 19:11:09 +0000] [9506] [INFO] Starting gunicorn 20.0.4
[2020-06-24 19:11:09 +0000] [9506] [INFO] Listening at: http://127.0.0.1:8001 (9506)
[2020-06-24 19:11:09 +0000] [9506] [INFO] Using worker: sync
[2020-06-24 19:11:09 +0000] [2901] [INFO] Booting worker with pid: 2901
[2020-06-24 19:11:09 +0000] [2902] [INFO] Booting worker with pid: 2902
[2020-06-24 19:11:09 +0000] [2903] [INFO] Booting worker with pid: 2903
[2020-06-24 19:11:09 +0000] [2904] [INFO] Booting worker with pid: 2904
[2020-06-24 19:11:09 +0000] [2905] [INFO] Booting worker with pid: 2905
[2020-06-24 19:11:09 +0000] [2906] [INFO] Booting worker with pid: 2906
[2020-06-24 19:11:10 +0000] [2907] [INFO] Booting worker with pid: 2907
[2020-06-24 19:11:10 +0000] [2908] [INFO] Booting worker with pid: 2908
[2020-06-24 19:11:10 +0000] [2909] [INFO] Booting worker with pid: 2909
```

## NGINX

Go to nginx config
```
sudo vim /etc/nginx/sites-enabled
# chouse default
```

Nginx config

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    
    root /var/www/html;
    
    index index.html index.htm index.nginx-debian.html;
    
    server_name _;
    
    location / {
        proxy_pass http://127.0.0.1:8001;
        proxy_set_header X-Forwarded-Host $server_name;
        proxy_set_header X-Real-IP $remote_addr;
        add_header P3P 'CP="ALL DSP COR PSAa PSDa OUR NOR ONL UNI COM NAV"';
        add_header Access-Control-Allow-Origin *;
    }
    location /static {
        autoindex on;
        alias /home/www/projects/project/pr/static/;
    }
    location /media {
        autoindex on;
        alias /home/www/projects/project/pr/media/;
    }
    location /sockets {
        try_files $uri @proxy_to_ws;
    }
    location @proxy_to_ws {
        proxy_pass http://127.0.0.1:8002;
        
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Host $server_name;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

And restart Nginx
```
sudo service nginx stop
sudo service nginx start
```

p.s. If you see error ```Job for nginx.service failed because the control process exited with error code.
See "systemctl status nginx.service" and "journalctl -xe" for details``` try to open log ```systemctl status nginx.service``` and check error
