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
 IP Address : 54.200.16.195

 Private IP : 172.26.0.10
 
 SSH Port : 2200 
 
* Download the default private key 
* Move the private key file into the folder ~/.ssh (where ~ is your environment's home directory). like, you downloaded the file to the documents folder, just execute the following command in your terminal. mv ~/documents/lightsail_key.rsa ~/.ssh/  

* Open your terminal and type in chmod 600 ~/.ssh/lightsail_key.rsa
* In your terminal, type in ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@54.200.16.195, where 54.200.16.195 is the public IP address of instance.

# Secure the server
_____________________________________
** Update and upgrase installed packages :

 *  $ sudo apt-get update
 *  $ sudo apt-get upgrade

# Change the SSH port from 22 to 2200
* Edit the /etc/ssh/sshd_config file: sudo nano /etc/ssh/sshd_config.
* Change the port number on line 5 from 22 to 2200.
* Save and exit using CTRL+X and confirm with Y.
* Restart SSH: sudo service ssh restart

# Configure the Uncomplicated Firewall (UFW) 
* Configure the default firewall for ubuntu to only incoming connection for SSH (port 2200), HTTP (port 80), and NTP (port 123).
    
      sudo ufw status                  # The UFW should be inactive.
      sudo ufw default deny incoming   # Deny any incoming traffic.
      sudo ufw default allow outgoing  # Enable outgoing traffic.
      sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200.
      sudo ufw allow www               # Allow HTTP traffic in.
      sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.
      sudo ufw deny 22                 # Deny tcp and udp packets on port 53.


* Turn UFW on: sudo ufw enable. The output should be like this:

         Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
      Firewall is active and enabled on system startup

* Check the status of UFW to list current roles: sudo ufw status. The output should be like this:

      Status: active

      To                         Action      From
      --                         ------      ----
      2200/tcp                   ALLOW       Anywhere                  
      80/tcp                     ALLOW       Anywhere                  
      123/udp                    ALLOW       Anywhere                  
      22                         DENY        Anywhere                  
      2200/tcp (v6)              ALLOW       Anywhere (v6)             
      80/tcp (v6)                ALLOW       Anywhere (v6)             
      123/udp (v6)               ALLOW       Anywhere (v6)             
      22 (v6)                    DENY        Anywhere (v6)

* Exit the SSH connection:  exit .

* Click on the Manage option of the Amazon Lightsail Instance, then the Networking tab, and then change the firewall configuration to match the internal firewall settings above.
* Allow ports 80(TCP), 123(UDP), and 2200(TCP), and deny the default port 22. 
* From your local terminal, run: ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@54.200.16.195, where 54.200.16.19 is the public IP address of the instance.

* Some packages have not been updated because the server need to be rebooted. I found the solution here.
* I did these commands :

      sudo apt-get update
      sudo apt-get dist-upgrade
      sudo shutdown -r now
      
# Give grader access :

* While logged in as ubuntu, add user: sudo adduser grader.
* Enter a password (twice) and fill out information for this new user.(sonam) 

# Give grader the permission to sudo :

* Edits the sudoers file : sudo visudo .
* Search for the line that look like this :

      root    ALL=ALL(ALL:ALL) ALL
      
* Below this line, add a new line to give sudo privileges to grader user.

       root    ALL=(ALL:ALL) ALL
       grader  ALL=(ALL:ALL) ALL       
      
* Save and exit using CTRL+X and confirm with Y.
* Verify that grader has sudo permissions. Run su - grader, enter the password(sonam), run sudo -l and enter the password again. The output should be like this:

      Matching Defaults entries for grader on ip-172-26-13-170.us-east-2.compute.internal:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

      User grader may run the following commands on ip-172-26-13-134.us-east-2.compute.internal:
      (ALL : ALL) ALL
      
# Create an SSH key pair for grader using the ssh-keygen tool :      
* On the local machine :
        
      * Run ssh-keygen 
      * Enter file in which to save the key (I gave the name grader_key) in the local directory ~/.ssh
      * Enter in a passphrase twice. Two files will be generated ( ~/.ssh/grader_key and ~/.ssh/grader_key.pub)
      * Run cat ~/.ssh/grader_key.pub and copy the contents of the file
      * Log in to the grader's virtual machine

* On the grader's virtual machine:

      * Create a new directory called ~/.ssh (mkdir .ssh)
      * Run sudo nano ~/.ssh/authorized_keys and paste the content into this file, save and exit
      * Give the permissions: chmod 700 .ssh and chmod 644 .ssh/authorized_keys
      * Check in /etc/ssh/sshd_config file if PasswordAuthentication is set to no
      * Restart SSH: sudo service ssh restart

* On the local machine, run: ssh -i ~/.ssh/grader_key -p 2200 grader@54.200.16.195 . If, it is not working then try , $ su - grader and enter the grader's password.

#  Configure the local timezone to UTC :
 * While logged in as grader, configure the time zone: sudo dpkg-reconfigure tzdata. You should see something like that:
 
       Current default time zone: 'America/Montreal'
       Local time is now:      Thu may 19 21:55:16 EDT 2019.
       Universal Time is now:  Fri may 20 01:55:16 UTC 2019.

# Install and configure Apache to serve a Python mod_wsgi application :

* While logged in as grader, install Apache: sudo apt-get install apache2.
* Enter public IP of the Amazon Lightsail instance into browser. You should see Apache2 Ubuntu Default Page .
* My project is built with Python 3. So, I need to install the Python 3 mod_wsgi package:

      sudo apt-get install libapache2-mod-wsgi-py3 
 *  Enable mod_wsgi using: sudo a2enmod wsgi.  

# Install and configure PostgreSQL :
* While logged in as grader, install PostgreSQL: 
      $ sudo apt-get install postgresql.

* PostgreSQL should not allow remote connections. In the /etc/postgresql/9.5/main/pg_hba.conf file, you should see:

      local   all             postgres                                peer
      local   all             all                                     peer
      host    all             all             127.0.0.1/32            md5
      host    all             all             ::1/128                 md5

* Switch to the postgres user: 
     $ sudo su - postgres
* Open PostgreSQL interactive terminal with :
          $psql
          
* Create the catalog user with a password and give them the ability to create databases:       

      postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
      postgres=# ALTER ROLE catalog CREATEDB;
          
* List the existing roles: \du. The output should be like this:
 
                                  List of roles
       Role name |                         Attributes                         | Member of 
          -----------+------------------------------------------------------------+-----------
      catalog   | Create DB                                                  | {}
      postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 
* Exit psql:   \q. 
* Switch back to the grader user:   exit.      

* Create a new Linux user called catalog: 

      $ sudo adduser catalog. Enter password (sonam) and fill out information.

* Give to catalog user the permission to sudo. Run:  sudo visudo.
* Search for the lines that looks like this:

      root    ALL=(ALL:ALL) ALL
      grader  ALL=(ALL:ALL) ALL
* Below this line, add a new line to give sudo privileges to catalog user.

      root    ALL=(ALL:ALL) ALL
      grader  ALL=(ALL:ALL) ALL
      catalog  ALL=(ALL:ALL) ALL

* Save and exit using CTRL+X and confirm with Y.
* Verify that catalog has sudo permissions. Run su - catalog, enter the password, run sudo -l and enter the password again. The output should be like this:
 
       Matching Defaults entries for catalog on ip-172-26-13-134.us-east-2.compute.internal:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

      User catalog may run the following commands on ip-172-26-13-134.us-east-2.compute.internal:
       (ALL : ALL) ALL
       
*  While logged in as catalog, create a database:   createdb catalog.      
* Run psql and then run \l to see that the new database has been created. The output should be like this:

                                           List of databases
      Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
      -----------+----------+----------+-------------+-------------+-----------------------
      catalog   | catalog  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
      postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
      template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
      template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
      (4 rows)
      
* Exit psql:   \q.
* Switch back to the grader user:    exit.


# Install git :
* While logged in as grader, install git: 
       $ sudo apt-get install git
       
#  Clone and setup the Item Catalog project from the GitHub repository :
* While logged in as grader, create /var/www/catalog/ directory.
* Change to that directory and clone the catalog project:
  sudo git clone https://github.com/SonamShukla10/ItemCatalog2.git catalog .
* From the /var/www directory, change the ownership of the catalog directory to grader using: sudo chown -R grader:grader catalog/ .
* Change to the /var/www/catalog/catalog directory.
* Rename the item.py file to __init__.py using:  mv item.py __init__.py 
*  **( Committing remove '__' from the init.py. Please add before init and after init)
* In __init__.py replace line :
   
      # app.run(host="0.0.0.0", port=8000)
        app.run()
        
* In db_setup.py, replace :

      # engine = create_engine("sqlite:///db_catalog.db")
      engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')
      
# Install the virtual environment and dependencies    
* While logged in as grader, install pip:   sudo apt-get install python3-pip . 
* Install the virtual environment:   sudo apt-get install python-virtualenv
* Change to the /var/www/catalag directory.
* Create the virtual environment:   sudo virtualenv -p python3 venv3 .
* Change the ownership to grader with:   sudo chown -R grader:grader venv3/ .
* Activate the new environment:   . venv3/bin/activate .

* Install the following dependencies :
    
      pip install httplib2
      pip install requests
      pip install --upgrade oauth2client
      pip install sqlalchemy
      pip install flask
      sudo apt-get install libpq-dev
      pip install psycopg2
      
* Run python3 __init__.py and you should see (If python3 does not work then put python2.7) :
* *  **( Committing remove '__' from the init.py. Please add before init and after init)
      * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)

