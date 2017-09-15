# UdacityFSW_Project6
Udacity Full Stack Web Developer - Linux Server Configuration

##  Server Details
IP address: `18.220.203.131`
SSH port: `2200`
URL: `http://ec2-18-220-203-131.us-east-2.compute.amazonaws.com`

### 1. Update all currently installed packages
Log into ssh instace console (Udacity_Instance2017)  
* Update: 
    `sudo apt-get update` 
* Upgrade: 
    `sudo apt-get upgrade` 
* Further updates 
    `sudo apt-get dist-upgrade` 
    
### 2. Give grader access.
* Create a new user account named grader.  
    * `sudo adduser grader` (add a password and complet information)  
    * `sudo cat /etc/passwd` to confirm creation    
* Give grader the permission to sudo. 
    * `sudo usermod -aG sudo grader`   
    * `sudo touch /etc/sudoers.d/grader` 
    * `sudo vi /etc/sudoers.d/grader` and add `grader ALL=(ALL:ALL) ALL`, save    
* Create an SSH key pair for grader using the ssh-keygen tool. 
    * `ssh-keygen -t rsa -b 4096 -C "grader@18.220.203.131"` 
* Add Public Key to Server 
    * Login to the server as the new grader user 
        * `su grader` 
    * Make permisions to grader         
        * `mkdir /home/grader/.ssh` 
        * `sudo chown grader:grader /home/grader/.ssh` (enter password) 
        * `sudo cp /root/.ssh/authorized_keys /home/grader/.ssh/` 
        * `sudo chown grader:grader /home/grader/.ssh/authorized_keys`  
        * `sudo vi .ssh/authorized_keys` (paste contents of pub keys generated) 
        * `sudo chmod 700 /home/grader/.ssh`  
        * `sudo chmod 644 /home/grader/.ssh/authorized_keys`  
    * Connect from local computer to the server 
        * `ssh -i ~/.ssh/linuxCourse grader@18.220.203.131 -p 22`     

### 3. Remote Login
* Edit the sshd_config file
    * `sudo vi /etc/ssh/sshd_config`
* Disable remote login of root user
    * Change PermitRootLogin: `prohibit-password` to `no`
* Enable grader user remote ssh login 
    * Add thie line: `AllowUsers grader`
* Force Login using Key Pair
    * Edit the sshd_config file: `sudo vi /etc/ssh/sshd_config`
    * Change Password Authentication from `yes` to `no`
* Restart the ssh service   
    * `sudo service ssh restart`
### 4. Configuration UFW (Uncomplicated Firewall)
* Block all incoming connections on all ports:
    * `sudo ufw default deny incoming`
* Allow outgoing connection on all ports:
    * `sudo ufw default allow outgoing`
* Allow incoming connection for SSH on port 2200:
    * `sudo ufw allow 2200/tcp`
* Allow incoming connections for HTTP on port 80:
    * `sudo ufw allow www`
* Allow incoming connection for NTP on port 123:
    * `sudo ufw allow ntp`    
* To check the rules that have been added 
    * `sudo ufw show added`   
* To enable the firewall, use:
    * `sudo ufw enable`     

### 5. Change the SSH port from 22 to 2200
* Edit the sshd_config file
     * `sudo vi /etc/ssh/sshd_config`
* Change Port 22 to Port 2200 (also enable port 2200 on Amazon Lightsail)
* Restart the ssh service   
     * `sudo service ssh restart`
* Test login from local
     * `ssh -i ~/.ssh/linuxCourse grader@18.220.203.131 -p 2200`     

### 6. Prepare to deploy your project.
* Configure the local timezone to UTC.
    * `sudo timedatectl set-timezone UTC`
* Install/configure Apache to serve a Python mod_wsgi application.    
    * `sudo apt-get install apache2`
    * `sudo apt-get install libapache2-mod-wsgi`   
* Install and configure PostgreSQL
    * `sudo apt-get install postgresql postgresql-contrib`
* Denied remote connections
    *  Check file: `/etc/postgresql/9.5/main/pg_hba.conf` (By default, remote connections to the database are disabled for security reasons when installing PostgreSQL from the Ubuntu repositories)     
### 7. Database
* Set up: 
    * `sudo -u postgres psql postgres`

* Set-up a password for user postgres:
    * `\password postgres` and enter a password (postgres)

* Connect and create catalog: `sudo su - postgres`
* Promp to psql: Type `psql`    
* Create a new user
    * `CREATE USER catalog WITH PASSWORD 'catalog';`
* Confirm it whith 
    * `\du`
* Update permissions for new user
    * `ALTER ROLE catalog WITH LOGIN;`
    * `ALTER USER catalog CREATEDB;`   
* Create database
    * `CREATE DATABASE catalog WITH OWNER catalog;`
* Login the database
    * `\c catalog` 
* Grant only access to the catalog role 
    * `GRANT ALL ON SCHEMA public TO catalog;` 
* Exit: `\q`, then `exit`
* Restart postgresql
    * `sudo service postgresql restart`   

### 8. Project requisites.
* Install Git 
    * `sudo apt-get install git`
* Edit Git Configuration
    * `git config --global user.name "xxxxx"`
    * `git config --global user.email xxxx@xxxx.xxx`
* Confirm it with 
    * `git config --list`

* Install Flask, SQLAlchemy, etc    
    ```
    sudo apt-get install python-psycopg2 python-flask
    sudo apt-get install python-sqlalchemy python-pip
    sudo pip install psycopg2
    sudo pip install oauth2client
    sudo pip install requests
    sudo pip install httplib2
    sudo pip install flask-seasurf
    ```    

### 9. Set Up Aplication.
* Move to the /var/www directory 
    * `cd /var/www`
* Create Folder FlaskApp
    * `sudo mkdir Flask`
* Move inside this directory
    * `cd Flask`
* Clone the Catalog App to the virtual machine
    * `git clone https://github.com/lmichilot/UdacityFSW_Project4.git`
* Rename project directoty to FlaskApp
    * `sudo mv ./UdacityFSW_Project4 ./Flask`
* Move inside this directory using  
    * `cd Flask`
* Edit and change files:
  * database_setup.py 
  * database_seed.py
  * project.py
    * From: `engine = create_engine('sqlite:///dbcatalog.db')`
    * to: `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
* Execute:
    * `sudo python database_setup.py`
    * `sudo python database_seed.py`
* Rename project.py to __init__.py 
    * `sudo mv project.py __init__.py`
* Install dependencies
    * `sudo pip install -r requirements.txt`    

### 10. Virtual environment
* Install virtualenv
    * `sudo pip install virtualenv`
* Create temporary environment    
    * `sudo virtualenv Virtenv`
* Install Flask in that environment    
    * `source Virtenv/bin/activate`
* Test Application 
    * `sudo python __init__.p`
        * "It should display “Running on http://localhost:5000/” If you see this message, you have successfully configured the app."

* Deactivate the virtual environment
    * `deactivate`
### 11. Configure and Enable a New Virtual Host
* Create FlaskApp.conf under /etc/apache2/sites-available/:
    * `sudo nano /etc/apache2/sites-available/FlaskApp.conf`

* Add the following lines of code to the flaskapp.wsgi file:
   ```
  <VirtualHost *:80>
      ServerName 18.220.203.131
      ServerAlias ec2-18-220-203-131.us-east-2.compute.amazonaws.com
      ServerAdmin lmichilot@gmail.com
      WSGIScriptAlias / /var/www/Flask/flaskapp.wsgi
      <Directory /var/www/Flask/Flask/>
          Order allow,deny
          Allow from all
      </Directory>
      Alias /static /var/www/Flask/Flask/static
      <Directory /var/www/Flask/Flask/static/>
          Order allow,deny
          Allow from all
      </Directory>
      ErrorLog /var/www/Flask/error.log
      LogLevel warn
      CustomLog /var/www/Flask/access.log combined
  </VirtualHost>
   ```
### 13. Create the .wsgi File      
* Create the .wsgi File under /var/www/FlaskApp:
    `cd /var/www/Flask`
    `sudo vi flaskapp.wsgi`
* Add the following lines of code to the flaskapp.wsgi file:
    ```
      #!/usr/bin/python
      import sys
      import logging
      logging.basicConfig(stream=sys.stderr)
      sys.path.insert(0,"/var/www/Flask/")

      from Flask import app as application
      application.secret_key = 'Add your secret key'
    ```    
### 13. Finalize
* Restart Apache:
    * `sudo service apache2 restart`

* Visit site at http://ec2-18-220-203-131.us-east-2.compute.amazonaws.com/    

### *** Third-party resources/tutorials
* https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
* https://devops.profitbricks.com/tutorials/deploy-a-flask-application-on-ubuntu-1404/
* https://blog.tersmitten.nl/ufw-delete-firewall-rules-by-number.html
* https://aws.amazon.com/es/documentation/lightsail/    
* https://console.developers.google.com 
* http://www.techrepublic.com/article/how-to-start-stop-and-restart-services-in-linux/ 


