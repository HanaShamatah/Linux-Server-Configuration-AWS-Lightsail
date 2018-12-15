# Linux Server Configuration AWS Lightsail
In this project, Ubuntu Linux server installed and prepared to host a Restful web application (Item Catalog application), 
the fourth project in Full Stack Web Developer Nanodegree.


## Get Started

### Amazon Lightsail Instance
- Create a new ubuntu Linux server  instance on [Amazon Lightsail](https://lightsail.aws.amazon.com).
- Create an ssh key named udacitykey and download it.
- Follow the instructions to SSH into the created server.
```
Public IP: 52.59.248.6
URL: http://52.59.248.6.xip.io/
SSH Port: 22
SSH Key: udacitykey.pem
```
- Connect to the server using the public IP and key from my terminal gitbash ssh to get access to the server 
when ssh port changes as required in the next steps instead of browser-based ssh that open on a default port 22.
```
$ ssh ubuntu@52.59.248.6 -p 22 -i udacitykey.pem
```

### Secure the Server
- Update all currently installed packages and then upgrade them to install the latest versions.
```
$ sudo apt-get update
$ sudo apt-get upgrade
```
- Change the SSH port from 22 to 2200 by editing sshd_config file using sudo nano command 
and change the port number to 2200 inside it.
```
$ sudo nano /etc/ssh/sshd_config
```
- Configure UFW  
   At first, check the firewall status that by default inactive:
   ```
   $ sudo ufw status
   ```
   Denay all incoming and allow all outgoing connection requests:
   ```
   $ sudo ufw default deny incoming
   $ sudo ufw allow outgoing
   ```
   Allow incoming connections only for SSH (port: 2200). HTTP (port: 80), and NTP (port: 123) 
   and deny port 22 since it will not be used again.
   ```
   $ sudo ufw allow 2200/tcp
   $ sudo ufw allow www
   $ sudo ufw allow ntp
   $ sudo ufw deny 22
   ```
   Then, turn the firewall on, and check the status to ensure that it is enabled.
   ```
   $ sudo ufw enable
   $ sudo ufw status
   ```

- Enforce Key-based SSH authentication to get access to the server by editing sshd_config file by opening it 
using nano command and change PasswordAuthentication to no inside it. Restart the ssh to enable the 
edited configurations.
```
$ sudo nano /etc/ssh/sshd_config
$ sudo service ssh restart
```

### Give grader access
- Create a new user named grader with NUIX password: udacityuser.
```
$ sudo adduser grader
```

- Give sudo permission  
   At first, chech th permitted sudo users by listing the content of sudoers.d file, grader is not in the list.
   ```
   $ sudo ls /etc/sudoers.d
   ```
   To add grader to the permitted sudo users, create an empty grader file in sudoers.d directory 
   using sudo nano command, then, add the following piece of code to it.
   ```
   grader ALL=(ALL:ALL) ALL 
   ```
   ```
   $ sudo nano /etc/sudoers.d/grader
   ```
   Now, the grader will be in sudo users list :)

- Grader Login  
   The connection request to access the server as grader user from a new terminal will be denied 
   due to authontication purpose that restrict user to use ssh key to connect the server.
   ```
   $ ssh grader@52.59.248.6 -p 2200
   grader@52.59.248.6: Permission denied (publickey).
   ```
   Switch to grader user from ubuntu main user using its password
   ```
   $ sudo su - grader
   ```
   Or change SSH authentication to enables password authentication on to login and then return it to the 
   required configuration.

- Create an SSH key  
   For security purpose ssk key must be used to log into the server instance from the command line. 
   for this an ssh key for the grader user is generated locally using ssh-keygen tool.  
   Open a new local terminal and run ssh-keygen command, save the key in /.ssh/graderkey file in 
   my home directory with passphrase: udacityuser.
   Read the key content using cat command for graderkey.pub file, and copy it.
   ```
   $ ssh-keygen
   $ cat /.ssh/graderkey.pub
   ```
   From grader ssh opened terminal, create .ssh directory and authorized_keys file inside it. 
   Open authorized_keys file and paste the copied key content into it. Restart ssh service.
   ```
   $ mkdir .ssh
   $ touch .ssh/authorized_keys
   $ nano .ssh/authorized_keys
   $ sudo service ssh restart
   ```
   Then, connect grader to the server using the generated key:
   ```
   $ ssh grader@52.59.248.6 -p 2200 -i ~/.ssh/graderkey
   ```
   Set the permission to read, write, and execute of .ssh file only to the grader user, and the permission of authorized_keys 
   file to read and write to the grader user and only read for others.
   ```
   sudo chmod 700 /.ssh
   sudo chmod 644 /.ssh/authorized_keys
   ```

- Disable root login by setting PermitRootLogin to no inside the sshd_config file and then restart ssh service.


### Prepare to Deploy the Project
- Configure the local timezone to UTC using the following command, then, check the current timezone.
```
$ sudo timedatectl set-timezone UTC 
$ timedatectl status
```