* Deactivate the virtual environment:   deactivate .

# Set up and enable a virtual host :

* Add the following line in /etc/apache2/mods-enabled/wsgi.conf file to use Python 3 .

      #WSGIPythonPath directory|directory-1:directory-2:...
      WSGIPythonPath /var/www/catalog/venv3/lib/python3.5/site-packages

* Create /etc/apache2/sites-available/catalog.conf and add the following lines to configure the virtual host:

      <VirtualHost *:80>
      ServerName 54.200.16.195
      ServerAlias ec2-54-200-16-195.us-west-2.compute.amazonaws.com
      ServerAdmin grader@54.200.16.195
      WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv3/lib/python2.7/site-packages
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
      
 * Enable virtual host: sudo a2ensite catalog. The following prompt will be returned:
 
       Enabling site catalog.
       To activate the new configuration, you need to run:
       service apache2 reload

* Reload Apache:   sudo service apache2 reload .

# Set up the Flask application :
* Create /var/www/catalog/catalog.wsgi file add the following lines:

      activate_this = '/var/www/catalog/venv3/bin/activate_this.py'
      with open(activate_this) as file_:
      exec(file_.read(), dict(__file__=activate_this))

      #!/usr/bin/python
      import sys
      import logging
      logging.basicConfig(stream=sys.stderr)
      sys.path.insert(0, "/var/www/catalog/catalog/")
      sys.path.insert(1, "/var/www/catalog/")

      from catalog import app as application
      application.secret_key = "..."
      
* Restart Apache:    sudo service apache2 restart.     

* From the /var/www/catalog/catalog/ directory, activate the virtual environment:   . venv3/bin/activate .
 
*  Run: python3 lotsofitems.py . 
* ( If python3 does not work then put python2.7)
*  Deactivate the virtual environment: deactivate .

# Disable the default Apache site :
* Disable the default Apache site:   
$ sudo a2dissite 000-default.conf. The following prompt will be returned:

      Site 000-default disabled.
     To activate the new configuration, you need to run:
     service apache2 reload
     
* Reload Apache:    sudo service apache2 reload.

# LAunch the application :

* Change the ownership of the project directories: sudo chown -R www-lotsofitems:www-lotsofitems catalog/ . 
* Restart Apache again: sudo service apache2 restart.
* Open your browser to http://54.200.16.195 


Third party resources :::
 * https://phoenixnap.com/kb/ssh-to-connect-to-remote-server-linux-or-windows
 * https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
 * https://www.digitalocean.com/community/tutorials/how-to-tune-your-ssh-daemon-configuration-on-a-linux-vps
