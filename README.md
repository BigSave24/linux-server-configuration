Linux Server Configuration Project
----------------------------------
Ubuntu Linux Web Server


Purpose
--------
Configure and secure a baseline Linux server in preparation for hosting a web application.

## Requirements
----------------
The following software is needed to run this project:

- [Python 2](https://www.python.org/download/releases/2.7.2/)
- [Flask](http://flask.pocoo.org/)
- [PostgreSQL](https://www.postgresql.org/)
- [SQLAlchemy](https://www.sqlalchemy.org/)
- [OAuth 2.0](https://oauth.net/2/)
- [Git](https://git-scm.com/)
- [Apache](https://httpd.apache.org/)
- [AWS Lightsail VPS](https://aws.amazon.com/lightsail/)
- [WSGI is the Web Server Gateway Interface (WSGI)](https://wsgi.readthedocs.io/en/latest/index.html)


## Linux Configuration Details
------------------------------
### Setup AWS Lightsail Instance
At https://aws.amazon.com/lightsail/, follow the instructions for setting up an instance plan\
Use the following settings:
- Platform `Linux/Unix`
- Blueprint `OS Only > Ubuntu 16.04 LTS`  
- Instance Plan `Lowest plan available`
- Instance Name `Create unique name`

### Connect to and Update New Server
Connect using the AWS Lightsail *Connect with ssh* client\
Update the server packages and time zone(to UTC) \
##### COMMAND: `sudo apt-get update && sudo apt-get upgrade`
##### COMMAND: `sudo timedatectl set-timezone UTC`

### Create New User *grader*
Create a new user named *grader*\
##### COMMAND: `sudo adduser grader`

Add *grader* to sudoers configuration\
##### COMMAND: `sudo nano /etc/sudoers.d/90-cloud-init-users`

Create *.ssh/authorized_keys* file for *grader*\
##### COMMAND: `sudo mkdir /home/grader/.ssh`
##### COMMAND: `sudo touch /home/grader/.ssh/authorized_keys`

Change *.ssh/authorized_keys* file permissions and ownership\
##### COMMAND: `sudo chown -R grader:grader /home/grader/.ssh'`
##### COMMAND: ``sudo chmod 700 /home/grader/.ssh``
##### COMMAND: `sudo chmod 644 /home/grader/.ssh/authorized_keys`

### Setup SSH Key
On your local device, open a terminal window\
Linux\Mac - Use the default terminal application.\
Windows - You can use a terminal application such as
[Git Bash](https://gitforwindows.org/) or [Ubuntu](https://www.howtogeek.com/265900/everything-you-can-do-with-windows-10s-new-bash-shell/).\
Run the following command:
##### COMMAND: `ssh-keygen -t rsa`
##### COMMAND: `Enter file in which to save the key (/home/demo/.ssh/id_rsa): linuxServProject`
##### COMMAND: `Enter passphrase (empty for no passphrase): *create passphrase [optional]* `
Copy the public key (linuxServProject.pub file) to the server\
**_\*NOTE\*_** AWS Lightsail *Connect with ssh* client has a built-in clipboard to copy\paste
##### COMMAND: `sudo nano /home/grader/.ssh/authorized_keys`

### Setup SSH and Web Server Network Configuration
In the AWS Lightsail > Networking console, add a connection:
- Application: *Custom* Protocol: *TCP* Port Range: *2200*\

Add port 2200 to ssh configuration file\
##### COMMAND: `sudo nano /etc/ssh/sshd_config`

Configure the Uncomplicated Firewall (UFW) by first checking the status\
##### COMMAND: `sudo ufw status`

Add required firewall rules for http(80), ssh(2200), and ntp(123) *ONLY*\
##### COMMAND: `sudo ufw default allow outgoing`
##### COMMAND: `sudo ufw default deny incoming`

##### COMMAND: `sudo ufw deny 22`

##### COMMAND: `sudo ufw allow 80/tcp`
##### COMMAND: `sudo ufw allow 2200/tcp`
##### COMMAND: `sudo ufw allow 123/tcp`

##### COMMAND: `sudo ufw enable`

Verify ufw is running with correct configuration\
##### COMMAND: `sudo ufw status`

### Connect Using New SSH Configuration
From your local device terminal, connect with the *server public IP* using the *grader* account\
##### COMMAND: `ssh grader@34.205.122.25 -p 2200 -i linuxServProject.ppk`

Remove port 22 from ssh configuration file\
##### COMMAND: `sudo nano /etc/ssh/sshd_config`

Disable *root* remote login and password authentication\
##### COMMAND: `sudo nano /etc/ssh/sshd_config`
##### COMMAND: `PermitRootLogin without-password`
##### COMMAND: `PasswordAuthentication no`

Restart ssh server
##### COMMAND: `sudo service ssh restart`

### Install Required Applications
Install Apache web server
##### COMMAND: `sudo apt-get install apache2`

Install Python
##### COMMAND: `sudo apt-get install python`
##### COMMAND: `sudo apt-get install python-pip`
##### COMMAND: `sudo apt-get install libapache2-mod-wsgi`
##### COMMAND: `pip install Flask`
##### COMMAND: `pip install httplib2`
##### COMMAND: `pip install SQLAlchemy`
##### COMMAND: `pip install oauth2client`
##### COMMAND: `pip install requests`
##### COMMAND: `pip install bleach`
##### COMMAND: `pip install psycopg2`

Install PostgreSQL
##### COMMAND: `sudo apt-get install postgreSQL`

Install Git
##### COMMAND: `sudo apt-get install git`

### Clone and Setup Item Catalog Project

Navigate to *www* directory
##### COMMAND: `cd /var/www`

Create directory for project clone\
##### COMMAND: `sudo mkdir /var/www/sportsCatalog`

Update directory permissions
##### COMMAND: `sudo chown -R grader:grader sportsCatalog`

Clone item-catalog-project in *sportsCatalog* directory\
##### COMMAND: `cd sportsCatalog/`
##### COMMAND: `git clone https://github.com/BigSave24/item-catalog-project.git`

Update python web server name
- New Name: `_init_.py`
- Old Name `catalog-project.py`

### Setup PostgreSQL Database
Login to the database\
##### COMMAND: `sudo su - postgres`
##### COMMAND: `psql`

Create new database user *catalog*\
##### COMMAND: `CREATE USER catalog WITH PASSWORD 'password';`

Create new database\
##### COMMAND: `CREATE DATABASE sportscatalog;`

Assign *catalog* ownership of the *sportscatalog* database
##### COMMAND: `ALTER DATABASE sportscatalog OWNER TO catalog;`

Assign *catalog* limited permissions to the *sportscatalog* database
##### COMMAND: `ALTER USER catalog CREATEDB;`
##### COMMAND: `\c sportscatalog`
##### COMMAND: `REVOKE ALL ON SCHEMA public FROM public`
##### COMMAND: `GRANT ALL ON SCHEMA public TO catalog`
**_\*NOTE\*_** Verify PostgreSQL users with *\du or \du+* and databases with *\list*

Exit PostgreSQL Database
##### COMMAND: `\q`
##### COMMAND:  `exit`

Confirm that no remote connections are configured for PostgreSQL
##### COMMAND: `sudo nano /etc/postgresql/9.5/main/pg_hba.conf`

### Setup Python Virtual Environment
Install virtual environment
##### COMMAND: `pip install virtualenv`

Setup virtual environment name
##### COMMAND: `sudo virtualenv vapp`

Activate virtual environment
##### COMMAND: `source vapp/bin/activate `

### Setup and Apache Web Server and Flask Application Configuration
Navigate to the *sportsCatalog* directory\
##### COMMAND: `cd /var/www/sportsCatalog`

Create and configure the *.wsgi* file\
##### COMMAND: `touch sportsApp.wsgi`
##### COMMAND: `sudo nano sportsApp.wsgi`
```python
import logging
import sys

# Configure Logging
logging.basicConfig(stream=sys.stderr)

# Path of application execution
sys.path.insert(0, '/var/www/sportsCatalog/item-catalog-project/catalog')

# Import Catalog App as application
from _init_ import app as application
application.secret_key = 'super_secret_key'
```

Create and configure a *sportsApp.wsgi* configuration file for the Apache web server\
##### COMMAND: `sudo touch /etc/apache2/sites-available/sportsCatalog.conf`
##### COMMAND: `sudo nano /etc/apache2/sites-available/sportsCatalog.conf`
```
<VirtualHost *:80>
    ServerName 34.205.122.25
    ServerAdmin admin@34.205.122.25
    WSGIDaemonProcess sportsCatalog python-home=/var/www:/var/www/sportsCatalog/vapp/lib/python2.7/site-packages
    WSGIProcessGroup sportsCatalog
    WSGIScriptAlias / /var/www/sportsCatalog/sportsApp.wsgi
    <Directory /var/www/sportsCatalog/item-catalog-project/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/sportsCatalog/item-catalog-project/catalog/static
    <Directory /var/www/sportsCatalog/item-catalog-project/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```

Create and configure *.htaccess* file to prevent browser access to *.git* directory\
##### COMMAND: `sudo touch /etc/apache2/.htaccess`
##### COMMAND: `sudo nano /etc/apache2/.htaccess`
```
RewriteEngine on
RewriteRule "^(.*/)?\.git/" - [F,L]
RedirectMatch 404 /\.git
```

Navigate to the *catalog* application directory\
##### COMMAND: `cd /var/www/sportsCatalog/item-catalog-project/catalog/`

Update the following files with the new database engine\
- `_init_.py` Python web application server file
- `database_setup.py` Create *sportscatalog.db* database tables
- `sportsData.py` Data file to populate *sportscatalog.db* database
##### COMMAND: `engine = create_engine('postgresql://catalog:password@localhost/sportscatalog')`

Run *database_setup.py* to setup database tables\
##### COMMAND: `python database_setup.py`

Run *sportsData.py* to populate database\
##### COMMAND: `python sportsData.py`

Enable mod_wsgi interface\
##### COMMAND: `sudo a2enmod wsgi`

Enable the virtual host\
##### COMMAND: `sudo a2ensite sportsCatalog`

Restart the Apache web server\
##### COMMAND: `sudo service apache2 restart`

Navigate to the AWS Lightsail server's public IP\
##### http://34.205.122.25
