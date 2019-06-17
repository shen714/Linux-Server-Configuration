# Linux-Server-Configuration 


## Summary of the server 
- Public IP : [52.36.43.221](104.131.87.6)
- Server OS : Ubuntu 16.04 x64
- SSH Port: 2200
- Server Provider:  Amazon Lightsail
- Web Apllication: [Item-Catalog](https://github.com/shen714/Item-Catalog)
- URL: [52.36.43.221](104.131.87.6)


## Summary of software installed
- apache2
- libapache2-mod-wsgi
- postgresql
- git
- flask
- sqlalchemy
- google-auth
- python
- sqlite3

## Setting Up the Server

### 1. Create a new server
* Start a new Ubuntu Linux server instance on Amazon Lightsail.

### 2. Secure the server
* Log into the instance with SSH as ubuntu user.
* Update all currently installed packages.
```sh
~ $  sudo apt-get update 
~ $ sudo apt-get upgrade 
```

* Configure the Uncomplicated Firewall.
```sh
~ $ sudo ufw status 
~ $ sudo ufw default deny incoming 
~ $ sudo ufw default allow outgoing 
~ $ sudo ufw allow 2200/tcp # this has to be done before change the SSH port to prevent locking yourself out of the server.
~ $ sudo ufw allow 80/tcp 
~ $ sudo ufw allow 123/udp 
~ $ sudo ufw enable 
~ $ sudo ufw status 
```

* Change the SSH port from 22 to 2200. 
```sh
Add the Custom TCP 2200 on the app GUI 
~ $ sudo nano /etc/ssh/sshd_config
Change Port 22 to Port 2200
~ $ sudo service ssh restart
```

### 3.Create a grader user and give it the access
* Create a new user account named grader.
```sh
~ $ sudo adduser grader
```
* Give grader the permission to sudo.
```sh
~ $ sudo cat /etc.sudoers
add 'grader ALL=(ALL:ALL) NOPASSWD:ALL' after 
'root    ALL=(ALL:ALL) ALL'
```
*  Create an SSH key pair for grader using the ssh-keygen tool.'
on your LOCAL machine!!
```sh
~ $ ssh-keygen
enter your passphrase or leave it empty
enter the file path as ~/.ssh/grader
~ $ cat ~/.ssh/grader.pub
copy the public key

on the remote machine
~ $ sudo mkdir /home/grader/.ssh
~ $ sudo nano /home/grader/.ssh/authorized_keys
paste the public key 
~ $ sudo chmod 700 /home/grader/.ssh
~ $ sudo chmod 600 /home/grader/.ssh/authorized_keys
~ $ sudo chown /home/grader/.ssh/
~ $ sudo chown /home/grader/.ssh/authorized_keys
```

### 4.Install Apache and mod_wsgi
* Configure the local timezone to UTC.
```sh
~ $ sudo timedatectl set-timezone UTC
```
* Install and configure Apache to serve a Python mod_wsgi application.
```sh
~ $ sudo apt-get install apache2
~ $ sudo apt-get install libapache2-mod-wsgi 
~ $ sudo service apache2 restart
```
### 5.Install and configure PostgreSQL
```sh
~ $ sudo apt-get install postgresql
~ $ sudo su - postgres
~ $ psql
~ $ postgres=> CREATE DATABASE itemcatalog;
~ $ postgres=> CREATE USER itemcatalog;
~ $ postgres=> ALTER ROLE itemcatalog WITH PASSWORD 'password';
~ $ postgres=> GRANT ALL PRIVILEGES ON DATABASE itemcatalog TO itemcatalog;
~ $ postgres=> \q
```

### 6. Deploy the Item Catalog project.
* clone the Item Catalog poroject.
~ $ sudo mkdir /var/www/ItemCatalog;
~ $ cd /var/www/ItemCatalog;
~ $ git clone https://github.com/shen714/Item-Catalog.git;

* convert sqlite to postgresql
In database_setup.py and application.py, change engine = create_engine('sqlite:///itemcatalog.db') to engine = create_engine('postgresql://itemcatalog:password@localhost/itemcatalog')

* install all the dependencies
~ $ sudo apt-get install python-pip
~ $ sudo pip install -r requirements.txt

* set a new database
~ $ sudo python database_setup.py

* configure and Enable a New Virtual Host
```sh
~ $ sudo mv ItemCatalog/application.py ItemCatalog/__init__.py;
sudo nano /etc/apache2/sites-available/FlaskApp.conf
add :
<VirtualHost *:80>
	ServerName 52.24.125.52
	ServerAdmin qiaowei8993@gmail.com
	WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
	<Directory /var/www/FlaskApp/FlaskApp/>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/FlaskApp/FlaskApp/static
	<Directory /var/www/FlaskApp/FlaskApp/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
~ $ sudo service apache2 restart
~ $ sudo a2ensite FlaskApp
```
* create .wsgi file
~ $ sudo nano /var/www/ItemCatalog/itemcatalog.wsgi
add:
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/ItemCatalog/")

from ItemCatalog import app as application
application.secret_key = 'Add your secret key'
~ $ sudo service apache2 restart
```

## Reference
https://github.com/kongling893/Linux-Server-Configuration-UDACITY