# Linux Server Configuration

This project is about deploying this [flask application](https://github.com/georgeerol/ItemCatalog) on to AWS Light Sail. It provides the steps on creating an instance and deploy a flask application.

* AWS IP Address: http://18.215.148.81/
* AWS Port: 2200

## Get Started on Lighsail
Go to Amazon Lightsail to setup a server instance.
## Setup the Server
1. Login
2. Create an instance
3. Choose Ubuntu(OS only) as the instance image
4. Choose an instance plan
5. Give the instance a hostname

## Sign-in via ssh
From the LightSail account page download the default lightsail pem under
SSH Key pair management. After downloading the pem file  change the permission of the lightsail private key with
```sh
chmod 600 LightsailDefaultPrivateKey.pem
```
The command to login from your terminal is:
```sh
ssh -i LightsailDefaultPrivateKey user@PublicIP
```

## Update the Server
Update all installed packages with these commands:
```sh
sudo apt-get update
sudo apt-get upgrade
```
Set auto upgrade to low unattend upgrades via this command:
```sh
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

## Create a new user
Create the grader or a new user with this command:
```sh
sudo adduser grader
```
After creating the user run this command to give her/him sudo privileges:
```sh
sudo visudo
```
Once in the visudo file, under section user privilege specification  under root add `grader  ALL=(ALL:ALL) ALL`
It should look like this:
```sh
root    ALL=(ALL:ALL) ALL
grader  ALL=(ALL:ALL) ALL
```
To logins as the grader,use this command:
```
sudo login grader
```
## Configure SSH to non-default port
From AWS LightSail Firewall page, add a custom application port 2200.
 From the ssh terminal go into the sshd config file via this command:
 ```sh
 $ sudo nano /etc/ssh/ssd_config
 ```
 From the sshd config file change, commment out Port 22 and set the information
 below:
 ```txt
# What ports, IPs and protocols we listen for
# Port 22
Port 2200

# Authentication:
LoginGraceTime 120
#PermitRootLogin prohibit-password
PermitRootLogin no
StrictModes yes

# Change to no to disable tunnelled clear text passwords
PasswordAuthentication No

 ```

After making these changes restart ssh with this command:
```sh
$ sudo user@PublicIP -p 2200
```
You will only be able to login as the grader from your local computer after
following the Creating the ssh keys.

## Creating the ssh keys
On your local computer terminal run ssh-keygen:
```
Local machine:~$ ssh-keygen
```
After running the above command, a private and public key will be created base on the name provided.
Copy the the public key information by displaying on your terminal via this
 this command:
```sh
$ cat thePublicKey
```
SSH back to the default ubuntu user account and log in as the grader.
```
$ sudo login grader
```
As the grader create the `.ssh` folder on the home directory and cd into it
```sh
$ mdkir .ssh
$ cd .ssh
```
The directory path should be `~/home/grader/.ssh`. From the .ssh folder
create a public key file and paste the public key information.
```sh
$ sudo nano publicKeyFileName
```
After this step, we can ssh as the grader by using the private key.
```sh
$ ssh -i privateKey user@PublicIP -p 2200
```

# Configure the instance timezone
Change the server timezone to UTC by running this command
```sh
$ sudo dpkg-reconfigure tzdata
```
Choose `none` of the above and choose `UTC`

# Configuring The Firewall
Firewall rules need to be set using UFW. To check if ufw is active do
do this command:
```sh
$ sudo ufw status
```
Set these UFW rules to only allow incoming connections for SSH(port 2200),
HTTP(port 80) and NTP (port 123):
```sh
$ sudo ufw allow 2200/tcp
$ sudo ufw allow 80/tcp
$ sudo ufw allow 123/tcp
$ sudo ufw enable
  ```

# Install Apache
To install apache on the server, do:
```sh
$ sudo apt-get install apache2
```
If it's install properly a welcome page will show up when  the  AWS Light
Sail public ip address is type on a web browser.

#Install WSGI and Python
To install WSGI and Python do:
```sh
$ sudo apt-get install libapache2-mod-wsgi python-dev
```
To enable WSGI do:
```sh
$ sudo a2enmod wsgi
```
Create the WSGI file `catalog.wsgi`

```sh
$ sudo nano /var/www/catalog/catalog/catalog.wsgi
```
with the information below:
```txt

#!/usr/bin/python

import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/catalog")

from application import app as application
application.secret_key = 'your secret key'
```
Application is the name of your flask server file.
Create and edit the Virtual host file:
```sh
$ sudo nano /etc/apache2/sites-available/catalog.conf
```
Edit the file with this information
```txt
<VirtualHost *:80>
      ServerName YOU_IP_ADDRESS
      ServerAdmin YOUR_EMAIL_ADDRESS
      WSGIScriptAlias / /var/www/catalog/catalog/catalog.wsgi
      <Directory /var/www/catalog/catalog/>
          Order allow,deny
          Allow from all
      </Directory>
      Alias /static /var/www/catalog/catalog/static
      <Directory /var/www/catalog/catalog/static/>
          Order allow,deny
          Allow from all
      </Directory>
      ErrorLog ${APACHE_LOG_DIR}/error.log
      LogLevel warn
      CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
 ```
 After creating the `catalog.conf` and the `catalog.wsgi` files, restart
 the apache server so the WSGI can load:
 ```sh
 $ sudo service apache2 restart
 ```

# Install Git and Clone your project
Install Git with this command:
```sh
$ sudo apt-get install git

```
Clone your repository into the apache directory, `cd /var/www/catalog`
```sh
$ sudo git clone https://github.com/georgeerol/ItemCatalog.git  catalog
```
Add the catalog.wsgi inside your git project

# Install Flask and other libraries
To install flask do these commands:
```sh
$ sudo apt-get install python-pip python-flask python-sqlalchemy python-psycopg2
$ sudo pip install oauth2client requests httplib2
```

# Install PostgreSQL

## Install PostgreSQL via this command:
```sh
$ sudo apt-get install postgresql
```

## Create the database
To create the database type in from the terminam:
```sh
sudo su - postgres
```
Type `psql` as postgres user:
```sh
psql
```
Then do the following commands:
```sh
postgres=# CREATE USER catalog WITH PASSWORD 'catalog';
postgres=# ALTER USER catalog CREATEDB;
postgres=# CREATE DATABASE catalog WITH OWNER catalog;
```
Connect to the new create database:
```sh
\c catalog
```
Then do the following command:
```sh
catalog=# REVOKE ALL ON SCHEMA public FROM public;
catalog=# GRANT ALL ON SCHEMA public TO catalog;
```
Exit postgres:
```sh
postgres=# \q
postgres@PublicIP~$ exit
```
At last update the engine from your github `database_setup`  and any file in your project with this `engine`
```python
engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
```
At last run the `database_setup`:
```sh
sudo python database_setup.py
```

# Change Oauth Client Login
Update the path of the CLIENT_ID with the line below:
```python
CLIENT_ID = json.loads(open('/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']
```
