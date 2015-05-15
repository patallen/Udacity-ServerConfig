# Udacity-ServerConfig

IP: 52.24.224.202

### Set Up Remote Acces user - grader
1. ssh as root with udacity_rsa key
1. `adduser grader` and follow prompt
1. `adduser grader sudo` to put grader in sudo group
1. `login grader` and enter password

### Change default SSH settings
1. Change default SSH port
	- `sudo vim /etc/ssh/sshd_config`
	- Change `Port 22` to `Port 2200`
1. Turn off root remote access:
	- Change `PermitRootLogin without-password` to `PermitRootLogin no`

### Install Dependencies
- `sudo apt-get install libpq-dev apache2 libapache2-mod-wsgi python-dev python-pip git postgresql`

### Clone Repo and Setup
1. `cd /var/www`
1. `git clone https://github.com/patallen/Udacity_ItemCatalog CatalogApp`
1. `cd CatalogApp`
1. put client_secrets.json
1. `sudo vim catalogapp.wsgi` and paste:

	```python
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/CatalogApp/")

    from app import app as application
    application.secret_key = 'My Secret Key'
 	```

### Setup virtualenv

1. `sudo pip install virtualenv`
1. `sudo chown -R grader:grader venv`
1. `. venv/bin/activate`
1. `pip install --upgrade setuptools`
1. `pip install -r requirements.txt`
1. `pip install psycopg2`

### Setup Apache VirtualHost

1. `sudo vim /etc/apache2/sites-available/CatalogApp.conf`

    ```
    WSGIPythonPath /var/www/CatalogApp:/var/www/CatalogApp/venv/lib/python2.7/site-packages
    <VirtualHost *:80>
            ServerName localhost
            ServerAdmin prallen90@gmail.com
            WSGIScriptAlias / /var/www/CatalogApp/catalogapp.wsgi
            DocumentRoot /var/www/CatalogApp
            <Directory /var/www/CatalogApp/app/>
                    Order allow,deny
                    Allow from all
            </Directory>
            Alias /static /var/www/CatalogApp/app/static
            <Directory /var/www/CatalogApp/app/static/>
                    Order allow,deny
                    Allow from all
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log
            LogLevel warn
            CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```
1. `sudo a2ensite CatalogApp`
1. `sudo a2dissite 000-default`

### Setup Database

1. `sudo su -postgres`
1. `createuser catalog --interactive`
1. `Answer following questions: n, n, n`
1. `psql`
1. `alter role catalog with password 'put-password-here';`
1. `create database catalogdb;`
1. `grant all privileges on database catalogdb to catalog;`

### Config from environment variables
1. Edit app init file: `sudo vim /var/www/CatalogApp/app/__init__.py`
1. Change app.config from 'from_object' to 'from_envvar':
  - `app.config.from_envvar('CATALOGAPP_SETTINGS')`
1. Create environment variable in .wsgi file `sudo vim /var/www/CatalogApp/catalogapp.wsgi`
1. Add lines after current imports:

  ```
  import os
  os.environ['CATALOGAPP_SETTINGS']='/var/www/catalogapp_settings.py'
  ```

### Create and Seed database
1. `cd /var/www/CatalogApp`
1. `. venv/bin/activate`
1. `python fill_db.py`

### Set up Munin for monitoring:

Follow instructions in this [Digital Ocean Article](https://www.digitalocean.com/community/tutorials/how-to-install-munin-on-an-ubuntu-vps)

### Automatic Updates for Ubuntu Server
1. `sudo apt-get install unattended-upgrades`
1. Edit config : `sudo vim /etc/apt/apt.conf.d/50unattended-upgrades`
  - Uncomment line in 'allowed-origins' section `"${distro_id}:${distro_codename}-updates";`
1. Enable updates - open: `sudo vim /etc/apt/apt.conf.d/10periodic` and set:

	```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
```
Note: These instructions were found [here](https://help.ubuntu.com/lts/serverguide/automatic-updates.html)
