# Udacity_FSND_Linux_Server_Configuration

This is the fifth project for **Full Stack Web Developer Nanodegree** on Udacity, 
the purpose is to configure & host a web application **(Item Catalog)** in a linux virtual machine.

# The steps for completing the project are:
1.  Start new Ubuntu Linux server instance **in this case was use [Amazon Lighsail](https://aws.amazon.com) to create the instance**
2.  Follow the instructions provided to SSH into your server
### Instance URl and IP ```http://ec2-18-197-75-198.eu-central-1.compute.amazonaws.com```   IP-Address ```18.197.75.198```
3.  Create a new user named **grader** and give him sudo permissions
      ```linux
      sudo adduser grader
      sudo touch /etc/sudoers.d/grader
      sudo nano /etc/sudoers.d/grader
      grader ALL=(ALL) NOPASSWD:ALL
      ```
      
## Enable SSH login for user grader

- On **EC2 Dashboard** on AWS from the left side list select option **NETWORK & SECURITY > Key Pairs and create a Key Pair**
- Download the created key pair and place it in a safe place, you will have only one download so donwload & keep it safe.
- Download and install [PuTTY.exe and PuTTYgen](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
- Open PuTTYgen and load the [keypair.pem] file, select the type of key to generate **in this project was used RSA type**
  **Note:** to open the .pem extension you will have to filter by All Files(*.*)
- On virtual machine under the **grader** user create a folder named **.ssh** and then a file named authorized_keys, 
  paste in this file the earler generarted public_key with PuTTYgen and save it.
- Don't forget to set .ssh & authorized_keys permissions to chmod 700 .ssh chmod & 644 .ssh/authorized_keys respectively
- Update installed packages in you instance with command: ```sudo apt-get update``` & ```sudo apt-get upgrade```

- Change **SSH** port to 2200
  ```linux
  sudo nano /etc/ssh/sshd_config, on line port 22, change port 22 to 2200
  ```
- Disable **SSH** login root: ```sudo nano /etc/ssh/sshd_config```, on line ```PermitRootLogin without-password```, change to ```PermitRootLogin no```
  and ```PasswordAuthentication no```
- Restart ssh service: ```sudo service ssh restart```
- On your local machine bash navigate to the folder where you downloaded the **.pem** file and execute the following command:
  ```ssh -i linux-private-key.pem grader@ec2-18-197-75-198.eu-central-1.compute.amazonaws.com -p 2200```

## Configure UFW **(Uncomplicated Firewall)**
```linux
sudo ufw allow ssh
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable
```

## Configure local timezone to UTC

- ```dpkg-reconfigure tzdata```
- In the first list select **none of the above**, in the second list select UTC

## Install apache
```linux
sudo apt-get install apache2
Install mod_wsgi: sudo apt-get install python-setuptools libapache2-mod-wsgi
Restart Apache: sudo service apache2 restart
```

## Install and configure PostgreSQL

- Install PostgreSQL: ```sudo apt-get install postgresql```
- Edit this file to check if no remote connections are allowed: ```sudo nano /etc/postgresql/9.5/main/pg_hba.conf```
- Login as user "postgres" ```sudo su - postgres```
- Get into postgreSQL shell psql

## Create database and user catalog
```sql
postgres=# create database catalog;
postgres=# create user catalog;
postgres=# create role catalog with password '####';
postgres=# alter role catalog login;
postgres=# grant all privileges on database catalog to catalog;
```
- To login into database from user **grader@** when prompted to password enter 'catalog'
- ```psql -U catalog -d catalog -h 127.0.0.1 -W```

## Install git
- ```sudo apt-get install git```

## Deploy the application
```linux
cd /var/www
sudo mkdir app
cd app
```
- **Clone Item-Catalog-Application:** ```sudo git clone https://github.com/irzelindo/Item-Catalog-Application.git```
- Give user grader the privileges to the files with chown and chgrp commands
- **Install required dependencies:** ```sudo pip install -r requirements.txt```

## Configure and enable a new virtual host
- Create a configuration file: sudo nano /etc/apache2/sites-available/app.conf and add the following
```xml
<VirtualHost *:80>
  ServerName ec2-18-197-75-198.eu-central-1.compute.amazonaws.com
  ServerAdmin irzelindo.salvador@hotmail.com
  WSGIScriptAlias / /var/www/app/app.wsgi
<Directory /var/www/app/app/>
  Order allow,deny
  Allow from all
</Directory>
  Alias /static /var/www/app/app/static
<Directory /var/www/app/app/static/>
  Order allow,deny
  Allow from all
</Directory>
  ErrorLog ${APACHE_LOG_DIR}/error.log
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
- Enable the virtual host with the following command: ```sudo a2ensite app```

## Create app.wsgi and add the following:

```sudo touch /var/www/app/app.wsgi```
```sudo nano app.wsgi``` and add the following:

```python
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/app/app")

from app import app as application
application.secret_key = 'super_secret_key'
```

## run the app by following this link: [Item Catalog Application](http://ec2-18-197-75-198.eu-central-1.compute.amazonaws.com)

    
