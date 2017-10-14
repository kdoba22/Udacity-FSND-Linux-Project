### Udacity FSND Linux Server project
### Necessary Info to run and grade project

    Doba-Udacity-Linux-Project
    Private IP: 172.26.9.77


    Public IP: 13.59.130.140
    grader password is 'grader'
    grader passphrase is 'hello'
    SSH port- 2200
    log in with ssh grader@13.59.130.140 -p 2200 -i ~/.ssh/id_rsa
    passphrase for grader is 'hello'
    password for grader is 'grader'

    http://13.59.130.140
    http://ec2-13-59-130-140.us-east-2.compute.amazonaws.com/



### Setup VM and login using ssh
1.  create new amazon lightsail server
2.  note private and public ip address, see above
3.  grab default private key and download it.
4.  from a console on your local move the private key file into ~/.ssh folder
    before the next step, rename the lightsail private key to privatekey.pem
    mv ~/Downloads/privatekey.pem ~/.ssh

5.  change permission to r/w on the file
    chmod 600 ~/.ssh/privatekey.pem
6.  log into your new server as root
    ssh -i ~/.ssh/privatekey.pem  ubuntu@13.59.130.140   (public ip)


### Update available package lists
1.  sudo apt-get update     - get list of available updates
2.  sudo apt-get upgrade    - make actual updates
3.  sudo apt-get autoremove - see if there is anything we can remove

### Create new user "grader"
1.  sudo adduser grader     - create user grader
                            - password: grader
2.  cat /etc/passwd         - confirm user created  

#### give grader sudo permission

1.  Use the usermod command to add the user to the sudo group.
    sudo usermod -aG sudo grader, https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart

    use the 'pwd' command and verify your in /home/ubuntu
    If your not, get there before continuing.

#### Set up Key Based authentication
1.  Generate key pair on your local machine
    ssh-keygen                 passphrase : 'hello'

    Your identification has been saved in /Users/keithdoba/.ssh/id_rsa
    Your public key has been saved in /Users/keithdoba/.ssh/id_rsa.pub

2.  Place public key on the remote server
    a.  make sure your logged into server as grader
        from ubuntu login type 'su - grader'
        confirm your still in /home/grader, if not get there

    b.  make new directory
        mkdir .ssh
        touch .ssh/authorized_keys
    c.  on local machine, display contents of .ssh/id_rsa.pub
        cat .ssh/id_rsa.pub
    d.  Copy the contents of id_rsa.pub to the .ssh/authorized_keys file on the server
        sudo nano .ssh/authorized_keys   [save file and exit]

3.  Set up file permissions, from grader
    sudo chmod 700 .ssh
    sudo chmod 644 .ssh/authorized_keys

4.  log is as grader using key pairs
    ssh grader@13.59.130.140 -p 22 -i ~/.ssh/id_rsa


#### Change SSH port from 22 to 2200
1.  edit /etc/ssh/sshd_config by:
    sudo nano /etc/ssh/sshd_config
    change Port 22 to Port 2200

2.  You must change the port to 2200 in the lightsail web page as well.

3.  After changing this port, restart the server
    service sshd restart  - you will be asked for credentials
                            choose grader and enter password, grader
refer to:  https://www.liquidweb.com/kb/changing-the-ssh-port/



### logging in as grader
1.  ssh grader@13.59.130.140 -p 2200 -i ~/.ssh/id_rsa

### Disable remote login of root
1.  sudo nano /etc/ssh/sshd_config
2.  Change PermitRootLogin from prohibit-password to no                

### Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200),
### HTTP (port 80), and NTP (port 123).

1.  sudo ufw status - check status of firewall (inactive)
2.  sudo ufw default deny incoming  - set to initially deny all incoming
3.  sudo ufw default allow outgoing - sets default to allow outgoing traffic
4.  sudo ufw allow 2200/tcp - connections for SSH
5.  sudo ufw allow 80/tcp - connections for https
6.  sudo ufw allow 123/tcp - connections for NTP
7.  sudo ufw enable  - enable Firewall
8.  sudo ufw status - check status one last time:

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
123/tcp                    ALLOW       Anywhere                  
2200/tcp (v6)              ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
123/tcp (v6)               ALLOW       Anywhere (v6)

