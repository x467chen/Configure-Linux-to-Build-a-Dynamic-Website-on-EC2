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
