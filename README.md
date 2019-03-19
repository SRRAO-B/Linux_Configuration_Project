# Configuring Amazon Lightsail Ubuntu Instance to run a Flask Application with Database as PostgreSQL

The project involves taking a baseline installation of a Linux server and prepare it to host web applications. This includes securing it against a number of attack vectors, installing and configuring a database server and deploying an existing web application. The objective is to configure a new bare-bones Linux server to securely host web applications.

## Getting Started (Setps 1 and 2)

A new linux distribution server instance (Ubuntu) is created on Amazon Lightsail [https://aws.amazon.com/lightsail]. The newly created server is accessed through SSH browser console on AWS site and through SSH using a Mac Terminal.

### Basic Info

IP Address: 52.23.190.192

PORT: 2200

SSH Public Key: Downloaded public key from Amazon AWS site was used for SSH connection.

## Secure Your Server
### Step 3

Update and upgrade all currently installed packages by running the command:

```
$ sudo apt-get update
```
```
$ sudo apt-get udgrade
```

### Step 4

SSH port changed from 22 to 2200 in the SSH config file.
```linux
$ sudo nano /etc/ssh/sshd_config
```
Change listening ports "Port 22" to "Port 2200"


### Step 5

Configure the UFW (Uncomplicated FireWall)

Verify that firewall is inactive:

```
$ sudo ufw status
```
Block all incoming and allow all outgoing connections:

```
$ sudo ufw default deny incoming
$ sudo ufw allow outgoing
```

Allow HTTP traffic on Port 80 (www), NTP on port 123 and SSH on port 2200

```
$ sudo ufw allow 2200/tcp
$ sudo ufw allow www
$ sudo ufw allow ntp
```

Verify and Check the firewall rules

```
$ sudo ufw show added
```

Enable the Firewall and verify

```
$ sudo ufw enable
$ sudo ufw status
```

## Give grader user access
### Step 6

Create a user "grader" and grant access by editing the sudoers file.
Add ```grader ALL=(ALL:ALL) ALL``` to the file  /etc/sudoers.d/grader
```
$ sudo adduser grader
$ sudo nano /etc/sudoers.d/grader
```

### Step 8

The grader user can log into server instance using SSH Keys. Public key was downloaded from Amazon site and SSh Keys setup for grader user. Amazon lightsail has by default user "ubuntu" setup. Same SSH keypair is used for the user grader.

```
mkdir /home/grader/.ssh
chown grader:grader /home/grader/.ssh
chmod 700 /home/grader/.ssh
cp /ubuntu/.ssh/authorized_keys /home/grader/.ssh/
chown grader:grader /home/grader/.ssh/authorized_keys
chmod 644 /home/grader/.ssh/authorized_keys
```
Check the SSH connection using LightsailDefaultKey.pem

```
ssh -i LightsailDefaultKey.pem grader@52.23.190.192 -p 2200
```
Disable root login by Changing the following line in the file /etc/ssh/sshd_config:
From PermitRootLogin without-password to PermitRootLogin no.
Uncomment line PasswordAuthentication no

Do a SSH Restart  ```$ sudo service ssh restart```

## Prepare to Deploy Your Project
### Step 9

The instance timezone was already set to UTC, but to verify I ran ```$ sudo dpkg-reconfigure tzdata```

### Step 10

Install Apache and the libapache2-mod-wsgi, and the python set up tools packages and then restart the Apache service.

```
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi
$ sudo apt-get install python-setuptools
$ sudo service apache2 restart
```

### Step 11
Database Setup
Install PostgreSQL and then open the file '/etc/postgresql/9.5/main/pg_hba.conf' to ensure that remote connections weren't allowed.

```
$ sudo apt-get install postgresql
$ sudo nano /etc/postgresql/9.5/main/pg_hba.conf
```

Create a new database by switching to postgres user, create a new database and a user named catalog  with password set to catalog. "catalog" user is given permission to use catalog database.

```
$ sudo su - postgres
postgres $ psql
postgres=# CREATE DATABASE catalog;
# CREATE USER catalog;
# ALTER ROLE catalog WITH PASSWORD 'catalog'
# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
```

### Step 12

Install git  and clone the remote repository within Amazon Lightsail Ubuntu instance

 ```$ sudo apt-get install git```

## Deploy the Item Catalog Application
### Step 13

Enable mod_wsgi for apache2 Server

```
$ sudo a2enmod wsgi
```

 Navigate to /var/www directory, create a nested catalog directory within catalog directory.

```
$ cd /var/www
/var/www $ mkdir catalog
/var/www $ cd catalog
/var/www/catalog $ mkdir catalog
/var/www/catalog $ cd catalog
```

Clone GitHub Item Catalog project into the catalog directory, rename 'application.py' to '__init__.py' Change the Database connection string in __init__.py, database_setup.py and lotsofmenus.py to reflect the use of PostgreSQL Database: "create_engine('postgresql://catalog:catalog@localhost/catalog')".
```
$ git clone https://github.com/SRRAO-B/Item_Catalog_Project.git
$ sudo mv application.py __init__.py
$ sudo nano database_setup.py
$ sudo nano __init__.py
$ sudo nano lotsofmenus.py
```

Install psycopg2  and python using
```
$ sudo apt-get install python-psycopg2
$ sudo apt-get install python
```


### Step 14

Create a *.conf file '/etc/apache2/sites-available/catalog.conf'
```$ sudo nano /etc/apache2/sites-available/catalog.conf```
and insert the following lines:
```
<VirtualHost *:80>
		ServerName itemcatalogapp.com
		ServerAdmin srxxxx@gmail.com
		WSGIScriptAlias / /var/www/catalog/catalog.wsgi
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
In the above config file server name is the domain name. Domain name that can be registered and bought using domain name providers  like google domains or Namecheap.com

Navigate to '/var/www/catalog' directory create and edit the 'catalog.wsgi' file:

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'
```

Restart Apache ```$ sudo service apache2 restart```
Add the domain name to Authorized javascript origins Google Developers console for OAuth.

## Acknowledgments

* [Digital Ocean article referenced above](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [Google Sign-In for Websites](https://developers.google.com/identity/sign-in/web/server-side-flow)
* [Google Domain Forwarding](https://support.google.com/domains/answer/4522141?hl=en#)
* [Amazon Lightsail Documentation](https://aws.amazon.com/documentation/lightsail/)