### configure local timezone to UTC
1.  sudo timedatectl set-timezone Etc/UTC - sets timezone to UTC - https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt

2. date +%Z - used to check timezone
    UTC

### install and configure Apache to serve a Python mod_wsgi application.
    https://www.linode.com/docs/web-servers/apache/apache-and-modwsgi-on-ubuntu-14-04-precise-pangolin
    http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/
    http://modwsgi.readthedocs.io/en/develop/user-guides/quick-configuration-guide.html

1.  sudo apt-get install apache2 - install Apache2
2.  check if Apache configured by going to
     http://ec2-13-59-130-140.us-east-2.compute.amazonaws.com
      The Apache page will appear if so.

3.  sudo apt-get install libapache2-mod-wsgi - set up mod_wsgi application
4.  sudo nano /etc/apache2/sites-enabled/000-default.conf -
     configure Apache to handle requests using the WSGI module.
     For now, add the following line at the end of the <VirtualHost *:80> block, right before the closing </VirtualHost> line: WSGIScriptAlias / /var/www/html/myapp.wsgi*    

     remember to save

5.  Finally, restart Apache with the sudo apache2ctl restart command
6.  to quickly test if you have your Apache configuration correct you’ll write a very basic WSGI     application.
7.  sudo nano /var/www/html/myapp.wsgi - open location to add the following script

      def application(environ, start_response):
      status = '200 OK'
      output = 'Please type 13.59.130.140 into your browser and hit enter'

      response_headers = [('Content-type', 'text/plain'), ('Content-Length', str(len(output)))]
      start_response(status, response_headers)

      return [output]
8.  refresh your browser, http://ec2-13-59-130-140.us-east-2.compute.amazonaws.com/
    you should see 'Hello Udacity'

### Install PostgreSQL - https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04
                         http://www.stuartellis.name/articles/postgresql-setup/
                         https://www.postgresql.org/docs/8.0/static/sql-createuser.html
                         https://wiki.postgresql.org/wiki/First_steps
                         https://askubuntu.com/questions/413585/postgres-password-authentication-fails
                         https://www.postgresql.org/docs/9.0/static/sql-alterrole.html
                         https://stackoverflow.com/questions/36232458/revoke-access-to-postgres-database-for-a-role

1. sudo apt-get install postgresql postgresql-contrib - installs POSTgresql with additional utilities
2. server setup, sudo -u postgres psql postgres
3. set password, \password postgres password    - password is password

### Configuring the PostgreSQL Server and create new db user
1.  sudo -u postgres   (sudo -u postgres psql postgres)
2.  psql postgress
3.  Create user
     CREATE USER catalog WITH PASSWORD 'password';
4.  Confirm user created - \du  - this command also shows users permissions...currently no permissions
5.  To exit the PostgreSQL shell, type: \q

NOTE;  To get a list of all the user roles type \h CREATE ROLE

### Create a new database user named *catalog* that has limited permissions
1. sudo -u postgres psql - First connect/login
2.  Update permissions of catalog user
      ALTER ROLE catalog WITH LOGIN;
      ALTER USER catalog CREATEDB;

3.  create DB
      CREATE DATABASE catalog WITH OWNER catalog;
4.  log into DB
      \c catalog
5.  revoke rights as we are suppose to have limited rights
      REVOKE ALL ON SCHEMA public FROM public;
6.  grant access to catalog user      
      GRANT ALL ON SCHEMA public TO catalog;
      \q to exit
7.  Restart
      sudo service postgresql restart



### Install git, clone and setup your Catalog App project.
  https://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible
  https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup
  https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

