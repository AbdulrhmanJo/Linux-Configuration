# Linux Server Configuration
## About
take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.
## IP & Hostname
IP Address: 18.184.98.41 

Host Name: ec2-18-184-98-41.eu-central-1.compute.amazonaws.com
## Step by Step Walkthrough
1- create an instance on [Amazon Lightsail](https://aws.amazon.com/lightsail/) website

2- set up the firewall on your instance [Amazon Lightsail](https://aws.amazon.com/lightsail/) website by adding TCP/123 and TCP/2200

3- Download your private key from instance.

4- move the private key to ./ssh folder.

5- secure our key type `$ chmod 600 ~/.ssh/udacity.pem` into the terminal.

6- Now in terminal log into the server by type `$ ssh -i ~/.ssh/udacity.pem ubuntu@18.184.98.41`

7- create a user called `grader`. From the command line type `$ sudo adduser grader`.

8- create a file to give the user grader superuser Sudo. type `$ sudo nano /etc/sudoers.d/grader`. then type `grader ALL=(ALL:ALL) ALL` in the file.

9- updating package list by type:
```
 $ sudo apt-get update
 $ sudo apt-get upgrade
 $ sudo apt-get dist-upgrade
 ```
10- install **Finger** with the command  `$ sudo apt-get install finger`
 
11- create an SSH Key for **grader**. From a new terminal type the command: `$ ssh-keygen -f ~/.ssh/udacity.rsa`
 
12- In the same terminal copy the public key by type the command: `$ cat ~/.ssh/udacity.rsa.pub`. Copy the key from the terminal.
 
13- Back in the server terminal locate the folder for **grader**, Run the command `$ cd /home/grader` then Create a directory called .ssh with the command `$ mkdir .ssh`, then Create a file to paste the public key int it with the command `$ touch .ssh/authorized_keys`, then `$ nano .ssh/authorized_keys` and past the key and save it.
 
14- change the permissions of the file and its folder 
 ```
 $ sudo chmod 700 /home/grader/.ssh
 $ sudo chmod 644 /home/grader/.ssh/authorized_keys 
 ```
15- Change the owner of the `.ssh` directory to **grader** by type `$ sudo chown -R grader:grader /home/grader/.ssh`.
 
16- restart the service with `$ sudo service ssh restart`, then disconnect from the server.
 
17- Now login with the grader user local terminal type `$ ssh -i ~/.ssh/udacity.rsa grader@18.184.98.41`.
 
18- enforce key authentication from the ssh configuration file by editing `$ sudo nano /etc/ssh/sshd_config`. Find **PasswordAuthentication** and change it to no, find **Port 22** and change it to **Port 2200**, change **PermitRootLogin** to no.
 
19- Restart ssh again: `$ sudo service ssh restart`, and disconnect from the server.
 
20- configure the firewall using these commands:
 ```
 $ sudo ufw allow 2200/tcp
 $ sudo ufw allow 80/tcp
 $ sudo ufw allow 123/udp
 $ sudo ufw enable
 ```
 
### Application Deployment
 
 21- installing the required software
 ```
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi python-dev
$ sudo apt-get install git
```

22- Enable mod_wsgi by type `$ sudo a2enmod wsgi`, then restart Apache by type `$ sudo service apache2 restart`.

23- create a folder for catalog application and make the **grader** the owner.
```
$ cd /var/www
$ sudo mkdir catalog
$ sudo chown -R grader:grader catalog
$ cd catalog
```

24- cloning Movie_Catalog Application repository from github by `$ git clone https://github.com/AbdulrhmanJo/Movie_catalog.git catalog`

25- Create the .wsgi file by `$ sudo nano catalog.wsgi` and type in it this:
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'
```
26- Rename `application.py` to '__init__.py', by `$ mv application.py __init__.py`

27- create virtual environment:
```
$ sudo pip install virtualenv
$ sudo virtualenv venv
$ source venv/bin/activate
$ sudo chmod -R 777 venv
``` 
28- install all packages required for Flask application
```
$ sudo apt-get install python-pip
$ sudo pip install flask
$ sudo pip install httplib2 oauth2client sqlalchemy psycopg2 #etc...
$ pip install requests
```
29- change the `client_secrets.json` in `__init__.py` to its complete path `/var/www/catalog/catalog/client_secrets.json`
30- configure and enable virtual host to run the web app
```
$ sudo nano /etc/apache2/sites-available/catalog.conf
``` 
  then write this with IP and Host Name:
  ```
<VirtualHost *:80>
    ServerName 18.184.98.41 
    ServerAlias ec2-18-184-98-41.eu-central-1.compute.amazonaws.com
    ServerAdmin admin@35.167.27.204
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
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
31- Enable to virtual host: `$ sudo a2ensite catalog.conf` then **DISABLE** the default host `$ sudo a2dissite 000-default.conf`.

32- setting up the database
```
$ sudo apt-get install libpq-dev python-dev
$ sudo apt-get install postgresql postgresql-contrib
$ sudo su - postgres -i
$ psql
```
then create a database user and password
```
postgres=# CREATE USER catalog WITH PASSWORD [password];
postgres=# ALTER USER catalog CREATEDB;
postgres=# CREATE DATABASE catalog with OWNER catalog;
postgres=# \c catalog
catalog=# REVOKE ALL ON SCHEMA public FROM public;
catalog=# GRANT ALL ON SCHEMA public TO catalog;
catalog=# \q
$ exit
```
### Authenticate login through Google
33- edit an OAuth Client ID, add http://18.184.98.41.xip.io as authorized JavaScript origins, add http://18.184.98.41.xip.io/login	
 ,and http://18.184.98.41.xip.io/gconnect as authorized redirect URI.

34- Download the JSON file, open it et copy the contents, then open client_secret.json and paste the contents into the this file.

35- Replace the client ID of the `templates/login.html` file in the project directory.

36- edit ` __init__.py`, `database_setup.py`, and `seddr.py` files to change the database engine to `postgresql://catalog:[password]@localhost/catalog`.

37- restart your apache server `$ sudo service apache2 restart`

now anyone can visit the web app through http://18.184.98.41.xip.io

### References:

 - https://github.com/boisalai/udacity-linux-server-configuration
 - https://github.com/iliketomatoes/linux_server_configuration
 - https://github.com/stueken/FSND-P5_Linux-Server-Configuration
 - https://github.com/mulligan121/Udacity-Linux-Configuration

