# aws-config-flask-python3-udacity-proj-5
Project 5 by udacity

# Deploy of a flask app with postgres in aws lightsail
This project is a requirement of Udacity fullstack nanodegree. 

# Step by Step Guide

## Server Basic Information
Public IP: 3.93.33.208

SSH port: 2200

App URL: http://3.93.33.208.xip.io


## Create an aws lightsail instance
- Enter https://lightsail.aws.amazon.com/ls/webapp/home/instances
- Choose: [Create Instance] -> [Linux] -> [Ubuntu 16.04] -> [Create Instance]
- Wait "Pending" status change

## Create a key pair (in your local machine / CMD [Your machine])
# Key for enter linux server (aws)
I'm using gitbash in a windows machine. In gitbash/cmd prompt:
$ ssh-keygen

Choose file to save the key: /c/Users/<YOUR_WINDOWS_USERNAME>/.ssh/lbo-key

### port 2200 for acess in your machine
# Instances -> [click in your Instances]- >> networking 
https://lightsail.aws.amazon.com/ls/webapp/{your-local}/instances/{your-lightsail-name/networking
- In Networking (aws panel) set another rule in firewall:
-- Application = Custom
-- Protocol = TCP
-- Port = 2200

### Create a new user (in aws console)
Thanks to CharInt: 
- Login in aws ssh console and type the following:
```sh
$ sudo adduser grader
$ sudo touch /etc/sudoers.d/grader
$ sudo nano /etc/sudoers.d/grader
```
- Paste in grader file the following:
*grader ALL=(ALL:ALL) ALL*
- CTRL+X (exit nano), yes (save), enter (name file).
User grader have sudo privilegies now! Let's authorize grader to do remote login. In aws ssh:
```sh
$ su - grader
$ mkdir .ssh
$ sudo touch ~/.ssh/authorized_keys
$ sudo nano .ssh/authorized_keys
```
- Copy the public key generated on your local machine to this file c/Users/<YOUR_WINDOWS_USERNAME>/.ssh/lbo-key.pub
- CTRL+X (exit nano), yes (save), enter (name file).
- Again, in aws console:
```sh
$ sudo chown -R  grader.grader /home/grader/.ssh
$ sudo chmod  700 /home/grader/.ssh
$ sudo chmod 644 /home/grader/.ssh/authorized_keys
$ ls -als .ssh/
```

```
Now, let's restart ssh and change its port to 2200:
```sh
$ sudo service ssh restart
$ sudo nano /etc/ssh/sshd_config
```
- Change *Port* to *2200*
- CTRL+X (exit nano), yes (save), enter (name file).
```sh
$ sudo service sshd restart
$ sudo service ssh restart
```
 You can login with grader from your windows machine using gitbash now:
 ```sh
$ ssh grader@3.80.9.86 -p 2200 -i ~/.ssh/grader-key-lbo
```
Here 3.80.9.86 must be change to your Public IP. We change the usual ssh port to 2200.

## Disable root login
Logged as grader:
 ```sh
$ sudo vim /etc/ssh/sshd_config
```
- Change *PermitRootLogin* to *no*
- CTRL+X (exit nano), yes (save), enter (name file).

## UPDATE and UPGRADE
Logged as grader:
 ```sh
$ sudo apt-get update
$ sudo apt-get upgrade
configure grub-pc -> keep the local version currently installed
```

## Configure FIREWALL
Logged as grader:
 ```sh
$ sudo ufw allow 2200/tcp
$ sudo ufw allow 80/tcp
$ sudo ufw allow 123/udp
$ sudo ufw enable
```

## Configure UTC TIME_ZONE
Logged as grader:
 ```sh
$ sudo dpkg-reconfigure tzdata
Current default time zone: 'America/Sao_Paulo'
```

# Deploy catalog application on server

Logged as grader:

### Setup Apache to serve a Python mod_wsgi application
```
$ sudo apt-get install apache2
#to install appache

$ sudo apt-get install python-setuptools libapache2-mod-wsgi
#to install Install mod_wsgi

$ sudo service apache2 restart
#to restart apache
```

### Setup PostgreSQL
```
$ sudo apt-get install postgresql
#to install postgresql

