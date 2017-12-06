# Linux Server Configuration on EC2
# Synopsis
This project is a guide to build a safe WSGI application. Installation of a Linux distribution on a virtual machine and prepare it to host my web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

IP address:13.59.78.168
SSH port: 2200
URL for Dynamic Website: http://ec2-13-59-78-168.us-east-2.compute.amazonaws.com
URL for Static Website: http://xuanqi.xyz
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
4. What else

## Configuration Process
# Get server
1. Configure and launch a new Ubuntu Linux server instance on AWS EC2
2. Setting key not be publicly viewable for SSH to work
```$ chmod 400 xuanqiKey.pem```
3. Follow the instructions provided to SSH into server
``` $ ssh -i "key.pem" ubuntu@ec2-18-220-99-128.us-east-2.compute.amazonaws.com```
