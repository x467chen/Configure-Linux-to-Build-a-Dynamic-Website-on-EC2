# Synopsis of Linux Server Configuration on EC2
This project is a guide to build a safe WSGI application. Installation of a Linux distribution on a virtual machine and prepare it to host my web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

IP address:13.59.78.168<br> 
SSH port: 2200<br> 
URL for Dynamic Website: http://ec2-13-59-78-168.us-east-2.compute.amazonaws.com<br> 
URL for Static Website: http://xuanqi.xyz<br> 
Note: I configure the AWS S3 and set DNS to run Static Website and the detail of configuration has not posted here. This project is only focus on how to configure a LAMP Website on AWS EC2.


# Motivation
1. Learn the basic knowledge of User Management
 - How to create a new user and log into the EC2 server as the new user by using the submitted key
 - How to set remote login of the root user disabled
 - How to give the new user sudo access
2. Learn the Security Configuration
 - How to build firewall only allow for SSH, HTTP, and NTP
 - Force users required to authenticate using RSA keys
 - Make sure the applications up-to-date
 - Set SSH hosted on non-default port
3. Learn the Application Functionality
 - Configure a web server running on port 80
 - How Database server has been configured to serve data (PostgreSQL).
 - How web server has been configured to serve my restaurant application as a WSGI app.


# Configuration Process
## Get server
1. Configure and launch a new Ubuntu Linux server instance on AWS EC2
2. Setting key not be publicly viewable for SSH to work
```$ chmod 400 xuanqiKey.pem```
3. Follow the instructions provided to SSH into server<br> 
``` $ ssh -i "key.pem" ubuntu@ec2-13-59-78-168.us-east-2.compute.amazonaws.com```

## Update installed packages
1. Update the package indexes:<br> 
```$ sudo apt-get update```

2. Upgrade the installed packages:<br> 
```$ sudo apt-get upgrade```

3. Others Optional configuration:<br> 
check details by: ```man apt-get```<br> 
remove package no longer needed by: ```sudo apt-get auto remove```<br> 
Install a new software finger to check info quickly: ```sudo apt-get install finger```<br> 

## Add user
1. Add new user called grader<br> 
```$ sudo adduser grader```<br> 
2. Check the new user <br> 
```$ finger grader```<br> 
3. Add user grader to sudo group<br> 
```$ sudo usermod -aG sudo grader```<br> 
4. Switch to and test sudo access on new user account<br> 
```$ su - grader```<br> 
```$ sudo cat /etc/passwd```<br> 

## Set-up SSH keys for user grader
1. Copy the public key<br> 
```$ sudo mkdir /home/grader/.ssh```<br> 
```$ sudo cp /home/ubuntu/.ssh/authorized_keys /home/grader/.ssh/```<br> 

2. Set the permission
```$ sudo chown grader:grader /home/grader/.ssh/authorized_keys```<br> 
```$ sudo chmod 700 /home/grader/.ssh```
```$ sudo chown grader:grader /home/grader/.ssh/authorized_keys```<br> 
```$ sudo chmod 644 /home/grader/.ssh/authorized_keys```<br> 

3. Login as the grader user using the command:
```$ ssh -i "xuanqiKey.pem" grader@ec2-13-59-78-168.us-east-2.compute.amazonaws.com```<br> 

## Forcing Key Based Authentication and disable root login
1. Edit the authentication file
```$ sudo nano /etc/ssh/sshd_config```<br> 
a.From PermitRootLogin without-password to PermitRootLogin no.<br> 
b.PasswordAuthentication no<br> 
3. Restart for the changes to take effect<br> 

## Change timezone to UTC
1. Configure the time zone<br>
```$ sudo timedatectl set-timezone UTC```<br>
2. Check timezone<br>
```$ timedatectl status```<br>

## Configure the Uncomplicated Firewall (UFW)
1. Change the SSH port from 22 to 2200(Notes: Also add a new custom port with port 2200 on AWS)<br>
```$ sudo vim /etc/ssh/sshd_config ```<br>

2. Reload SSH using and and login again
```$ sudo service ssh restart  ```<br>
```$ ssh -i "xuanqiKey.pem" grader@ec2-18-216-252-134.us-east-2.compute.amazonaws.com -p 2200 ```<br>

3. By default, block all incoming connections on all ports:
```$ sudo ufw default deny incoming ```<br>

4. Allow outgoing connection on all ports:
```$ sudo ufw default allow outgoing ```<br>

5. Allow incoming connection
```$ sudo ufw allow 2200/tcp ```<br>
```$ sudo ufw allow www ```<br>
```$ sudo ufw allow ntp ```<br>

6. Check the rules
```$ sudo ufw show added ```<br>

7. Start the firewall
```$ sudo ufw enable ```<br>
 
8. Check the status of the firewall
```$ sudo ufw status ```<br>

## Install Apache, Git, Flask, SQLAlchemyand etc.
1.Install Apache 
```$ sudo apt-get install apache2```<br>
2.Install mod_wsgi
```$ sudo apt-get install libapache2-mod-wsgi```<br>
```$ sudo a2enmod wsgi```<br>
```$ sudo service apache2 restart```<br>

3. Other useful module for my project
```
sudo apt-get install git
sudo apt-get install python-psycopg2 python-flask
sudo apt-get install python-sqlalchemy python-pip
sudo pip install flask-seasurf
sudo pip install --upgrade oauth2client
pip install httplib2
pip install requests
pip install Flask-SQLAlchemy
```

## Clone the repository and Configure Apache
1. Create the application directory 
```
cd /var/www
sudo mkdir catalog
```

2. Change owner of the newly created catalog folder<br>
```sudo chown -R grader:grader catalog```

3. Clone the Catalog App to the virtual machine <br>
```git clone https://github.com/x467chen/Item-Catalog-for-Restaurant.git catalog```

4.Create a catalog.wsgi file and add this inside:
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'supersecretkey'
```

5. Move to the inner catalog directory using <br>
```cd catalog```

6. Rename project.py to __init__.py using:
```sudo mv project.py __init__.py```

## Install and configure PostgreSQL
1.Install PostgreSQL 
sudo apt-get install postgresql postgresql-contrib
2. Check if no remote connections are allowed 
sudo subl /etc/postgresql/9.5/main/pg_hba.conf 
3. Configure
```
sudo su - postgres
psql
CREATE USER catalog WITH PASSWORD 'password';
ALTER USER catalog CREATEDB;
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
exit
```
4. Change create engine line in __init__.py and database_setup.py to: engine = create_engine('postgresql://catalog:password@localhost/catalog')

5. Start new PostgreSQL
```python /var/www/catalog/catalog/database_setup.py```

## Update catalog.wsgi file
```
<VirtualHost *:80>
    ServerName 13.59.78.168
    ServerAlias ec2-13-59-78-168.us-east-2.compute.amazonaws.com
    ServerAdmin admin@13.59.78.168
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

## Update OAuth client secrets file
1. Google<br>
 Go to the app website add new "javascript_origins": http://ec2-13-59-78-168.us-east-2.compute.amazonaws.com
 
2. Facebook<br>
Go to the app website add new the website URL: http://ec2-13-59-78-168.us-east-2.compute.amazonaws.com

3. Update all path of json in __init__.py to absolutely path:<br>
```/var/www/catalog/catalog/client_secrets.json```

4. If there is any mistake make sure restart server or log the error:<br>
```sudo cat /var/log/apache2/error.log```

## Restart Apache and visit Website
1. Restart Apache by ```sudo service apache2 restart```
2. Visit site at http://ec2-13-59-78-168.us-east-2.compute.amazonaws.com
