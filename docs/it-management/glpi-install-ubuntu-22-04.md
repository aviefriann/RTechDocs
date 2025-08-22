_You can find these steps on the official GLPI website. Please check the following link:_ [Install GLPI on Ubuntu](https://help.glpi-project.org/tutorials/procedures/install_glpi)

For this installation, we need

- A Linux Ubuntu Server 22.04 LTS

- A Web server - Apache

- A code interpreter - PHP

- A Data Base Server Management Platform - MariaDB in this case

We are separating this process in 6 steps:

1. Installing components

2. Database configuration

3. Preparing files and folders to install GLPI

4. Giving correct folder and files permission on linux to install GLPI

5. Configuring web server and PHP

6. Starting web installation

## 1 - Installing the components

Before we start, make sure your server is up to date

```
apt update && apt upgrade
```

For this post, we are using Apache 2, MariaDB Server, PHP and its respective extensions. If your operating system and repositories are updated, the latest stable version of the extensions are already the ones downloaded.

```
apt install -y apache2 php php-{apcu,cli,common,curl,gd,imap,ldap,mysql,xmlrpc,xml,mbstring,bcmath,intl,zip,redis,bz2} libapache2-mod-php php-soap php-cas
apt install -y mariadb-server
```

- After all the components are installed, we need to follow the steps 2 - 6.

## 2 - Database Configuration

MariaDB, by default is provided without a default password set to the root user and with some default settings that need to be correctly configured.

### Secure MariaDB Installation

```
mysql_secure_installation
```

**Minimum recommendation**

- Change the root password

- Remove anonymous users

- Disallow root login remotely

- Remove test database

- Reload privilege tables

Furthermore, since GLPI is a global ITSM tool which may be used by people from all around the world at the same time, you would like to activate the possibility to GLPI database service user to read timezone information from your default mysql database.

```
mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql mysql
```

### Create a user and database dedicated to GLPI

```
mysql -uroot -pmysql
CREATE DATABASE glpi;
CREATE USER 'glpi'@'localhost' IDENTIFIED BY 'yourstrongpassword';
GRANT ALL PRIVILEGES ON glpi.* TO 'glpi'@'localhost';
GRANT SELECT ON `mysql`.`time_zone_name` TO 'glpi'@'localhost';
FLUSH PRIVILEGES;
```

## 3 - Preparing files to Install GLPI

After you installed the components and created a first database and service user to receive GLPI folders, you will download the \*.tgz latest version of GLPI and store it on apache main root folder.

You may always find the latest stable release on [https://glpi-project.org/download](https://glpi-project.org/download)

```
cd /var/www/html
wget https://github.com/glpi-project/glpi/releases/download/10.0.19/glpi-10.0.19.tgz
tar -xvzf glpi-10.0.19.tgz
```

### Filesystem Hierarchy Standard Breakdown

In this scenario we are storing GLPI information in different folders and following the FHS where, usually.

- **/etc/glpi** : for the files of configuration of GLPI (config_db.php, config_db_slave.php) ;

- **/var/www/html/glpi** : for the source code of GLPI (in reading only), served by Apache;

- **/var/lib/glpi** : for the variable files of GLPI (session, uploaded documents, cache, cron, plugins, …);

- **/var/log/glpi** : for the log files of GLPI.

To make sure GLPI will find those files, we need to indicate in two different files where these folders are on the system:

### The Downstream file

The `downstream.php` file is responsible for instructing GLPI application where the `GLPI_CONFIG_DIR` - the configuration directory of GLPI - is stored. Remember, we must indicate `/etc/glpi` as the new folder for configuration files. GLPI understands that a file called `downstream.php` inside the `inc` folder has these instructions.

- Create the `downstream.php` file

```
vim /var/www/html/glpi/inc/downstream.php
```

- Declare the new config file folder - you can insert this content in this file you have created

```
<?php
define('GLPI_CONFIG_DIR', '/etc/glpi/');
if (file_exists(GLPI_CONFIG_DIR . '/local_define.php')) {
require_once GLPI_CONFIG_DIR . '/local_define.php';
}
```

- Now you may move the folders from its current directory to the new directories:

```
mv /var/www/html/glpi/config /etc/glpi
mv /var/www/html/glpi/files /var/lib/glpi
mv /var/lib/glpi/_log /var/log/glpi
```

After you declare the new **`GLPI_CONFIG_DIR`** with the **`downstream.php`**, navigate to this new directory **`/etc/glpi`** and create a new file called local_define.php. This file is reponsible for instructing GLPI where the other directories are stored.

We are changing the documents folder ( **`files`** ) and the logs folder ( **`files/_log`** ) to their new directory.

- Create the `local_define.php` file

```
vim /etc/glpi/local_define.php
```

- Paste the following in this file

```
<?php
define('GLPI_VAR_DIR', '/var/lib/glpi');
define('GLPI_DOC_DIR', GLPI_VAR_DIR);
define('GLPI_CACHE_DIR', GLPI_VAR_DIR . '/_cache');
define('GLPI_CRON_DIR', GLPI_VAR_DIR . '/_cron');
define('GLPI_GRAPH_DIR', GLPI_VAR_DIR . '/_graphs');
define('GLPI_LOCAL_I18N_DIR', GLPI_VAR_DIR . '/_locales');
define('GLPI_LOCK_DIR', GLPI_VAR_DIR . '/_lock');
define('GLPI_PICTURE_DIR', GLPI_VAR_DIR . '/_pictures');
define('GLPI_PLUGIN_DOC_DIR', GLPI_VAR_DIR . '/_plugins');
define('GLPI_RSS_DIR', GLPI_VAR_DIR . '/_rss');
define('GLPI_SESSION_DIR', GLPI_VAR_DIR . '/_sessions');
define('GLPI_TMP_DIR', GLPI_VAR_DIR . '/_tmp');
define('GLPI_UPLOAD_DIR', GLPI_VAR_DIR . '/_uploads');
define('GLPI_INVENTORY_DIR', GLPI_VAR_DIR . '/_inventories');
define('GLPI_THEMES_DIR', GLPI_VAR_DIR . '/_themes');
define('GLPI_LOG_DIR', '/var/log/glpi');
```

## 4 - Folder and File Permissions

Here is a suggestion of permissions for your GLPI installation

```
chown root:root /var/www/html/glpi/ -R
chown www-data:www-data /etc/glpi -R
chown www-data:www-data /var/lib/glpi -R
chown www-data:www-data /var/log/glpi -R
chown www-data:www-data /var/www/html/glpi/marketplace -Rf
find /var/www/html/glpi/ -type f -exec chmod 0644 {} \;
find /var/www/html/glpi/ -type d -exec chmod 0755 {} \;
find /etc/glpi -type f -exec chmod 0644 {} \;
find /etc/glpi -type d -exec chmod 0755 {} \;
find /var/lib/glpi -type f -exec chmod 0644 {} \;
find /var/lib/glpi -type d -exec chmod 0755 {} \;
find /var/log/glpi -type f -exec chmod 0644 {} \;
find /var/log/glpi -type d -exec chmod 0755 {} \;
```

## 5 - Configure the Web Server

For GLPI to run smoothly, without the need of complex URLs, we recomend you use a DNS name for your server and create a Virtual Host to forward all the requests coming to your instance looking for this previously created DNS entry to the correct path in your Apache configuration. More information about web server configuration [can be found here](https://glpi-install.readthedocs.io/en/latest/prerequisites.html#web-server)

### How to create a VirtualHost dedicated to GLPI?

- Create a file on `/etc/apache2/sites-available/glpi.conf`

```
/etc/apache2/sites-available/glpi.conf
```

If you need, you can change the file name to your webserver standards.

- In this file, you will add the following content:

```
# Start of the VirtualHost configuration for port 80

<VirtualHost *:80>
    ServerName yourglpi.yourdomain.com
    # Specify the server's hostname
    DocumentRoot /var/www/html/glpi/public
    # The directory where the website's files are located
    # Start of a Directory directive for the website's directory
    <Directory /var/www/html/glpi/public>
        Require all granted
        # Allow all access to this directory
        RewriteEngine On
        # Enable the Apache rewrite engine
        # Ensure authorization headers are passed to PHP.
        # Some Apache configurations may filter them and break usage of API, CalDAV, ...
        RewriteCond %{HTTP:Authorization} ^(.+)$
        RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
        # Redirect all requests to GLPI router, unless the file exists.
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteRule ^(.*)$ index.php [QSA,L]
    </Directory>
    # End of the Directory directive for /var/www/glpi/public
</VirtualHost>

# End of the VirtualHost configuration for port 80
```

The variables can be changed to your standards, file locations or folder names

- **ServerName** if you have a public URL, you can type it here

- **DocumentRoot** if you will store GLPI in a different page, change it too.

After the Virtual Host file is created you should disable the default apache site configuration, enable the rewrite module and reload the new vhost file.

`a2dissite 000-default.conf` # Disable default apache site
`a2enmod rewrite` # enable the rewrite module
`a2ensite glpi.conf` # enable the new apache virtual host settings for your glpi instance `systemctl restart apache2`

### Set up the PHP.ini file

We recommend to use always the latest supported PHP release for better performance.

For GLPI to work properly it is recommended to change the following parameters on your php.ini file

- Open the php.ini file

```
vim /etc/php/8.1/apache2/php.ini
```

Change the following parameters

- `upload_max_filesize = 20M` Maximum size for uploaded files is set to 20 megabytes.

- `post_max_size = 20M` Maximum size for POST data (e.g., form submissions) is also set to 20 megabytes.

- `max_execution_time = 60` Maximum execution time for a PHP script is set to 60 seconds.

- `max_input_vars = 5000` Maximum number of input variables (e.g., form fields) a script can accept is 5000.

- `memory_limit = 256M` The maximum amount of memory a single PHP script can use is 256 megabytes.

- `session.cookie_httponly = On` Sets the "HttpOnly" attribute for session cookies

- `date.timezone = America/Sao_Paulo` Sets the default timezone for PHP to yours.

To add your timezone, please refer to the official [list of supported timezones for PHP](https://www.php.net/manual/en/timezones.php)

## 6 - Start Web Installation

Once the installation and configuration of dependencies are done, the installation can continue on a web browser with access to this same server. Open a Web browser and type the DNS record you have created for this server.
