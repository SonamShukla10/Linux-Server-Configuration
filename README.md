# Linux-Server-Configuration

*Installed Packages:

apache2 -> Web Server.

libapache2-mod-wsgi -> Tool that serves Python web apps.

postgresql -> Relational database system.

sqlalchemy -> SQL toolkit.

flask -> Python web framework.

git -> Version control tool

 *Access :
 _________
 IP Address : 54.203.8.184

 Private IP : 172.26.0.10
 
 SSH Port : 2200
 (If it's not working then run sudo apt-get install ssh or try on port 2222).
 
 Username and password : grader, sonam
 
 Download the private key LightsailDefaultKeyPair.pem
 
 
 
 * Start a new Ubuntu Linux server instance on Amazon Lightsail
* Login into Amazon Lightsail using an Amazon Web Services account.
* Once you are login into the site, click Create instance.
* Choose Linux/Unix platform, OS Only and Ubuntu 16.04 LTS.
* Choose a instance plan (I took the cheapest, $5/month).
* Keep the default name provided by AWS or rename your instance.
* Click the Create button to create the instance.
* Wait for the instance to start up.

 *Configuration:
 ________________
 
 * Download private key from the SSH keys section in the Account section on Amazon Lightsail. The file name should be like LightsailDefaultPrivateKey-us-east-2.pem
 
 * Create a new file named lightsail_key.rsa under ~/.ssh folder on your local machine
 
 * Copy and paste content from downloaded private key file to lightsail_key.rsa
 
 * Set file permission as owner only : $ chmod 600 ~/.ssh/lightsail_key.rsa.
 
 * SSH into the instance: $ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@54.203.8.184
 
 *  Create a new user with name 'grader':
  $ sudo adduser grader
  
* Grant user permission to perform sudo command:
  $ sudo nano /etc/sudoers.d/grader
  $ grader ALL=(ALL) NOPASSWD:ALL 

 
 * Create an SSH key pair for grader using the ssh-keygen tool on your local machine. Save it in ~/.ssh path
    Deploy public key on development environment
    
 * On your local machine, read the generated public key cat ~/.ssh/FILE-NAME.pub
 
 * On your virtual machine
 
   $ mkdir .ssh
   
   $ touch .ssh/authorized_keys
   
   $ nano .ssh/authorized_keys
   
* Copy the public key to this authorized_keys file on the virtual machine and save

* Run chmod 700 .ssh and chmod 644 .ssh/authorized_keys on your virtual machine to change file permission

* Restart SSH: $ sudo service ssh restart

* Now you are able to login in as grader: $ ssh -i ~/.ssh/grader_key -p 2200 grader@54.203.8.184

* If you will be asked for grader's password is sonam or open configuration file again: $ sudo nano /etc/ssh/sshd_config

 * Change PasswordAuthentication yes to no
 
* Restart SSH: $ sudo service ssh restart
   
 *  file name where rsa key pair has saved : lightsailDefaultKeyPair.pem and public key lightsailDefaultKeyPair.pem.pub
   
   And passphrase : sonam

* Login to the server using ssh key:
  $ ssh grader@54.203.8.184 -p PORT_NUMBER -i ~/.ssh/authorized_keys
  
* Get a list of packages to be upgraded:
  $ sudo apt-get update
* Upgrade packages to newer versions:
  $ sudo apt-get upgrade
* Change the SSH port from 22 to 2200
* Open the config file:
  $ sudo nano /etc/ssh/sshd_config
* Change to Port 2200

* Block all incomming requests:
  $ sudo ufw default deny incoming
* Allow all outgoing requests from the server:
  $ sudo ufw default allow outgoing
* Allow incoming connections for SSH (port 2200):
  $ sudo ufw allow 2200/tcp
* Allow incoming connections for HTTP (port 80):
  $ sudo ufw allow www
* Allow incoming connections for NTP (port 123):
  $ sudo ufw allow ntp
  
* Configure the local timezone to UTC
  $ sudo dpkg-reconfigure tzdata   
  
* Install Apache2:
  $ sudo apt-get install apache2
* Install mod_wsgi for serving Python apps from Apache:
  $ sudo apt-get install python-setuptools libapache2-mod-wsgi
* Restart the Apache server for mod_wsgi to load:
  $ sudo service apache2 restart

* Install Git and clone Catalog app:
  $ sudo apt-get install git 
* Install and configure PostgreSQL

* Install PostgreSQL:
  $ sudo apt-get install postgresql postgresql-contrib
  
* Open the database setup file:
  $ sudo vim db_setup.py
* Change from sqlite to postgresql:
  python engine = create_engine('postgresql://catalog:PW-FOR-DB@localhost/catalog')
* Rename item.py: 
  $ mv item.py __init__.py
* Create linux user for psql:
  $ sudo adduser catalog
* Change to default user postgres:
  $ sudo su - postgres
* Connect to the system:
  $ psql
* Create user with LOGIN role and set a password:
  CREATE USER catalog WITH PASSWORD 'SONAM';
* Allow the user to create database tables:
  ALTER USER catalog CREATEDB;
* Create database:
  CREATE DATABASE catalog WITH OWNER catalog;
* Connect to the database catalog:
  \c catalog
* Revoke all rights:
  REVOKE ALL ON SCHEMA public FROM public;
* Grant only access to the catalog role:
  GRANT ALL ON SCHEMA public TO catalog;
* Create postgreSQL database schema:
  $ python db_setup.py
* Run the application 