$ sudo nano /etc/postgresql/9.5/main/pg_hba.conf
#to check if no remote connections are allowed

$ sudo su - postgres
#to login as postgres
```

- as postgree get into postgreSQL shell with ```psql``` in there do:

1. Create a new database named (in my case **brandsstore**) and a user named **catalog**
```
postgres=# CREATE DATABASE brandsstore;
```

2. Create a new user named catalog
```
postgres=# CREATE USER catalog;
```

3. Set a password for user catalog
```
postgres=# ALTER ROLE catalog WITH PASSWORD 'passwd123';
```
_I've also paste the passwd123 on the "Notes to Reviewer" field._

4. Give user "catalog" permission to "catalog" application database
```
postgres=# GRANT ALL PRIVILEGES ON DATABASE brandsstore TO catalog;
```

5. Quit postgreSQL
```
postgres=# \q
```

Exit from user "postgres"
```
exit
```

### Install git to clone Catalog app project.

```shell
$ sudo apt-get install git

$ cd /var/www

$ sudo mkdir FlaskApp

$ cd FlaskApp/

$ sudo git init
# Initiate an empty git repository in the current folder

$ sudo git remote add origin https://github.com/lboortigoza/catalog_python3_udacity-proj-4
# Add my project as a remote repository

$ sudo git remote -v
# Check if the remote repository was added successfully

$ sudo git pull origin master
# Pull the remote repository

$ ls
# Check if the files were downloaded successfully

```

### Configure the Catalog app

1. Rename ```project-1.py``` to```__init__.py```
``` shell
$ sudo mv project-1.py __init__.py
```
2. Edit database_setup.py, lotsofmenus.py, and the now renamed __init.py__, and change all occurrences of 'sqlite:///brandsstore.db' to 'postgresql://catalog:passwd123@localhost/brandsstore', editing the files with: sudo nano 'FILE-NAME'
sudo nano __init__.py
```python
sudo nano __init__.py
# engine = create_engine('sqlite:///brandsstore.db')
#to
create_engine('postgresql://catalog:passwd123@localhost/brandsstore')
```
at

```shell
$ sudo nano __init__.py
$ sudo nano db_config.py
```
3. Install pip

```shell
$ sudo apt-get install python-pip
$ sudo python -m pip install --upgrade pip
$ sudo pip install flask
$ sudo pip install SQLAlchemy
$ sudo apt install libpq-dev python-dev
$ sudo pip install psycopg2
$ sudo pip install httplib2
$ sudo pip install requests
$ sudo pip install --upgrade oauth2client 
$ sudo pip install -r requirements.txt
```
- Then, use it to install all dependencies listed on requirements.txt
```
$ sudo pip install -r requirements.txt

```

5. Create database schema
```
$ sudo python database_setup.py
```

6. To fill the database
```
$ sudo python lotsofmenus.py
```
```
$ sudo nano __init__.py
Remove threaded=False in last line
```

6. Configure Apache and Enable a New Virtual Host
Create FlaskApp.conf to edit:
```
$ sudo nano /etc/apache2/sites-available/FlaskApp.conf
```
Add the following lines of code to the file to configure the virtual host.
```
<VirtualHost *:80>
        ServerName 3.93.33.208
        ServerAdmin leonardo.b.ortigoza@gmail.com
        ServerAlias 3.93.33.208.xip.io
        WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
        <Directory /var/www/FlaskApp/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/FlaskApp/static
        <Directory /var/www/FlaskApp/static/>
                Order allow,deny
                Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```

Enable the virtual host with the following command:
```
$ sudo a2ensite FlaskApp
```

Activate the new configuration
```
$ sudo service apache2 restart
$ sudo service apache2 reload
```

7. Create and config the *.wsgi file

To create AND edit the desired file:
```
$ sudo nano /var/www/FlaskApp/flaskapp.wsgi
```

add the following code to flaskapp.wsgi
```
#!/usr/bin/python
import sys
import logging

logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp")

from __init__ import app as application
application.secret_key = 'super_secret_key'_ import app as application
```

# Logging
If something breaks or go wrong, you can access a log for this application with the following command:

```
sudo tail -f /var/log/apache2/error.log
```
