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
sudo python3.7 -m pip install -U pip
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
And it working... You can see your django project
Ok. First step was ended. We have django on our server, but this is start
At the next we should install gunicorn for starting django.

