# Linux-Server-Configuration

*Installed Packages:

apache2 -> Web Server.

libapache2-mod-wsgi -> Tool that serves Python web apps.

postgresql -> Relational database system.

sqlalchemy -> SQL toolkit.

flask -> Python web framework.

git -> Version control tool


** Start a new Ubuntu Linux server instance on Amazon Lightsail
** Login into Amazon Lightsail using an Amazon Web Services account.
** Once you are login into the site, click Create instance.
** Choose Linux/Unix platform, OS Only and Ubuntu 16.04 LTS.
** Choose a instance plan (I took the cheapest, $5/month).
** Keep the default name provided by AWS or rename your instance.
** Click the Create button to create the instance.
** Wait for the instance to start up.

 *Access :
 _________
 IP Address : 54.184.57.2

 Private IP : 172.26.0.10
 
 SSH Port : 2200 
 
* Download the default private key 
* Move the private key file into the folder ~/.ssh (where ~ is your environment's home directory). like, you downloaded the file to the documents folder, just execute the following command in your terminal. mv ~/documents/udacity_key.rsa ~/.ssh/  

* Open your terminal and type in chmod 600 ~/.ssh/udacity_key.rsa
* In your terminal, type in ssh -i ~/.ssh/udacity_key.rsa ubuntu@54.184.57.2

# Create a new user grder ::
 
     sudo adduser grader ( Password = sonam)
     sudo apt-get install nano
     sudo nano /etc/sudoers
     sudo nano /etc/sudoers.d/grader
     sudo nano /etc/sudoers.d/grader, type in grader ALL=(ALL:ALL) ALL, save and     quit
     
* generate keys on local machine using $ ssh-keygen ( Passphrase = sonam) ; then save the private key in  ~/.ssh on local machine

* deploy public key on developement enviroment On you virtual machine:

      $ su - grader
      $ mkdir .ssh
      $ touch .ssh/authorized_keys
      $ vim .ssh/authorized_keys     

* Copy the public key generated on your local machine to this file and save

   $ chmod 700 .ssh
   $ chmod 644 .ssh/authorized_keys
  
 * reload SSH using service ssh restart
 
* now you can use ssh to login with the new user you created
   ssh -i [privateKeyFilename] grader@54.184.57.2

*  $ sudo apt-get update
*  $ sudo apt-get upgrade

* Use sudo vim /etc/ssh/sshd_config and then change Port 22 to Port 2200 , save & quit.
* Reload SSH using sudo service ssh restart
 
* Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

$ sudo ufw allow 2200/tcp
$ sudo ufw allow 80/tcp
$ sudo ufw allow 123/udp
$ sudo ufw enable  


# Configure the local timezone to UTC
* Configure the time zone sudo dpkg-reconfigure tzdata
* It is already set to UTC.

# Install apache::
* $ sudo apt-get install apache2
* install mod_wsgi, $ sudo apt-get install python-setuptools libapache2-mod-wsgi
* $ sudo service apache2 restart

# Install and configure postgresql ::
* $ sudo apt-get install postgresql
* $ sudo vim /etc/postgresql/9.3/main/pg_hba.conf
* $ sudo su - postgres
* $ psql

# Create Database ::
* postgres=# CREATE DATABASE catalog;  
* postgres=# CREATE USER catalog;
* postgres=# ALTER ROLE catalog WITH PASSWORD 'sonam';
* Give permission, postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
* postgres=# \q
* Exit from postgres:::
    exit
    
# Install git, clone and setup ItemCatalog project:::
* Install Git, $ sudo apt-get install git
* Use cd /var/www to move to the /var/www directory
* Create the application directory sudo mkdir FlaskApp
* Move inside this directory using cd FlaskApp
* Clone the Catalog App to the virtual machine, $ git clone https://github.com/SonamShukla10/ItemCatalog.git
* Rename the project's name sudo mv ./ItemCatalog ./FlaskApp
* Move to the inner FlaskApp directory using, $ cd FlaskApp
* Rename item.py to __init__.py using,
       $ sudo mv item.py __init__.py
* Edit db_setup.py, item.py and lotsofitems.py and change engine = create_engine('sqlite:///db_catalog.db') to
    engine =  create_engine('postgresql://catalog:sonam@localhost/catalog')
* $ sudo apt-get install python-pip
* Install psycopg2 sudo apt-get -qqy install postgresql python-psycopg2
* Create database schema, $ sudo python db_setup.py

# Configure and Enable a New Virtual Host ::
  * Create FlaskApp.conf to edit: $ sudo nano /etc/apache2/sites-available/FlaskApp.conf
  
* Add the following lines of code to the file to configure the virtual host.

         <VirtualHost *:80>      
         ServerName 54.184.57.2
         ServerAdmin admin@54.184.57.2
         WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
        <Directory /var/www/FlaskApp/FlaskApp/>
        Order allow,deny	
        Allow from all	
        </Directory>
        Alias /static /var/www/FlaskApp/FlaskApp/static
        <Directory /var/www/FlaskApp/FlaskApp/static/>
        Order allow,deny	
        Allow from all	
        </Directory>  ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>

* Enable the virtual host with the following command: $ sudo a2ensite FlaskApp

# Create the .wsgi File ::
 * $ cd /var/www/FlaskApp
* $ sudo nano flaskapp.wsgi 

* Add the following lines of code to the flaskapp.wsgi file:

          #!/usr/bin/python  
          import sys
          import loggin
          logging.basicConfig(stream=sys.stderr)
          sys.path.insert(0,"/var/www/FlaskApp/")
          from FlaskApp import app as application
          application.secret_key = 'Secret Key'
         
# Restart apache ::
 * $ sudo service apache2 restart


Third party resources :::
 * https://phoenixnap.com/kb/ssh-to-connect-to-remote-server-linux-or-windows
 * https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
 * https://www.digitalocean.com/community/tutorials/how-to-tune-your-ssh-daemon-configuration-on-a-linux-vps
