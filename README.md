# udacity_linux_web_server
Project Five: Linux Web Server

This is the repository for Project Five of the Udacity Fullstack Web Developer Nanodegree - Linux Web Server.

## Server Information
+ IP Address - 52.33.198.83
+ SSH Port - 2200
+ http://ec2-52-33-198-83.us-west-2.compute.amazonaws.com/

### SSH Command for grader:
+ ssh grader@52.33.198.83 -p 2200 -i ~/.ssh/udacity_key.rsa


## Software

### sudo apt-get install:
+ apache2
+ libapache2-mod-wsgi
+ postgresql
+ git
+ python-pip
+ geomview
+ build-essential
+ python-dev
+ libxml2-dev
+ libxslt-dev
+ libpq-dev
+ fail2ban
+ glances

### sudo pip install:
+ virtualenv
+ Flask
+ SQLAlchemy
+ Flask-SQLAlchemy
+ flask-seasurf
+ -U psycopg2
+ httplib2
+ oauth2client

### sudo git clone:
+ https://github.com/juicyjc/udacity_catalog.git


## Configuration

### Create users and add to sudo
+ adduser grader
+ adduser grader sudo

### Permanent sudo settings for users:
+ sudo touch /etc/sudoers.d/jeremy
+ sudo cat /etc/sudoers.d/jeremy
jeremy    ALL=(ALL:ALL) ALL

### Setting up SSH for users
+ login as user
+ mkdir .ssh
+ touch .ssh/authorized_keys
+ nano .ssh/authorized_keys
+ chmod 700 .ssh
+ chmod 644 .ssh/authorized_keys

### Configuring and Enabling UFW
+ sudo ufw status
+ sudo ufw default deny incoming
+ sudo ufw default allow outgoing
+ sudo ufw allow www
+ sudo ufw allow ntp
+ sudo ufw allow 2200
+ sudo ufw enable
+ sudo ufw status

### Update /etc/hosts line "127.0.0.1 localhost" with:
+ ip-10-20-36-96
    
### Install Git and configure
+ git config --global user.name "Jeremy Collins"
+ git config --global user.email "collins.jeremy@gmail.com"

###  Postgresql

#### Install Postgresql and start
+ sudo service postgres start

#### Create catalog user in Ubuntu

#### Create catalog user in Postgresql
+ sudo -su postgres
+ psql

####Create catalog DB
+ CREATE DATABASE catalog
+ psql catalog

### Website

#### Install Apache, Mod_wsgi and virtualenv
+ sudo a2enmod wsgi

#### Download code for website from Git to /var/www/udacity_catalog/
		
#### Create virtual enviroment for the application and activate
+ sudo virtualenv venv
+ source venv/bin/activate
			
#### Rename "application.py" to "__init__.py"

#### Update config file
+ sudo nano /etc/apache2/sites-available/udacity_catalog.conf
```
<VirtualHost *:80>
ServerName 52.33.198.83
ServerAlias ec2-52-33-198-83.us-west-2.compute.amazonaws.com
ServerAdmin collins.jeremy@gmail.com
WSGIScriptAlias / /var/www/udacity_catalog/udacity_catalog.wsgi
<Directory /var/www/udacity_catalog/udacity_catalog/>
Order allow,deny
Allow from all
</Directory>
Alias /static /var/www/udacity_catalog/udacity_catalog/static
<Directory /var/www/udacity_catalog/udacity_catalog/static/>
Order allow,deny
Allow from all
</Directory>
ErrorLog ${APACHE_LOG_DIR}/error.log
LogLevel warn
CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

#### Enable the virtual host
+ sudo a2ensite udacity_catalog
			
#### Create wsgi file and configure
+ sudo touch /var/www/udacity_catalog/udacity_catalog.wsgi
+ sudo nano /var/www/udacity_catalog/udacity_catalog.wsgi
```
\#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/udacity_catalog/")
from udacity_catalog import app as application
```

#### Restart apache
+ sudo service apache2 restart

#### Update code for Postgresql
+ Create "settings.py" and populate with:
```python
DATABASE = {
	'drivername': 'postgres',
	'host': 'localhost',
	'port': '5432',
	'username': 'catalog',
	'password': ******,
	'database': 'catalog'
}
```

#### Add the following line to all files that access the DB:
`from sqlalchemy.engine.url import URL`

#### Update create_engine call on all files that access the DB:
`engine = create_engine(URL(**settings.DATABASE))`

#### Update the __main__ section as follow:
```python
if __name__ == '__main__':
	app.debug = False
	app.run(host='0.0.0.0', port=8000)
