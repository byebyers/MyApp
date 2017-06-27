Steps and Instructions:

Tasks

Create an Amazon Lightsail account configured with Ubuntu
Follow the instructions provided to SSH into your server
Create a new user named grader
Give the grader the permission to sudo
Update all currently installed packages
Change the SSH port from 22 to 2200
Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80)
Configure the local timezone to UTC
Install and configure Apache to serve a Python mod_wsgi application
Install and configure PostgreSQL:
Do not allow remote connections
Create a new user named catalog that has limited permissions to your catalog application database
Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!
Launch Virtual Machine

Instructions for SSH access to the instance

Download Private Key

Move the private key file into the folder ~/.ssh (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal. mv ~/Downloads/LightsailDefaultPrivateKey.pem ~/.ssh/

Open your terminal and type in chmod 600 ~/.ssh/LightsailDefaultPrivateKey.pem

In your terminal, type in ssh -i ~/.ssh/LightsailDefaultPrivateKey.pem root@35.160.122.180

Development Environment Information

Public IP Address

35.160.122.180

Private Key ( is not provided here. )

Create a new user named grader

sudo adduser grader
vim /etc/sudoers
touch /etc/sudoers.d/grader
vim /etc/sudoers.d/grader, type in grader ALL=(ALL:ALL) ALL, save and quit
Set ssh login using keys

generate keys on local machine usingssh-keygen ; then save the private key in ~/.ssh on local machine

deploy public key on developement enviroment

On you virtual machine:

$ su - grader
$ mkdir .ssh
$ touch .ssh/authorized_keys
$ vim .ssh/authorized_keys
Copy the public key generated on your local machine to this file and save

$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys

***Check to make sure grader owns .ssh/authorized_keys

$ sudo su root
$ chmod 600 /home/grader/.ssh/authorized_keys

Sign back in as grader

$ sudo su grader

reload SSH using service ssh restart

now you can use ssh to login with the new user you created

ssh -i [privateKeyFilename] grader@35.160.122.180

Update all currently installed packages

sudo apt-get update
sudo apt-get upgrade
Change the SSH port from 22 to 2200

Use sudo vim /etc/ssh/sshd_config and then change Port 22 to Port 2200 , save & quit.
Reload SSH using sudo service ssh restart
Configure the Uncomplicated Firewall (UFW)

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80)

sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw enable
Configure the local timezone to UTC

Configure the time zone sudo dpkg-reconfigure tzdata
It is already set to UTC.
Install and configure Apache to serve a Python mod_wsgi application

Install Apache sudo apt-get install apache2
Install mod_wsgi sudo apt-get install python-setuptools libapache2-mod-wsgi
Restart Apache sudo service apache2 restart
Install and configure PostgreSQL

Install PostgreSQL sudo apt-get install postgresql

Check if no remote connections are allowed sudo vim /etc/postgresql/9.3/main/pg_hba.conf

Login as user "postgres" sudo su - postgres

Get into postgreSQL shell psql

Create a new database named catalog and create a new user named catalog in postgreSQL shell

postgres=# CREATE DATABASE lotsofbabies;
postgres=# CREATE USER catalog;
Set a password for user catalog

postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
Give user "catalog" permission to "catalog" application database

postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
Quit postgreSQL postgres=# \q

Exit from user "postgres"

exit
Install git, clone and setup your Catalog App project.

Install Git using sudo apt-get install git
Use cd /var/www to move to the /var/www/html directory
Create the application directory sudo mkdir MyApp
Move inside this directory using cd MyApp
Clone the Catalog App to the virtual machine git clone https://github.com/byebyers/MyApp.git
Rename the project's name sudo mv ./catalog ./MyApp
Move to the inner MyApp directory using cd MyApp
Rename project.py to __init__.py using sudo mv project.py __init__.py
Edit band_database_setup.py, lotsofbabies.py and change engine = create_engine('sqlite:///lotsofbabies.db') to engine = create_engine('postgresql://catalog:password@35.160.122.180/lotsofbabies')
Install pip sudo apt-get install python-pip
Use pip to install dependencies sudo pip install -r requirements.txt
Install psycopg2 sudo apt-get -qqy install postgresql python-psycopg2
Create database schema sudo python band_database_setup.py
Configure and Enable a New Virtual Host

Create MyApp.conf to edit: sudo nano /etc/apache2/sites-available/000-default.conf

Add the following lines of code to the file to configure the virtual host.

<VirtualHost *:80>
    # The ServerName directive sets the request scheme, hostname and port that
    # the server uses to identify itself. This is used when creating
    # redirection URLs. In the context of virtual hosts, the ServerName
    # specifies what hostname must appear in the request's Host: header to
    # match this virtual host. For the default virtual host (this file) this
    # value is not decisive as it is used as a last resort host regardless.
    # However, you must set it for any further virtual host explicitly.
    #ServerName www.example.com

    ServerAdmin webmaster@localhost

    # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
    # error, crit, alert, emerg.
    # It is also possible to configure the loglevel for particular
    # modules, e.g.
    #LogLevel info ssl:warn

    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    # For most configuration files from conf-available/, which are
    # enabled or disabled at a global level, it is possible to
    # include a line for only one particular virtual host. For example the
    # following line enables the CGI configuration for this host only
    # after it has been globally disabled with "a2disconf".
    #Include conf-available/serve-cgi-bin.conf
    WSGIScriptAlias / /var/www/html/myapp.wsgi
    <Directory /var/www/html>
            Order allow,deny
            Allow from all
    </Directory>
    Alias /static /var/www/html/MyApp/static
    <Directory /var/www/html/MyApp/static/>
            Order allow,deny
            Allow from all
    </Directory>
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet

Enable the virtual host with the following command: sudo a2ensite MyApp

Create the .wsgi File

Create the .wsgi File under /var/www/html/MyApp:

cd /var/www/MyApp
sudo nano myapp.wsgi
Add the following lines of code to the myapp.wsgi file:

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/html/")

from MyApp import app as application
application.secret_key = 'Add your secret key'
Restart Apache

Restart Apache sudo service apache2 restart
References:

https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps