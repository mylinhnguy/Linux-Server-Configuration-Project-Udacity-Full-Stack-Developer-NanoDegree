# Configure Linux Server Project

### Project Overview
Install and configure a Linux web server on a virtual machine AWS and prepare it to host a web applications. 
Secure and install updates the server from a number of attack vectors.
Install and configure a database server.
Deploy one of my existing web applications onto it and the web application is running live on the a secure web server.
You can visit www.totalbeautyzone.com for the website deployed.
 
- The virtual server is [Amazon Lighsail](https://lightsail.aws.amazon.com/).
- The database server is [PostgreSQL](https://www.postgresql.org/).
- The web application is my [Item Catalog project](https://github.com/mylinhnguy/Item-Catalog-Project-Udacity-Full-Stack-Developer-NanoDegree) created earlier in this Full Stack Web Developer Nanodegree Program
- My local machine is a Dell Pro 

### Why this Project?
A deep understanding of exactly what web applications are doing, how they are hosted, and the interactions between multiple systems are what define you as a Full Stack Web Developer. In this project, you’ll be responsible for turning a brand-new, bare bones, Linux server into the secure and efficient web application host your applications need.

### What will I Learn?
Learn how to access, secure, and perform the initial configuration of a bare-bones Linux server.
Learn how to install and configure a web and database server and actually host a web application.

## Get a server. Launch Virtual Machine

### 1. Start a new Ubuntu Linux server instance on Amazon Lightsail 

- [Sign up AWS](https://aws.amazon.com/) and sign in to the console and look for the Amazon Lightsail 

  <img src="https://github.com/mylinhnguy/Linux-Server-Configuration-Project-Udacity-Full-Stack-Developer-NanoDegree/blob/master/images/AmazonLightsail.PNG" width="600px"> 

- Once you are login into the site, click `Create instance`. 
- Choose `Linux/Unix` platform, `OS Only` and  `Ubuntu 16.04 LTS`.
- Choose a instance plan 
- Keep the default name provided by AWS or rename your instance.
- Click the `Create` button to create the instance.
- Wait for the instance to start up.

### 2. SSH into the server

- From the account menu on Amazon Lightsail, click on `SSH keys` tab and download the Default Private Key.
- Move this private key file named `LightsailDefaultPrivateKey-*.pem` into the local folder `~/.ssh` and rename it `Lightsail_Key.pem`.
- To connect to the instance via the terminal: `ssh -i ~/.ssh/Lightsail_Key.pem ubuntu@18.209.97.219` 
  where `18.209.97.219` is the public IP address of the instance.

## Secure the server

### 1. Update and upgrade installed pack
    
    sudo apt-get update
    sudo apt-get upgrade

#### 2. Automatic Updates

- The `unattended-upgrades` package can be used to automatically install updated packages

  ```
  sudo apt install unattended-upgrades
  ```

- Edit `sudo nano /etc/apt/apt.conf.d/50unattended-upgrades`, uncomment the line  `${distro_id}:${distro_codename}-security` and save it.

  ```
  Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}";
    "${distro_id}:${distro_codename}-security";
  //"${distro_id}:${distro_codename}-updates";
  //"${distro_id}:${distro_codename}-proposed";
  //"${distro_id}:${distro_codename}-backports";
  };
  ```
- Modidy automatic updates `sudo nano/etc/apt/apt.conf.d/20auto-upgrades`, so that the upgrades are downloaded and installed every day

  ```
  APT::Periodic::Update-Package-Lists "1";
  APT::Periodic::Download-Upgradeable-Packages "1";
  APT::Periodic::AutocleanInterval "7";
  APT::Periodic::Unattended-Upgrade "1";
  ```
- Enable it, run `sudo dpkg-reconfigure --priority=low unattended-upgrades`
- Restart Apache: `sudo service apache2 restart`

#### 3. Updated packages to most recent versions

- Some packages have not been updated because the server need to be rebooted
  
  ```
  sudo apt-get update
  sudo apt-get dist-upgrade
  sudo shutdown -r now
  ```  
#### 4. The `PermitRootLogin` property should be set to `no` so a root user cannot be used to manipulate your server

  ```
  sudo nano cat /etc/ssh/sshd_config
  ```  
- Change this config to `no`

  ```
  # Authentication:
  LoginGraceTime 120
  PermitRootLogin prohibit-password
  StrictModes no
  ```
  
 <img src="https://github.com/mylinhnguy/Linux-Server-Configuration-Project-Udacity-Full-Stack-Developer-NanoDegree/blob/master/images/PermitRootLogin.PNG" width="600px">
        
### 2. Change the SSH port from 22 to 2200

- Edit `sudo nano /etc/ssh/sshd_config`.
- Change the port number from `22` to `2200`.
- Save and exit using CTRL+X and confirm with Y.
- Restart SSH: `sudo service ssh restart`.

### 3. Configure the Uncomplicated Firewall (UFW) 

- Only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

    ```
    sudo ufw status                  # The UFW is inactive.
    sudo ufw default deny incoming   # Deny any incoming traffic.
    sudo ufw default allow outgoing  # Allow outgoing traffic.
    sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200.
    sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.
    sudo ufw allow www               # Allow HTTP traffic in.
    sudo ufw deny 22                 # Deny tcp and udp packets on port 22.
    ```

- Turn UTW on: `sudo ufw enable`. The output should be like this :

    ```
    Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
    Firewall is active and enabled on system startup
    ```
- Check the current status of UFW: `sudo ufw status`. The output should be like this:

    ```
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
    ```
- Exit the SSH connection: `exit`.

- Click on the `Manage` option of the Amazon Lightsail Instance, select the `Networking` tab, and  the firewall configuration to match the firewall settings above.

  <img src="https://github.com/mylinhnguy/Linux-Server-Configuration-Project-Udacity-Full-Stack-Developer-NanoDegree/blob/master/images/LightsailInstance.PNG" width="600px">

- Allow ports 80(TCP), 123(UDP), and 2200(TCP), and deny the default port 22.

  <img src="https://github.com/mylinhnguy/Linux-Server-Configuration-Project-Udacity-Full-Stack-Developer-NanoDegree/blob/master/images/ufw.PNG" width="600px">

- From your local terminal, run: `ssh -i ~/.ssh/Lightsail_Key.pem -p 2200 ubuntu@18.209.97.219`, where `18.209.97.219` is the public IP address of the instance.

## Give `grader` access

### 1. Create a new user account named `grader`

- While logged in as `ubuntu`, add user grader: `sudo adduser grader`. 
- Enter a password (twice) and fill out information for this new user.

### 2. Give `grader` the permission to sudo

- Edits the sudoers file: `sudo visudo`.

- Add a new line to give sudo privileges to `grader` user.

    ```
    root    ALL=(ALL:ALL) ALL
    grader  ALL=(ALL:ALL) ALL
    ```
- Save and exit using CTRL+X and confirm with Y.
- Verify that `grader` has sudo permissions, run `su - grader`, enter the password.
- Run `sudo -l` . The output should be like this:

    ```
    Matching Defaults entries for grader on ip-172-26-3-64.ec2.internal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

    User grader may run the following commands on ip-172-26-3-64.ec2.internal:
    (ALL : ALL) ALL
    ```

### 3. Create an SSH key pair for `grader` using the `ssh-keygen` tool
          
    On the local machine:
      a. Run `ssh-keygen`
      b. Save the key called `grader_key` in the local directory `~/.ssh`
      c. Enter in a passphrase twice. Two files will be generated (`~/.ssh/grader_key` and `~/.ssh/grader_key.pub`)
      d. Run `cat ~/.ssh/grader_key.pub` and copy the contents of the file 
    On the grader's virtual machine:
      a. Log in to the grader's virtual machine
      b. Create a new directory called `~/.ssh` (`mkdir .ssh`)
      c. Run `sudo nano ~/.ssh/authorized_keys` and paste the content grader_key.pub into this file, save and exit
      d. Give the permissions: `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
      e. Check in `/etc/ssh/sshd_config` file , update `PasswordAuthentication` is set to `no`
      f. Restart `sudo service ssh restart`
         On the local machine, run: `ssh -i ~/.ssh/grader_key -p 2200 grader@18.209.97.219`, 
         where  Public IP Address: 18.209.97.219  and  Private Key ( is not provided here )      
  
## Prepare to deploy your project
 
### 1. Configure the local timezone to UTC

- While logged in as `grader`, configure the time zone `sudo dpkg-reconfigure tzdata`. The output should be like this:

	```
	Current default time zone: 'America/Los_Angeles'
	Local time is now:      Fri Apr  5 11:21:04 PDT 2019.
	Universal Time is now:  Fri Apr  5 18:21:04 UTC 2019
	```

### 2. Install and configure Apache to serve a Python mod_wsgi application

- While logged in as `grader`, install Apache `sudo apt-get install apache2`
- Enter public IP of the Amazon Lightsail instance into browser. If Apache is working, you should see:

   <img src="https://github.com/mylinhnguy/Linux-Server-Configuration-Project-Udacity-Full-Stack-Developer-NanoDegree/blob/master/images/apache2.PNG" width="600px">

- The project is built with Python2, install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
- Enable `mod_wsgi`, run  `sudo a2enmod wsgi`
- Restart Apache `sudo service apache2 restart`

### 3. Install and configure PostgreSQL

- While logged in as `grader`, install PostgreSQL `sudo apt-get install postgres`

- PostgreSQL should not allow remote connections  `sudo nano /etc/postgresql/9.5/main/pg_hba.conf`

- Login as user "postgres" `sudo su - postgres`

- Open PostgreSQL interactive terminal with `psql`

- Create a new database named catalog and create a new user named catalog in postgreSQL shell

	```
	postgres=# CREATE DATABASE catalog;
	postgres=# CREATE USER catalog;
	```
- Set a password for user catalog;
	
	```
	postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
	```
- Give user "catalog" permission to "catalog" application database
	
	```
	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```
- Verify the new database "catalog" has been created

    ```
    postgres-# \l
    postgres-# \du
    ```
- Connect to the database catalog

	```
	postgres=# \c catalog
	catalog=# select * from category_item
    catelog=# select * from user
    catelog=# select * from categories
	```

- Quit postgreSQL

    ```
    postgres=# \q 
    ```
- Exit from user "postgres
	
	```
	exit
	```  
### 4. Install git using

    sudo apt-get install git
    
## Deploy the Item Catalog Project.

### 1. Clone and setup Item Catalog App Project from the Github repository you created earlier in this Nanodegree program

- While logged in as `grader`, `cd /var/www`  

- Create the application directory `sudo mkdir FlaskApp` 

- Move inside this directory using `cd FlaskApp`

- Clone the Item Catalog App to the virtual machine in /var/www/FlaskApp

    ```
    git clone https://github.com/mylinhnguy/Item-Catalog-Project-Udacity-Full-Stack-Developer-NanoDegree.git
    ```
- Rename the application's name 

    ```
    sudo mv ./Item-Catalog-Project-Udacity-Full-Stack-Developer-NanoDegree ./FlaskApp
    ```
- Move to the inner FlaskApp directory using `cd FlaskApp`

- Inside /var/www/FlaskApp/FlaskApp rename `project.py` to `__init__.py` using `sudo mv project.py __init__.py`
   
- In `__init__.py` Replace 

  ```
  # app.run(host="0.0.0.0", port=8000, debug=True)
  app.run()

  # Apache error log= json.loads(open('client_secrets.json', 'r').read())['web']['client_id']
  Apache error log=json.loads(open('/var/www/FlaskApp/FlaskApp/client_secrets.json','r').read())['web']['client_id']

  # oauth_flow = flow_from_clientsecrets('client_secrets.json', scope='')
  oauth_flow = flow_from_clientsecrets('/var/www/FlaskApp/FlaskApp/client_secrets.json', scope='')
  ```

- In `database_setup.py`, `lotsofitems.py` and `project.py` Replace

    ```
    #engine = create_engine('sqlite:///catalog.db')
    engine = create_engine('postgresql://catalog:password@localhost/catalog')
    ```

### 2. Install the following dependencies: 

    While logged in as `grader'
    ```  
    sudo apt-get install -y httplib2
    sudo apt-get install -y requests
    sudo apt-get install -y python-requests   
    sudo apt-get install -y python-oauth2client 
    sudo apt-get install -y python-sqlalchemy
    sudo apt-get install -y flask     
    sudo apt-get install -y python-psycopg2
    sudo apt-get install -y install -y python-flask
    ```
        
### 3. Configure and Enable a New Virtual Host

- Create FlaskApp.conf to edit:

    ```
    sudo nano /etc/apache2/sites-available/FlaskApp.conf
    ```
- Add the following lines of code to the file to configure the virtual host. 
	
	```	
	  <VirtualHost *:80>
        ServerName www.totalbeautyzone.com
        ServerAdmin webmaster@localhost
        WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
        <Directory /var/www/FlaskApp/FlaskApp/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/FlaskApp/FlaskApp/static
        <Directory /var/www/FlaskApp/FlaskApp/static/>
                Order allow,deny
                Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
     </VirtualHost>
	```

- Enable the virtual host with the following command: `sudo a2ensite FlaskApp`. The following prompt will be returned:
   
    ```
    Enabling site catalog.
    To activate the new configuration, you need to run:
    service apache2 reload
    ```

### 4. Create the .wsgi File

- Create the .wsgi File under /var/www/FlaskApp: 
	
	```
	cd /var/www/FlaskApp
	sudo nano flaskapp.wsgi 
	```
- Add the following lines of code to the flaskapp.wsgi file:
	
	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/FlaskApp/")
	from FlaskApp import app as application
	application.secret_key = 'Add your secret key'
	```

- Disable the default Apache site: `sudo a2dissite 000-default.conf`.  The following prompt will be returned:

    ```
    Site 000-default disabled.
    To activate the new configuration, you need to run:
    service apache2 reload
    ```
- Reload Apache: `sudo service apache2 reload`.

### 5. Update on OAuth2 authentication using Google Sign-in API to make the app run on AWS

- Go to [Google APIs](https://console.developers.google.com).
- Go to the project on the Developer Console, and navigate to APIs & Auth > Credentials > Edit Settings. 
- Add the hostname www.totalbeautyzone.com  to the Authorized JavaScript origins.  
- hostname + '/oauth2callback', hostname + '/gconnect' to Authorized redirect URIs.

### 6. Launch the Web Application

- Create database schema 

    ```
    sudo python database_setup.py
    ```
- Populate data into tables: categories, category_item, user run 

    ```
    sudo python lotsofitems.py
    ```
- Run the web application 

    ```
    sudo python __init__.py
    ```
- Open your browser to www.totalbeautyzone.com 
- If everything works, the application should come up 
- If getting an internal error, check the Apache error files  
- `sudo tail /var/log/apache2/error.log`

### 7. Restart Apache and Debug website

-  Restart Apache `sudo service apache2 restart`
-  To get error log messages from Apache server `sudo tail /var/log/apache2/error.log`

## Project Structure 

The project structure should look like

    ```
    /var/www/FlaskApp
    |--  flaskapp.wsgi 
    |__ /FlasKApp           
        |-- __Init__.py 
        |-- client_secrets.json
        |-- database_setup.py 
        |-- lotsofitems.py
        |-- catalog.db 
        |-- LICENSE 
        |-- README.md 
        |-- /static 
            |-- style.css 
        |-- /Templates 
            |-- catalog.html 
            |-- category.html 
            |-- deleteitem.html
            |-- edititem.html 
            |-- footer.html 
            |-- header.html 
            |-- item.html
            |-- login.html 
            |-- newitem.html 
            |-- publiccatalog.html
            |-- publicitem.html 
    ```
    
## References:
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
