# Linux Server Configuration
### For Full Stack Web Developer Nano-degree program  
<a href="https://www.udacity.com/">
  <img src="https://s3-us-west-1.amazonaws.com/udacity-content/rebrand/svg/logo.min.svg" width="100" alt="Udacity logo">
</a>

This document provides summary steps on securing a server and deploying a Flask based application using AWS Lightsail services

The deployed Application can be found on github, [Items Catalog](https://github.com/andreony/Items-Catalog.git)  and currently served at http://ec2-34-234-175-61.compute-1.amazonaws.com/

## Table of Content

- [Table of Content](#table-of-content)
- [Server Side Config ](#server-side-config)
- [Deployment Packages](#deployment-packages)
- [Postgresql Configuration](#postgresql-configuration)
- [Apache and Mod WSGI](#apache-and-mod-wsgi)
- [App Tree](#app-tree)


## Server Side Config

**Access**
The server is accessible via the public ip: **34.234.175.61** with SSH listening on port 2200 

**SSH Config**
- created new user `grader`
- granted sudo privileges to user `grader`
- configured **ssh** to listen on port 2200, disabled root login and disabled password auth:
	- `sudo vim vi /etc/ssh/sshd_config` :
`Port 2200`
`AddressFamily any`
`ListenAddress 0.0.0.0`
`ListenAddress ::`
`PermitRootLogin no`
`PasswordAuthentication no`
- generated ssh key and pasted the public key into the authorized keys file 
	- `/home/grader/.ssh/authorized_keys`
- limited permissions on .ssh folder and authorized keys file:
	-  `chmod 700 .ssh`
	- `chmod 644 .ssh/authorized_keys`

**Firewall Config**
- changed ufw default behavior:
	- `sudo ufw default deny incoming`
	- `sudo ufw default allow outgoing`
-  added rules for ports 2200/tcp, HTTP(80) and NTP(123 )
	- `sudo ufw allow 2200/tcp`
	- `sudo ufw allow http`
	- `sudo ufw allow ntp`

**Packages Update/Upgrade**
- updated all packages to the newest, available versions:
	- `sudo apt-get update`
	- `sudo apt-get upgrade`

## Deployment Packages

**System Related**
- installed the following packages: postgresql, libpq-dev, libapache2-mod-wsgi, python-dev, virtualenv
	- `sudo apt-get install libapache2-mod-wsgi python-dev postgresql libpq-dev virtualenv`
	- enabled apache mode wsgi:
		- `sudo a2enmod wsgi`

**Application Related**

The app repository was cloned from github to the following path:
`/var/www/Items_Catalog/items_catalog`
- created the virtual environment with python2.7 -> `/var/www/Items_Catalog/venv`
- installed requirements.txt after activating the virtual env
- also installed psycopg2 

## Postgresql Configuration
- connected to the database using postgres user and connected to the database 
	- `sudo su - postgres` 
	- `psql`

- created a new database and role for the application, as follows :
	-  `create database items_catalog;`
	- `create role udacity_grader with login password <password>` -> the actual password is available in the wsgi.py file
	- `grant all privileges on database items_catalog to udacity_grader;`
	- `alter role udacity_grader set client_encoding to 'utf8';`
	- `alter role udacity_grader set default_transaction_isolation to 'read committed';`
	- `alter role udacity_grader set timezone to 'UTC';`

## Apache and Mod WSGI
- created the wsgi.py file, in the project path root, and the vhost config on port 80 -> `/etc/apache2/sites-available/items_catalog.conf`, according with the mode wsgi [Flask Documentation](https://www.bogotobogo.com/python/Flask/Python_Flask_HelloWorld_App_with_Apache_WSGI_Ubuntu14.php)  
- added db name and password as environmental variables to the wsgi.py file and set file permission to 750
- changed the owner & group of all files under the Flask App (`/var/www/Items_Catalog`) to `www-data`
	- `sudo chown -R www-data:www-data /var/www/Items_Catalog`
- enabled  the items_catalog.conf config file:
	- `sudo a2ensite items_catalog`
- reloaded apache2 service:
	- `sudo service apache2 reload`
- finally, changed the session engine of the app from sqlite3 to postgresql

The application is successfully deployed!

## App Tree

    Items_Catalog
    └── items_catalog
        ├── README.md
        ├── __init__.py
        ├── __init__.pyc
        ├── application.py
        ├── application.pyc
        ├── catalog_items2.db
        ├── client_secrets.json
        ├── fb_client_secrets.json
        ├── models.py
        ├── models.pyc
        ├── requirements.txt
        ├── static
        ├── templates
        ├── venv
        └── wsgi.py

