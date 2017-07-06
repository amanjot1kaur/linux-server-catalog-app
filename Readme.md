# Linux Server Configuration
A Linux virtual machine needs to be configurated to support the Item Catalog application. This is a set of instructions on how to set up a Linux server to host a simple web application built with Flask. The instructions are written for hosting a Food Menu Catalog App (github link to this application is https://github.com/amanjot1kaur/catalog) on an Amazon Lightsail instance. You can visit http://13.59.147.36 for the website deployed.

## 1. Server specific details
The IP address is 13.59.147.36
The SSH port used is `2200`.
The URL to the hosted webpage is: http://13.59.147.36/ 

## 2. Software to install during the configuration
- Apache2
- mod_wsgi
- PostgreSQL
- git
- oauth2client
- SQLAlchemy
- Flask
- libpq-dev
- Psycopg2
- pip
- virtualenv
- httplib2
- Python Requests

## 3. Configuration steps
### Create an instance with Amazon Lightsail & connect to the insctance on local machine
1. Sign in to [Amazon Lightsail](https://amazonlightsail.com) using an Amazon Web Services account. Create an AWS Account.
2. Login using your account.
3. Create an instance. Choose an instance image:Ubuntu, give your instance a hostname. It take few minutes to startup.
4. Download default Private Key available at the Amazon Lightsail 'account page'.
5. A file called LightsailDefaultPrivateKey.pem will be downloaded. open this in a text editor. Copy the text and put it in a file called udacity_key.rsa
6. Move the private key file into the folder ~/.ssh (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal. mv ~/Downloads/udacity_key.rsa ~/.ssh/
7. Open your terminal and type in chmod 600 ~/.ssh/udacity_key.rsa (I am using git bash in my windows system)
8. In your terminal, type in ssh -i ~/.ssh/udacity_key.rsa ubuntu @13.59.147.36

Note: While Amazon Lightsail provides a broswer-based connection method, this will no longer work once the SSH port is changed from its default value 22. 

### Upgrade currently installed packages
1. Update packages by running `sudo apt-get update`
2. Download available package updates by running `sudo apt-get upgrade`

### Configure the Uncomplicated Firewall (UFW)
1. Run `sudo nano /etc/ssh/sshd_config` to edit the file, change the port number from `22` to `2200`, then restart SSH by running `sudo service ssh restart`. 
2. Check to see if the ufw  is active by running `sudo ufw status`
3. Run `sudo ufw default deny incoming` to set the ufw firewall to block everything coming in
4. Run `sudo ufw default allow outgoing` to set the ufw firewall to allow everything outgoing
5. Run `sudo ufw allow ssh` to set the ufw firewall to allow SSH
6. Run `sudo ufw allow 2200/tcp` to allow all tcp connections for port `2200` so that SSH will work
7. Run `sudo ufw allow www` to set the ufw firewall to allow a basic HTTP server
8. Run `sudo ufw allow 123/udp` to set the ufw firewall to allow NTP
9. Run `sudo ufw deny 22` to deny port `22` (deny this port since virtual machine has now been configured so that SSH uses port `2200`)
10. Run `sudo ufw enable` to enable the ufw firewall
11. Run `sudo ufw status` to check which ports are open and to see if the ufw is active; if done correctly, it should look like this:

	```
	To                         Action      From
	--                         ------      ----
	22                         DENY        Anywhere
	2200/tcp                   ALLOW       Anywhere
	80/tcp                     ALLOW       Anywhere
	123/udp                    ALLOW       Anywhere
	22 (v6)                    DENY        Anywhere (v6)
	2200/tcp (v6)              ALLOW       Anywhere (v6)
	80/tcp (v6)                ALLOW       Anywhere (v6)
	123/udp (v6)               ALLOW       Anywhere (v6)
	```
12. Update the external (Amazon Lightsail) firewall on the browser by clicking on the 'Manage' , then the 'Networking', and then changing the firewall configuration settings as: ports `80`(TCP), `123`(UDP), and `2200`(TCP). Deny the default port `22`)
13. Open Terminal and run:
	`ssh -i ~/.ssh/udacity_key.rsa -p 2200 ubuntu@13.59.147.36`, where 13.59.147.36 is the public IP address of the instance(also mentioned above in server specific details)

### Create and configure `grader` user
1. Run `sudo adduser grader`
2. Enter in a new UNIX password.
3. Fill out information for the new `grader` user
4. Run `su - grader`, and enter the password to switch user from ubuntu to grader.
5. Run `sudo visudo`
6. Search for a line that looks like this:
	`root    ALL=(ALL:ALL) ALL`
7. Add the following line below this one to give permissions to`grader`:
	`grader	   ALL=(ALL:ALL) ALL`
8. Save and close the visudo file

### Allow `grader` to log in to the virtual machine
1. Run `ssh-keygen` on the local machine
2. Choose a file name `id_rsa` for the key pair.
3. Enter in a passphrase twice (two files will be generated; the second one will end in .pub)
4. Deploy public key on developement enviroment
	-On you virtual machine:run following
	`su - grader`
	`mkdir .ssh`
	`touch .ssh/authorized_keys`
	`vim .ssh/authorized_keys`
5. Copy the  public key generated on your local machine, and paste them in the .ssh/authorized_keys file on the virtual machine
6. Run
	`chmod 700 .ssh`
	` chmod 644 .ssh/authorized_keys`
7. To force key-based authentication(log in as `grader`, open the `/etc/ssh/sshd_config` file, and find the line that says, '# Change to no to disable tunnelled clear text passwords'; if the next line says, 'PasswordAuthentication yes', change the 'yes' to 'no'; save and exit the file run `sudo service ssh restart`
8. Now you can use ssh to login with the `grader` user using following command:
`ssh -i ~/.ssh/id_rsa -p 2200 grader@13.59.147.36`

### Configure the local timezone to UTC
1. Run `sudo dpkg-reconfigure tzdata`, and follow the instructions 

### Install Apache, mod_wsgi, PostgreSQL
1. Run `sudo apt-get install apache2` to install Apache
2. Check to make sure it worked by using the public IP of the Amazon Lightsail instance as as a URL in a browser; if Apache is working correctly, a page with the title 'Apache2 Ubuntu Default Page' should load
3. Install the mod_wsgi package  along with python-dev. Run following command:
	`sudo apt-get install libapache2-mod-wsgi python-dev`
4. Make sure mod_wsgi is enabled by running `sudo a2enmod wsgi`
5. Install PostgreSQL by running `sudo apt-get install postgresql`
6. Check if no remote connections are allowed /etc/postgresql/9.5/main/pg_hba.conf file
7. run `python` to check if python is already installed. Following should appear:
	```Python 2.7.12 (default, Nov 19 2016, 06:48:10) 
	[GCC 5.4.0 20160609] on linux2
	Type "help", "copyright", "credits" or "license" for more information.
	>>> ```
8.  Create a new PostgreSQL user named `catalog` with limited permissions
9. PostgreSQL creates a Linux user with the name `postgres` during installation; switch to this user by running `sudo su - postgres`
10. Run `psql` to connect psql
11. Create a new database named catalog and create a new user named catalog in postgreSQL shell
postgres=# CREATE DATABASE catalog;
postgres=# CREATE USER catalog;
12. Set a password for user catalog
postgres=# ALTER ROLE catalog WITH PASSWORD 'catalog';
13. Exit psql by running `\q`and enter to the `grader` user by running `exit`

### Create a Linux user called `catalog` and a new PostgreSQL database
1. Create a new Linux user called `catalog`:
	- run `sudo adduser catalog`
2. Give the `catalog` user sudo permissions:
	- run `sudo visudo`
	- search for a line that looks like this: `root    ALL=(ALL:ALL) ALL`
	- add the following line below this one: `catalog    ALL=(ALL:ALL) ALL`
	- save and close the visudo file
3. While logged in as `catalog`, create a database called catalog by running `createdb catalog`
4. Run `psql` and enter `grader` user by running `exit`

### Install git and clone the catalog project
1. Install Git using `sudo apt-get install git`
2. Use `cd /var/www` to move to the `/var/www` directory
3. Create the application directory `sudo mkdir catalogapp`
4. Move inside this directory using `cd catalogapp`
5. Clone the Catalog App to the virtual machine `git clone https://github.com/amanjot1kaur/catalog.git`
6. Rename the project's name `sudo mv ./catalog ./catalogapp`
7. Run from (/var/www):
 `sudo chown -R grader:grader catalogapp/`
to change the ownership of the 'catalogapp' directory to `grader` 
8. Move to the inner directory `cd /var/www/catalogapp/catalogapp` 
9. Rename `application.py` to `__init__.py` using `sudo mv application.py __init__.py`
10. open `__init__.py` in editor with `sudo nano ./__init__.py`
	Change `app.run(host='0.0.0.0', port=8000)` line to:`app.run()`
    save file and exit
11. `sudo touch ./lotsofmenuswidusers.py` to create a populator file and paste a lot of menus to populate in the database.
12. save the file and exit.


### create a virtual environment 
1. Run to install pip:`sudo apt-get install python-pip`
2. Run to Install virtualenv:  `sudo apt-get install python-virtualenv`
3. Run `cd /var/www/catalogapp/catalogapp/` 
4. Run to give a temporary name to environment: `virtualenv venv` 
5. Run to activate environment: `. venv/bin/activate`

### Install some packages
1. In active virtual environment active, Run to Install:
	`pip install httplib2`
	`pip install requests`
	`pip install --upgrade oauth2client`
	`pip install sqlalchemy`
	`pip install flask`
	`sudo apt-get install libpq-dev` 
	`pip install psycopg2`
2. Run this command to test if the installation is successful and the app is running: `sudo python __init__.py`
-It should display “Running on http://localhost:5000/” or "Running on http://127.0.0.1:5000/". If you see this message, you have successfully configured the app. 
3. Run to deactivate the environment:`deactivate`

###  Configure & Enable a virtual host
1. Run from 'grader user': `sudo touch /etc/apache2/sites-available/catalogapp.conf` to create a file catalogapp.conf  
2. Add the following into the file:
	```
	<VirtualHost *:80>
			ServerName 13.59.147.36
			ServerAdmin YOUREMAIL@gmail.com
			WSGIScriptAlias / /var/www/catalogapp/catalogapp.wsgi
			<Directory /var/www/catalogapp/catalogapp/>
				Order allow,deny
				Allow from all
				Options -Indexes
			</Directory>
			Alias /static /var/www/catalogapp/catalogapp/static
			<Directory /var/www/catalogapp/catalogapp/static/>
				Order allow,deny
				Allow from all
				Options -Indexes
			</Directory>
			ErrorLog ${APACHE_LOG_DIR}/error.log
			LogLevel warn
			CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	```
3. Run `sudo a2ensite catalogapp` to enable the virtual host

	The following prompt will be returned:
	```
	Enabling site catalogapp.	
	To activate the new configuration, you need to run:
	  service apache2 reload
	```
4. Run `sudo service apache2 reload`
5. Run `sudo touch /var/www/catalogapp/catalogapp.wsgi` to create a file catalogapp.wsgi  
6. Add the following to the file:
	```
	activate_this = '/var/www/catalogapp/catalogapp/venv/bin/activate_this.py'
	execfile(activate_this, dict(__file__=activate_this))

	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/catalogapp/")

	from catalogapp import app as application
	application.secret_key = 'super_secret_key'
	```

7. Resart Apache: `sudo service apache2 restart`


### Update engine and third party authentication system
1. `sudo nano \_\_init__.py`,  `sudo nano .\database_setup.py`, and `sudo nano .\lotsofmenuswithusers.py` to udate engine as following:

	engine = create_engine('postgresql://catalog:catalog@localhost/catalog')

2. Add updated client_secrets.json and fb_client_secrets.json files from Google API Console and Facebook for Developers link. Update credentials of both google api consle and facebook developer site with http://13.59.147.36 for authorized JavaScript origins & Add http://ec2-13-59-147-36.compute-1.amazonaws.com/login, http://ec2-13-59-147-36.compute-1.amazonaws.com/gconnect, and http://ec2-13-59-147-36.compute-1.amazonaws.com/oauth2callback as authorized redirect URIs in google and Make http://13.59.147.36/ the site URL & http://13.59.147.36 as the Valid OAuth redirect URI in facebook.
3. `sudo nano ./__init__.py`
    -- Replace 'client_secrets.json' with complete file path '/var/www/catalogapp/catalogapp/client_secrets.json' 
    --Replace 'fb_client_secrets.json' with complete file path '/var/www/catalogapp/catalogapp/fb_client_secrets.json' 

### Disable the default Apache site
1.  run `sudo a2dissite 000-default.conf`
	The following prompt will be returned:
	```
	Site 000-default disabled.
	To activate the new configuration, you need to run:
	  service apache2 reload
	```
1. Run `sudo service apache2 reload`  
2. Run Change the ownership of the project direcotries:
	`sudo chown -R www-data:www-data catalogapp/`

### Final setup for database
1. `cd /var/www/catalogapp/catalogapp/` 
2. Activate the virtualenv by running `. venv/bin/activate`
3. Run `python lotsofmenuswidusers.py`
1. Run `deactivate`
2. Run `sudo service apache2 restart`
1. Application is running now. open up a browser and goto url `http://13.59.147.36` to see the deployed app.

## 6.  List of third-party resources used to completed this project
`http://flask.pocoo.org/docs/0.12/installation/`
`http://docs.python-guide.org/en/latest/dev/virtualenvs/`
`https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps`
`https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps`
`https://www.youtube.com/watch?v=HcwK8IWc-a8`
--Stackoverflow 
--udacity discussion forum posts:
`https://discussions.udacity.com/t/ssh-into-lightsail/246792/14`
`https://discussions.udacity.com/t/cannot-ssh-after-changing-ports/224971/11`