```

#### Move app.secret_key line to right after app definition:
```python
app = Flask(__name__)
app.secret_key = ******
```

#### Update all file references to absolute paths (ie. client_secrets.json):
```python
CLIENT_ID = json.loads(
	open('/var/www/udacity_catalog/udacity_catalog/client_secrets.json', 'r').read())['web']['client_id']
```
	
#### Create the tables in the DB:
+ sudo python database_setup.py

#### Update the following field types on catalog DB:
+ item/description
from character varying(80) to text
+ user/picture
from character varying(80) to character varying(200)

#### Populate the tables with data:
+ sudo python load_data.py

#### Change ownership of the img folder to www-data
+ sudo chown -R www-data /var/www/udacity_catalog/udacity_catalog/static/img/

### Automatically update packages
+ sudo touch autoupdt
+ sudo nano autoupdt
```
\#!/bin/bash
apt-get update
apt-get upgrade -y
apt-get autoclean
```
+ sudo mv autoupdt /etc/cron.weekly
+ cd /etc/cron.weekly
+ sudo chmod 755 autoupdt
+ To test:
`run-parts --test /etc/cron.weekly`
    
### Monitor repeat unsuccessful login attempts and ban
+ sudo apt-get install fail2ban
+ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
+ sudo nano /etc/fail2ban/jail.local
+ Update ssh config:
```
[ssh]
enabled = true
banaction = ufw-ssh
port = 2200
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
```
+ sudo touch /etc/fail2ban/action.d/ufw-ssh.conf
+ sudo nano /etc/fail2ban/action.d/ufw-ssh.conf
```
[Definition]
actionstart =
actionstop =
actioncheck =
actionban = ufw insert 1 deny from <ip> to any app OpenSSH
actionunban = ufw delete deny from <ip> to any app OpenSSH
```
+ sudo service fail2ban stop
+ sudo service fail2ban start
     
### Monitor web server with Glances
+ install Glances
+ Turn on at startup -
+ sudo nano /etc/default/glances
```
\# Change to 'true' to have glances running at startup
RUN="true"
```
+ sudo service glances start


## Resources

+ How To Deploy a Flask Application on an Ubuntu VPS
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

+ Web Scraping (general SQLAlchemy help)
http://newcoder.io/scrape/
http://newcoder.io/begin/setup-your-machine/

+ Creating Postgresql Users
http://www.postgresql.org/docs/9.1/static/app-createuser.html

+ View tables in Postgresql
http://stackoverflow.com/questions/109325/postgresql-describe-table

+ Increasing the size of character varying type in postgres without data loss
http://stackoverflow.com/questions/5488428/increasing-the-size-of-character-varying-type-in-postgres-without-data-loss

+ Update packages every week
https://help.ubuntu.com/community/AutoWeeklyUpdateHowTo

+ Configure firewall to monitor for repeat unsuccessful login attempts and ban attackers
https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04
http://askubuntu.com/questions/54771/potential-ufw-and-fail2ban-conflicts

+ Glances
https://pypi.python.org/pypi/Glances
https://github.com/nicolargo/glances/blob/master/docs/glances-doc.rst
http://www.cyberciti.biz/faq/linux-install-glances-monitoring-tool/

+ Update permissions on img folder
http://stackoverflow.com/questions/9181254/enabling-write-permissions-ubuntu-server-in-var-www-image-directory