- Install PostgreSQL database package using:
```
$ sudo apt-get install postgresql postgresql-contrib
```
   Check the connections to only local host 127.0.0.0 to do not allow any remote connections 
   by reading pg_hba.conf configuration file:
   ```
   $ sudo nano /etc/postgresql/9.5/main/pg_hba.conf
   ```
   Create a new postgres user named catalog with password: catalog, 
   and limited permissions to the application database.
   ```
   $ sudo -u postgres createuser -P catalog
   ```
   Create a new database catalog owned to catalog postgres user.
   ```
   $ sudo -u postgres createdb -O catalog catalog
   ```
   Create a new linux user with the same name as the Postgres role (user) and database 
   in order to log in with ident based authentication with password: catalog.
   ```
   $ sudo adduser catalog
   ```
   Then, access the catalog database to give catalog user permission to use catalog database.
   ```
   $ sudo -i -u catalog psql
   catalog=> GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
   catalog=> GRANT CONNECT ON DATABASE catalog To catalog;
   ```

- Install git and configure it with my details by creating a configuration file named .gitconfig 
and insert the details into it. Then list the configured details to insure that it is inserted correctly.
```
$ sudo apt-get install git
$ nano ~/.gitconfig
```
```
[user]
    name = Hana Shamata
    email = hanashamata@gmail.com
```
```
$ sudo git config --list
```

- Installed packages:
```
$ sudo apt-get install apache2  # to install Apache
$ sudo apt-get install libapache2-mod-wsgi  # to install an application handler - mod_wsgi 
$ sudo apt-get install python-dev
$ sudo apt install python-pip  # pip used ti install python packages
$ sudo pip2 install Flask
$ sudo pip2 install oauth2client
$ sudo pip2 install sqlalchemy
$ sudo pip2 install psycopg2 
$ sudo pip2 install requests
```
Test apache is installed and working by accessing the page with its public ip address: http://52.59.248.6/

### Deploy the Item Catalog project
- Enable the installed mod_wsgi package and restart apache.
```
$ sudo a2enmod wsgi
$ sudo service apache2 restart
```

- Create the application directory (catalog) in /var/www directory and cd to it.
```
$ sudo mkdir catalog
$ cd catalog
```

- Clone th application repository [Item Catalog project](https://github.com/HanaShamatah/catalog) (the forth project in Udacity nanodegree) into catalog directory, then, cd to cloned diretory catalog, and rename the application.py to __init__.py
This version of the project is an edited version of original project repository [Item Catalog](https://github.com/HanaShamatah/Item-Catalog)
```
$ sudo git clone https://github.com/HanaShamatah/catalog.git
$ cd catalog
$ sudo mv application.py __init__.py 
```
To make .git inaccessible create .htaccess file in your project root catalog/catalog then 
add the following piece of code to it and save it.
```
RedirectMatch 404 /\.git
```
```
$ sudo nano .htaccess 
```

- Update the Google OAuth credintials by adding [http://52.59.248.6.xip.io](http://52.59.248.6.xip.io/) 
to Authorized Javascript origins and update Authorized redirect URIs with:
```
http://52.59.248.6.xip.io/gconnect
http://52.59.248.6.xip.io/login
```
Note that a DNS name xio.io is used from [xio.io](http://xip.io/) service.
Download client secret.json file to update the cooresponding file in catalog cloned project using nano to edit it.
```
$ sudo nano client_secrets.json 
```

- Edit __init__.py application using sudo nano command to edit the path of open functions to full path 
'/var/www/catalog/catalog/client_secrets.json' instead of 'client_secrets.json'
and database engine to connect to postgres created database ('postgresql://catalog:catalog@localhost/catalog').
```
$ sudo nano __init__.py
```

- Edit the database engine also in database setup file [catalog_database.py](https://github.com/HanaShamatah/catalog/blob/master/catalog_database.py) and fill data file [filldatabase_clothes.py](https://github.com/HanaShamatah/catalog/blob/master/filldatabase_clothes.py)
```
$ sudo nano catalog_database.py
$ sudo nano filldatabase_clothes.py
```

- Create catalog.wsgi configuration file in main application diretory (/var/www/catalog) and 
add the following piece of code then save it:
```
$ sudo nano /var/www/catalog/catalog.wsgi
```
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")


from catalog import app as application
application.secret_key = 'super_secret_key'
```

- Configure and Enable a New Virtual Host by creating catalog.conf file using nano then paste the 
following code and save it:
```
$ sudo nano /etc/apache2/sites-available/catalog.conf 
```
```
<VirtualHost *:80>
                ServerName 52.59.248.6
                ServerAdmin hanashamata@gmail.com.com
                WSGIScriptAlias / /var/www/catalog/catalog.wsgi
                <Directory /var/www/FlaskApp/FlaskApp/>
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
Disable the default virtual host and Enable the catalog.conf virtual host, then restart apache.
```
$ sudo a2dissite 000-default.conf
$ sudo a2ensite catalog.conf
$ sudo service apache2 restart
```

### Run the Application
Run database setup file, database fill data file, the run init.py application.
```
$ python database_create.py  # to build database tables
$ python filldatabase_clothes.py  # to fill data into database tables
$ python __init__.py  # to run th application
```

## Exrernal Resources
- [DigitalOcean tutorial](https://www.digitalocean.com/community/tutorials/)  
   [Deploy a Flask Application on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)  
   [Install the Apache Web Server on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-16-04)  
   [PostgreSQL on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04)  
   [Install Git on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-git-on-ubuntu-16-04)

- [Ubuntu Documentation](https://help.ubuntu.com/)
- [Amazon Lightsail Documentation](https://docs.aws.amazon.com/lightsail/index.html#lang/en_us)


## License
The content of this repository is licensed under an [MIT](https://choosealicense.com/licenses/mit/).