1.  sudo apt-get install git
2.  git config --global user.name "Keith Doba"
3.  git config --global user.email kdoba22@gmail.com

### Creating a Flask App
    https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
1.  cd /var/www - change directories
2.  sudo mkdir FlaskApp - create an application directory
3.  cd FlaskApp - enter new directory
4.  clone the respository:
      sudo git clone https://github.com/kdoba22/Udacity-FSND-project3-item-catalog.git
5.  rename project to align with https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

6.  sudo mv ./Udacity-FSND-project3-item-catalog ./FlaskApp
7.  cd FlaskApp - move into the new directory
8.  sudo mv -v /var/www/FlaskApp/FlaskApp/catalog/* /var/www/FlaskApp/FlaskApp - remove catalog folder and move all up one level
9.  sudo mv views.py __init__.py - create the __init__.py file that will contain the flask application logic.
10.  sudo mkdir FlaskApp - create another FlaskApp directory   #############I think remove
11. cd FlaskApp - move inside again
12.  sudo nano /etc/apache2/sites-available/FlaskApp.conf - create FlaskApp.conf to add the following:

      <VirtualHost *:80>
      		ServerName mywebsite.com
      		ServerAdmin admin@mywebsite.com
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


13.* sudo a2ensite FlaskApp - enable the virtual host
  Create the .wsgi File
  cd /var/www/FlaskApp
  sudo nano flaskapp.wsgi

  add the following lines of code:

  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/FlaskApp/")
  from FlaskApp import app as application
  application.secret_key = 'super_secret_key'

14.  restart Apache


15.  edit DB to point to postgresql DB
      /var/www/FlaskApp/FlaskApp$ sudo nano database_setup.py
      change engine = create_engine('sqlite:///catalogmenu.db') to
      engine = create_engine('postgresql://catalog:password@localhost/catalog')

      make the same change to __init__.py




### install flask/pip/etc, https://www.saltycrane.com/blog/2010/02/how-install-pip-ubuntu/
1.  sudo apt-get install python-pip python-dev build-essential
2.  sudo pip install --upgrade pip
3.  sudo pip install --upgrade virtualenv
4.  sudo pip install Flask
5.  sudo pip install sqlalchemy
6.  sudo pip install oauth2client
7.  sudo pip install requests
8.  sudo pip install psycopg2

9.  sudo apache2ctl restart - restart Apache
10.  edit DB to point to postgresql DB
      /var/www/html/catalog/Udacity-FSND-project3-item-catalog/catalog$ sudo nano database_setup.py
      change engine = create_engine('sqlite:///catalogmenu.db') to
      engine = create_engine('postgresql://catalog:password@localhost/catalog')

      make the same change to views.py


Note: as I ran into an error, I installed another package

11.  sudo python database_setup.py - create DB schema

      After all packages installed and all changes made to the code, I had a few more code issues I needed to troubleshoot.
12.  more changes needed to the sqlite statements I altered earlier.

3.  I had trouble with [Sun Oct 01 09:03:32.766631 2017] [wsgi:error] [pid 32408:tid 140574110631680] [client 108.235.40.135:50131] IOError: [Errno 2] No such file or directory: 'client_secrets.json', referer: http://13.59.130.140/

        found fix here, https://stackoverflow.com/questions/22282760/filenotfounderror-errno-2-no-such-file-or-directory
        I just used the entire path, open(r'/var/www/FlaskApp/FlaskApp/client_secrets.json', 'r').read())['web']['client_id']
### After installation complete I ran 'python lotsofmenues.py' and started the application, 'python __init__.py'

Note:
    All this worked, but it would only pick up my site from what was listed in the ServerName
    location in the FlaskApp.conf.  I wanted it to run from either the IP or the AWS site name
    To do this run the following
    1.  sudo a2dissite 000-default     disable the place holder site
    2.  sudo /etc/init.d/apache2 reload   restart Apache
    3.  sudo apachectl restart
