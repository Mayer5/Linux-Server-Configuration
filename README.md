# Linux-Server-Configuration
Udacity FSND Project5

## Description

Goals of the project given by our instructors from Udacity:

> You will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

The application meant to be deployed is the **Restaurant-Menu-App**, previously developed for [Project 3](https://github.com/Mayer5/Restaurant-Menu-App).

## Useful info

IP address: 52.25.246.88

Accessible SSH port: 2200.

Application URL: [http://ec2-52-25-246-88.us-west-2.compute.amazonaws.com/](http://ec2-52-25-246-88.us-west-2.compute.amazonaws.com/).

## Step by step walkthrough

### 1 - Create an AWS EC2 Instance

1. create an EC2 instance: use *Ubuntu Server 14.04 LTS (HVM), SSD Volume Type* and launch it
2. by default, the instance's security group will provide a SSH port 22
3. copy the public IP address for future use. (52.25.246.88)

### 2 - Create a new user named *udacity* and grant this user sudo permissions.

1. Log into the remote VM as *root* user through ssh: `$ ssh ubuntu@52.25.246.88`.
2. Add a new user called *udacity*: `$ sudo adduser udacity`.
3. Create a new file under the suoders directory: `$ sudo nano /etc/sudoers.d/udacity`. Fill that newly created file with the following line of text: "udacity ALL=(ALL:ALL) ALL", then save it.

### 3 - Update all currently installed packages

1. `$ sudo apt-get update`.
2. `$ sudo apt-get upgrade`.
3. Install *finger*, a utility software to check users' status: `$ apt-get install finger`.

### 4 - Configure the local timezone to UTC

1. Open time configuration dialog and set it to UTC with: `$ sudo dpkg-reconfigure tzdata`.
2. Install *ntp daemon ntpd* for a better synchronization of the server's time over the network connection: `$ sudo apt-get install ntp`.

Source: [UbuntuTime](https://help.ubuntu.com/community/UbuntuTime).

### 5 - Configure the key-based authentication for *udacity* user

1. Generate an encryption key **on remote machine** with: `$ ssh-keygen -f /home/udacity/.ssh/udacity_key.rsa`.
2. keep the public key *udacity_key.pub* in the same path and rename it to *authorized_keys*.
Then change some permissions:
	1. `$ sudo chmod 700 /home/udacity/.ssh`.
	2. `$ sudo chmod 644 /home/udacity/.ssh/authorized_keys`.
	3. Finally change the owner from *root* to *udacity*: `$ sudo chown -R udacity:udacity /home/udacity/.ssh`.
3. copy the private key *udacity_key.rsa* to *local machine* and use *puttygen.exe*(in Windows) transfer it into ppk for putty use.
4. Now you are able to log into the remote VM through ssh by using putty with the generated private key *udacity_key.ppk*

### 6 - Enforce key-based authentication
1. `$ sudo nano /etc/ssh/sshd_config`. Find the *PasswordAuthentication* line and edit it to *no*.
2. `$ sudo service ssh restart`.

### 7 - Change the SSH port from 22 to 2200
1. `$ sudo nano /etc/ssh/sshd_config`. Find the *Port* line and edit it to *2200*.
2. `$ sudo service ssh restart`.
3. add 2200 as the inbound custom TCP Rule port in AWS EC2 used Security Group.
3. Now you are able to log into the remote VM through ssh by using putty with *udacity_key.ppk* and port 2200.

Source: [Ubuntu forums](http://ubuntuforums.org/showthread.php?t=1739013).

### 8 - Disable ssh login for *root* user
1. `$ sudo nano /etc/ssh/sshd_config`. Find the *PermitRootLogin* line and edit it to *no*.
2. `$ sudo service ssh restart`.

Source: [Askubuntu](http://askubuntu.com/questions/27559/how-do-i-disable-remote-ssh-login-as-root-from-a-server).

### 9 - Configure the Uncomplicated Firewall (UFW)

Project requirements need the server to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

1. `$ sudo ufw allow 2200/tcp`.
2. `$ sudo ufw allow 80/tcp`.
3. `$ sudo ufw allow 123/udp`.
4. `$ sudo ufw enable`.
5. also add these 3 rules as Security Group inbound rules of AWS EC2

### 10 - Configure firewall to monitor for repeated unsuccessful login attempts and ban attackers

Install *fail2ban* in order to mitigate brute force attacks by users and bots alike.

1. `$ sudo apt-get update`.
2. `$ sudo apt-get install fail2ban`.
3. We need the *sendmail* package to send the alerts to the admin user: `$ sudo apt-get install sendmail`.
4. Create a file to safely customize the *fail2ban* functionality: `$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local` .
5. Open the *jail.local* and edit it: `$ sudo nano /etc/fail2ban/jail.local`. Set the *destemail* field to admin user's email address.

**Notes**: It doesn't make much sense to use *fail2ban* when the ssh key-based authentication is enforced. Though it is still useful for other things, like smtp/imap-logins.

Sources: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04), [Reddit](https://www.reddit.com/r/linuxadmin/comments/2lravs/fail2ban_does_not_detect_my_ssh_privatekey/).

### 11 - Configure cron scripts to automatically manage package updates

1. Install *unattended-upgrades* if not already installed: `$ sudo apt-get install unattended-upgrades`.
2. To enable it, do: `$ sudo dpkg-reconfigure --priority=low unattended-upgrades`.

### 12 - Install Apache, mod_wsgi

1. `$ sudo apt-get install apache2`.
2. Mod_wsgi is an Apache HTTP server mod that enables Apache to serve Flask applications. Install *mod_wsgi* with the following command: `$ sudo apt-get install libapache2-mod-wsgi python-dev`.
3. Enable *mod_wsgi*: `$ sudo a2enmod wsgi`.
3. `$ sudo service apache2 start`.

### 13 - Install Git

1. `$ sudo apt-get install git`.
2. Configure your username: `$ git config --global user.name <username>`.
3. Configure your email: `$ git config --global user.email <email>`.

### 14 - Install virtual environment, Flask and the project's dependencies

1. Install *pip*, the tool for installing Python packages: `$ sudo apt-get install python-pip`.
2. If *virtualenv* is not installed, use *pip* to install it using the following command: `$ sudo pip install virtualenv`.
3. Move to the *catalog* folder: `$ cd /var/www/catalog`. Then create a new virtual environment with the following command: `$ sudo virtualenv venv`.
4. Activate the virtual environment: `$ source venv/bin/activate`.
5. Change permissions to the virtual environment folder: `$ sudo chmod -R 777 venv`.
6. Install Flask: `$ pip install Flask`.
7. Install all the other project's dependencies: `$ pip install bleach httplib2 request oauth2client sqlalchemy'
8. Install *python-psycopg2*: `$ sudo apt-get install python-psycopg2`.

Sources: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps), [Dabapps](http://www.dabapps.com/blog/introduction-to-pip-and-virtualenv-python/).

### 15 - Create a sample flask app for testing (Optional):
[How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)


### 16 - Clone the Catalog app from Github

1. `$ cd /var/www`. Then: `$ sudo mkdir catalog`.
2. Change owner for the *catalog* folder: `$ sudo chown -R udacity:udacity catalog`.
3. Move inside that newly created folder: `$ cd /catalog` and clone the catalog repository from Github: `$ git clone https://github.com/Mayer5/Restaurant-Menu-App catalog`.
4. create a *catalog.wsgi* file in '/var/www/catalog/' folder to serve the application over the *mod_wsgi*. That file should look like this:

```python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
```
5. Some tweaks were needed to deploay the Restaurant-Menu-App, so I made a *deployment* branch which slightly differs from the *master*. Move inside the repository, `$ cd /var/www/catalog/catalog` and change branch with: `$ git checkout deployment`.

**Notes**: the *.git* folder will be inaccessible from the web without any particular setting. The only directory that can be listed in the browser will be the static folder: [static assets](http://ec2-52-25-246-88.us-west-2.compute.amazonaws.com/static/).

### 17 - Configure and enable a new virtual host

1. Create a virtual host conifg file: `$ sudo nano /etc/apache2/sites-available/catalog.conf`.
2. Paste in the following lines of code:
```
<VirtualHost *:80>
    ServerName 52.25.246.88
    ServerAlias ec2-52-25-246-88.us-west-2.compute.amazonaws.com
    ServerAdmin udacity@52.25.246.88
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
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
```
* The **WSGIDaemonProcess** line specifies what Python to use and can save you from a big headache. In this case we are explicitly saying to use the virtual environment and its packages to run the application.

3. Enable the new virtual host: `$ sudo a2ensite catalog`.

Source: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps).

### 18 - Install and configure PostgreSQL

1. Install some necessary Python packages for working with PostgreSQL: `$ sudo apt-get install libpq-dev python-dev`.
2. Install PostgreSQL: `$ sudo apt-get install postgresql postgresql-contrib`.
3. Postgres is automatically creating a new user during its installation, whose name is 'postgres'. That is a tusted user who can access the database software. So let's change the user with: `$ sudo su - postgres`, then connect to the database system with `$ psql`.
4. Create a new user called 'catalog' with his password: `# CREATE USER catalog WITH PASSWORD 'sillypassword';`.
5. Give *catalog* user the CREATEDB capability: `# ALTER USER catalog CREATEDB;`.
6. Create the 'catalog' database owned by *catalog* user: `# CREATE DATABASE catalog WITH OWNER catalog;`.
7. Connect to the database: `# \c catalog`.
8. Revoke all rights: `# REVOKE ALL ON SCHEMA public FROM public;`.
9. Lock down the permissions to only let *catalog* role create tables: `# GRANT ALL ON SCHEMA public TO catalog;`.
10. Log out from PostgreSQL: `# \q`. Then return to the *udacity* user: `$ exit`.
11. Inside the Flask application, the database connection is now performed with: 
```python
engine = create_engine('postgresql://catalog:sillypassword@localhost/catalog')
```
12. Setup the database with: `$ python /var/www/catalog/catalog/database_setup.py`.
13. Add data into the database with `$ python /var/www/catalog/catalog/initial_the_database.py`.
13. To prevent potential attacks from the outer world we double check that no remote connections to the database are allowed. Open the following file: `$ sudo nano /etc/postgresql/9.3/main/pg_hba.conf` and edit it, if necessary, to make it look like this: 
```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
Source: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps).

### 19 - Install system monitor tools

1. `$ sudo apt-get update`.
2. `$ sudo apt-get install glances`.
3. To start this system monitor program just type this from the command line: `$ glances`.
4. Type `$ glances -h` to know more about this program's options.

Source: [eHowStuff](http://www.ehowstuff.com/how-to-install-and-use-glances-system-monitor-in-ubuntu/).

### 20 - Update OAuth authorized JavaScript origins

1. Go to the project on the Developer Console: [https://console.developers.google.com/project](https://console.developers.google.com/project)
2. Navigate to APIs & Auth > Credentials > Edit Settings
3. add the hostname and piblic IP address to the Authorized JavaScript origins and host name + 'oauth2callback' to Authorized redirect URIs, e.g 'http://ec2-52-25-246-88.us-west-2.compute.amazonaws.com/oauth2callback'

### 21 - Restart Apache to launch the app
1. `$ sudo service apache2 restart`.

Note: If getting an internal server erroe, check the Apache error files:
Source: [A2 Hosting](https://www.a2hosting.com/kb/developer-corner/apache-web-server/viewing-apache-log-files)
`$ sudo tail -20 /var/log/apache2/error.log` is a good help for debugging


Special Thanks to [iliketomatoes](https://github.com/iliketomatoes/) and [stueken](https://github.com/stueken) who wrote really helpful documents.
