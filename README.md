# itemcat

1. Create a new user grader:
$ sudo adduser grader 
$ touch /etc/sudoers.d/grader
$ sudo nano /etc/sudoers.d/grader
key in: grader ALL=(ALL:ALL) ALL
save (ctrl + o) and quit (ctrl + x)

2. Update installed packages:
$ sudo apt-get update
$ sudo apt-get update

3. Generate public key in local machine in ~/.ssh folder and paste in the vm:
$ ssh-keygen
- copy the content of the public key generated (.pub)
- In the virtual machine:
$ su - grader
$ mkdir .ssh
$ touch .ssh/authorized_keys
$ sudo nano .ssh/authorized_keys
- paste the content of the public key from the local machine generated and save.
$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys
- reload ssh:
$ sudo ssh restart
You can login with ssh -v -i publickeyname.pem grader@13.228.49.137

4. change ssh port from 22 to 2200
$ sudo nano /etc/ssh/sshd_config
change port from 22 to 2200
save, quit and restart ssh service
$ sudo service ssh restart


login again using port 2200
ssh -v -i publickeyname.pem grader@13.228.49.137 -p 2200

5. Configure uncomplicated firewall for port 80, 2200, 123
$ sudo ufw allow 2200/tcp
$ sudo ufw allow 80/tcp
$ sudo ufw allow 123/udp
$ sudo ufw enable 

6. Install and configure apache, mod_wsgi, postgresql
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi python-dev
$ sudo a2enmod wsgi
$ sudo service apache2 start
$ sudo su - postgres
$ psql
create database and user catalog
postgres=# CREATE DATABASE catalog;
postgres=# CREATE USER catalog;
postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
postgres=# \q
exit

7. Install git and clone the project 
$ sudo apt-get install git
$ cd /var/www 
$ sudo mkdir FlaskApp
$ cd FlaskApp
$ git clone https://github.com/flow12345/itemcat
create catalog.wsgi file and paste below inside the file:
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")
from dessert import app as application
application.secret_key = 'your secret key'

$ sudo apt-get install python-pip
$ sudo pip install -r requirements.txt
$ sudo apt-get -qqy install postgresql python-psycopg2
$ sudo python database_setup.py

8. Configure virtual host
$ sudo nano /etc/apache2/sites-available/FlaskApp.conf
Paste in the file:
<VirtualHost *:80>
        ServerName 13.228.49.137
        ServerAdmin nakmalsam@gmail.com
        WSGIScriptAlias / /var/www/FlaskApp/catalog.wsgi
        <Directory /var/www/FlaskApp/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/catalog/static
        <Directory /var/www/catalog/static/>
                Order allow,deny
                Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

enable the virtual host:
$ sudo a2ensite catalog

9. restart apache service
$ sudo service apache2 restart










