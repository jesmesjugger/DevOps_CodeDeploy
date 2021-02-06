########
Deployment on Linux
###################
Using codeDeploy to deploy a wordpressDownload and install Git on your development machine.
***************************************************************
1. use : mkdir -p /tmp/WordPress
############################################
2. In the /tmp/WordPress folder, call the git init command.
#################################################
Call the git clone command to clone the public WordPress repository, making your own copy of it in the /tmp/WordPress destination folder:
#############################################
git clone https://github.com/WordPress/WordPress.git /tmp/WordPress
#############################################
Create scripts to run your application
#############################################
Create a scripts directory in your copy of the WordPress source code:
mkdir -p /tmp/WordPress/scripts
###################################
Create an install_dependencies.sh file in /tmp/WordPress/scripts. Add the following lines to the file.
This install_dependencies.sh script installs Apache, MySQL, and PHP. It also adds MySQL support to PHP
##########################################

#!/bin/bash
sudo amazon-linux-extras install php7.4
sudo yum install -y httpd mariadb-server php
#####################################
Create a start_server.sh file in /tmp/WordPress/scripts. Add the following lines to the file. 
This start_server.sh script starts Apache and MySQL.
######################################

#!/bin/bash
systemctl start mariadb.service
systemctl start httpd.service
systemctl start php-fpm.service
#######################################
Create a stop_server.sh file in /tmp/WordPress/scripts. 
Add the following lines to the file. This stop_server.sh script stops Apache and MySQL.
##########################################

#!/bin/bash
isExistApp=pgrep httpd
if [[ -n $isExistApp ]]; then
systemctl stop httpd.service
fi
isExistApp=pgrep mysqld
if [[ -n $isExistApp ]]; then
systemctl stop mariadb.service
fi
isExistApp=pgrep php-fpm
if [[ -n $isExistApp ]]; then
systemctl stop php-fpm.service

fi
##################################################
Create a create_test_db.sh file in /tmp/WordPress/scripts. Add the following lines to the file. 
This create_test_db.sh script uses MySQL to create a test database for WordPress to use.
########################################################

#!/bin/bash
mysql -uroot <<CREATE_TEST_DB
CREATE DATABASE IF NOT EXISTS test;
CREATE_TEST_DB
##########################################################
Finally, create a change_permissions.sh script in /tmp/WordPress/scripts.
This is used to change the folder permissions in Apache.
##########################################################

#!/bin/bash
chmod -R 777 /var/www/html/WordPress

chmod +x /tmp/WordPress/scripts/*

############################################################
The AppSpec file must be named appspec.yml. 
It must be placed in the root directory of the application's source code. 
#############################################################
appspec.yml 
#############################################################
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html/WordPress
hooks:
  BeforeInstall:
    - location: scripts/install_dependencies.sh
      timeout: 300
      runas: root
  AfterInstall:
    - location: scripts/change_permissions.sh
      timeout: 300
      runas: root
  ApplicationStart:
    - location: scripts/start_server.sh
    - location: scripts/create_test_db.sh
      timeout: 300
      runas: root
  ApplicationStop:
    - location: scripts/stop_server.sh
      timeout: 300
      runas: root
#############################################################