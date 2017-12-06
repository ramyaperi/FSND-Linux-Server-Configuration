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
    ![alt text](https://github.com/ramyaperi/FSND-Linux-Server-Configuration/blob/master/ScreenShot.png "screenshoot")
3. Enable the Firewall. From a different terminal make sure you can log in before you close this window as firewall is checked on new login's    
    `sudo ufw enable`    
    
 ##### Deploy Flask application
 
 ###### set-up
 
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
   
  At this point the dir strcuture looks like     
   
   |----FlaskApp   
   |---------ItemCatalogAPI   
   |--------------project.py(and other files)    
   |--------------static    
   |--------------templates     
   
  ###### Setting up virtual environment     
  1. Install virtual environment    
  `sudo apt-get install python-pip`    
  `sudo pip install virtualenv `    
  2. set up virtual env    
  `sudo virtualenv venv`    
  3. Activate Virtual Env     
  `source venv/bin/activate `    
  4. Install Flask     
  `sudo pip install Flask `    
  5. Run project file and install other packages depending on the errors.     
  `sudo python project.py`    
  ImportError: No module named sqlalchemy    
  `sudo pip install flask-sqlalchemy`    
  ImportError: No module named oauth2client.client     
  `sudo pip install oauth2client`    
  ImportError: No module named requests    
  ` sudo apt-get install python-requests`     
  6. deactivate the virtual env      
  `deactivate`      
  
  ###### setting up new virtual host
  
  1. create conf file     
  `sudo nano /etc/apache2/sites-available/0000-itemcatalog.com.conf`        
  2. and paste these contents         
  `<VirtualHost *:80>       
    ServerAdmin webmaster@itemcatalog.com       
    ServerName www.itemcatalog.com        
    #alias for server to access from eternal ip give ip here        
    ServerAlias 52.59.216.91 /itemcatalog     
    #set this up so that you can check errors files       
    ErrorLog /var/www/FlaskApp/itemcatalog.com/logs/error.log       
    CustomLog /var/www/FlaskApp/itemcatalog.com/logs/access.log combined        

    WSGIDaemonProcess itemcatalogapp python-path=/var/www/FlaskApp/ItemCatalogAPI/venv/lib/python2.7/site-packages threads=5   
    WSGIProcessGroup itemcatalogapp     
    WSGIScriptAlias / /var/www/FlaskApp/ItemCatalogAPI/ItemCatalogAPI.wsgi      
    Alias /static/ /var/www/FlaskApp/ItemCatalogAPI/static      
    <Directory /var/www/FlaskApp/ItemCatalogAPI/static>     
        Order allow,deny      
        Allow from all      
    </Directory>      

</VirtualHost>`     
3. make error log dir structure       
`cd /var/www/FlaskApp       
sudo mkdir itemcatalog.com`     
4. enable virtual host          
`sudo a2ensite 0000-itemcatalog.com`      

###### create .wsgi File
1. create the file        
`cd /var/www/FlaskApp/ItemCatalogAPI        
sudo nano ItemCatalogAPI.wsgi`      
2. past these contents        
`#!/usr/bin/python      
import sys ,site,os     
#working dir      
here = os.path.dirname('/var/www/FlaskApp/ItemCatalogAPI')      
#the virtual host installation you want the application to run on       
site.addsitedir('/var/www/FlaskApp/ItemCatalogAPI/venv/lib/python2.7/site-packages')        
sys.path.insert(0,"/var/www/FlaskApp/ItemCatalogAPI")       
#from .py file (__main__ file)      
from project import app as application      
application.secret_key = 'super_secret_dummy_key'`        

######
1. Restart apache         
`sudo service apache2 restart `       
2 .To access the site from different ip     
`sudo nano \etc\hosts\` and add     
`127.0.0.1 www.itemcatalog.com`     
###### Debugging

* To check if the virtual host file syntax is fine      
 `/usr/local/apache2/bin/httpd -S`      
* If the apache failed to restart the errors can be found by running      
  `systemctl status apache2.service` and `journalctl -xe`     
* If the apache restarted fine then try curl exact ServerName given in the virtual host file      
  `curl www.itemcatalog.com`      
* If curl shows errors then find the errors in error file specified in virtual host file      
  `cat /var/www/FlaskApp/itemcatalog.com/logs/error.log`      
* Since we have a database www-data should have write access to ItemCatalogAPI dir since it creates lock files while accessing database.      
* All the paths in the .py file might have to be chnage to absolute paths.      
* Oauth and facebook auth has to be reconfigured to work on new settings.     
* If you want to access website from external IP name the conf files as follows       
`0000-itemcatalog.com.conf` instead of `itemcatalog.com.conf` the default site is the one that is 1st alpabetical order.  so to go above 000-default add another 0 to start of your name.     
  
  
