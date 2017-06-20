# Udacity-Linux-Server-Configuration

### About

Take a Linux server on a virtual machine and host your web application, including installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

- IP address: 54.93.91.46

- Application URL: ec2-54-93-91-46.eu-central-1.compute.amazonaws.com

### Upgrade software

$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get autoremove

I then installed the application called Finger with the command:

$ sudo apt-get install finger

to check informations about a user.

### Create new users and configuring ssh

$ sudo adduser grader
$ sudo cp /etc/sudoers.d/vagrant /etc/sudoers.d/grader

and I changed the following text:
grader ALL=(ALL) NOPASSWD:ALL

I then used the keygen application with the command:
ssh-keygen

I created and changed the configuration of the athorized_key file

$ sudo mkdir /home/grader/.ssh
$ sudo touch /home/grader/.ssh/authorized_keys
$ sudo chown grader /home/grader/.ssh
$ sudo chown grader /home/grader/.ssh/authorized_keys
$ sudo chmod 700 /home/grader/.ssh
$ sudo chmod 644 /home/grader/.ssh/authorized_keys

Then I added the pared key into the authorized_keys file. 

Finally, I opened the $ sudo nano /etc/ssh/sshd_config file and set the following configuration settings:
Port 2200
PermitRootLogin no
PasswordAuthentication no

After this I exited the root login session and logged back into the server with my new user account and public key.

### How to Login as “Grader”

ssh grader@ec2-54-93-91-46.eu-central-1.compute.amazonaws.com -p 2200 -i ~/.ssh/udacity.pem

### Set the time zone

To set the time zone I ran the command:

$ sudo dpkg-reconfigure tzdata

This opened a dialog. From the menu I selected 'None of the above' and then 'UTC.'

### Set up a firewall

Next I configured a firewall to only allow incoming connections on ports 80, 123 and 2200:

### Instaled and configured Apache and mod-wsgi

Next I installed git, apache and mod-wsgi:

sudo apt-get install apache2 libapache2-mod-wsgi git

### Installed python and other requierment for my application to run

sudo apt-get -qqy install postgresql python-psycopg2
sudo apt-get -qqy install python-flask python-sqlalchemy
sudo apt-get -qqy install python-pip

sudo pip install bleach
sudo pip install oauth2client requests httplib2 redis passlib itsdangerous flask-httpauth

### Chanaged directory to
cd /var/www/

#####Copy Catalog App from Github to /var/www/catalog

sudo git clone https://github.com/irenesacchi/Catalog-App.git ./catalog

##### Create WSGI File for Apache (How to start the App)
sudo vim /var/www/catalog/catalog.wsgi


```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from project import app as application
application.secret_key = 'super-secret-key'
```


### Change Default Apache Virtual Host to point to the catalog app
cd /etc/apache2/sites-available/
sudo vim 000-default.conf

```
<VirtualHost *:80>
# The ServerName directive sets the request scheme, hostname and port that
# the server uses to identify itself. This is used when creating
# redirection URLs. In the context of virtual hosts, the ServerName
# specifies what hostname must appear in the request's Host: header to
# match this virtual host. For the default virtual host (this file) this
# value is not decisive as it is used as a last resort host regardless.
# However, you must set it for any further virtual host explicitly.
#ServerName www.example.com

ServerAdmin webmaster@localhost
DocumentRoot /var/www/catalog

WSGIScriptAlias / /var/www/catalog/catalog.wsgi

<Directory /var/www/catalog/>
Order allow,deny
Allow from all
</Directory>

# Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
# error, crit, alert, emerg.
# It is also possible to configure the loglevel for particular
# modules, e.g.
#LogLevel info ssl:warn

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined

# For most configuration files from conf-available/, which are
# enabled or disabled at a global level, it is possible to
# include a line for only one particular virtual host. For example the
# following line enables the CGI configuration for this host only
# after it has been globally disabled with "a2disconf".
#Include conf-available/serve-cgi-bin.conf
</VirtualHost>
```

### Reload Apache Config
sudo service apache reload


