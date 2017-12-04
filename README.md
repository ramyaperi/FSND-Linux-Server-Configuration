# FSND-Linux-Server-Configuration

Deploy Flask application on Ubuntu  hosted on apache using amazon aws lightsail.

IP : http://52.59.216.91/
Port : 80

### softwares installed 

Flask==0.12.2
Flask-SQLAlchemy==2.3.2
httplib2==0.10.3
oauth2client==4.1.2
pkg-resources==0.0.0
psycopg2==2.8.dev0
requests==2.18.4
rsa==3.4.2
SQLAlchemy==1.1.15
urllib3==1.22

### summary of configurations 

###### Basic set up 


1. Create a Amazon developr account.
2. Create a server with ubuntu.
3. Download the default private key from accounts page of AWS light sail.
4. Access the server from terminal using 
  `ssh -i locationToPEMFile user@Ipaddress -p port`
5. create user grader `sudo adduser grader`
6. Grand grader sudo permissions 
    `sudo ls /etc/sudoers.d`
    create a file grader and add following to give sudo permission with out password prompt.
    `grader ALL=(ALL:ALL) ALL`
7. update the software 
  ` sudo apt-get update
    sudo apt-get upgrade`
8. set timezone
   `sudo timedatectl set-timezone UTC`
    
###### Key-based SSH authentication

1. On your local machine generate private key pair 
`ssh-keygen`
2. Copy contents of .pub file 
3. Paste the contents into a authorized_keys file in .ssh on server 
4. Change the permissions 
    `sudo chmod 700 .ssh
     sudo chmod 644 .ssh/authorized_keys`
5. If the permissions on the local machine are not set it will throw an warning and not let you sign in. 

###### Firewall set up 

1. Only allow connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
` sudo ufw default deny incoming
  sudo ufw default allow outgoing
  sudo ufw allow 80/tcp
  sudo ufw allow 2200/tcp
  sudo ufw allow 123/udp` 
  
2. On the Amazon lightsail webpage go to networking and add port 2200. 
3. Enable the Firewall. From a different terminal make sure you can log in before you close this window as firewall is checked on new login's  
    `sudo ufw enable`
    
 ##### Deploy Flask application
 
 1. if you want to use python 2 
 `sudo apt-get install libapache2-mod-wsgi`
 for python 3 
 `sudo apt-get install libapache2-mod-wsgi-py3`
 2. install apache 
 `sudo apt-get install apache2`
 3. install pythonsetup tools
 `sudo apt-get install python-setuptools`
 4. enable wsgi 
 `sudo a2enmod wsgi `
 5. open  and create required structure
  `cd /var/www `
  `sudo mkdir FlaskApp`
  `cd FlaskApp`
 6. install git 
  `sudo apt-get install git`
 7. Clone from git , if its not a private repository login not required
   `git clone directory_link`
   
  
