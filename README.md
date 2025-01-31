
```markdown
# GLPI Installation Guide on Ubuntu Server

This guide will walk you through the process of installing GLPI (Gestionnaire Libre de Parc Informatique) on an Ubuntu Server.

## Prerequisites

- Ubuntu Server 22.04 LTS
- Root or sudo access
- Internet connection for downloading packages
- Apache2 web server
- PHP (with required extensions)
- MariaDB (or MySQL)
- Disk space for GLPI installation

## Installation Steps

### 1. Update the Server

Update your system packages to ensure everything is up to date.

```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install Required Components

Install Apache, PHP, and required PHP extensions.

```bash
sudo apt install -y apache2 php php-{apcu,cli,common,curl,gd,imap,ldap,mysql,xmlrpc,xml,mbstring,bcmath,intl,zip,redis,bz2} libapache2-mod-php php-soap php-cas
sudo apt install -y mariadb-server
```

### 3. Secure MariaDB

Secure the MariaDB installation:

```bash
sudo mysql_secure_installation
```

### 4. Create Database and User for GLPI

Create a database and user for GLPI:

```bash
sudo mysql -uroot -p
CREATE DATABASE glpi;
CREATE USER 'glpi'@'localhost' IDENTIFIED BY 'yourstrongpassword';
GRANT ALL PRIVILEGES ON glpi.* TO 'glpi'@'localhost';
FLUSH PRIVILEGES;
```

### 5. Download GLPI Files

Navigate to the web server root directory and download the GLPI package.

```bash
cd /var/www/html
wget https://github.com/glpi-project/glpi/releases/download/10.0.15/glpi-10.0.15.tgz
tar -xvzf glpi-10.0.15.tgz
```

### 6. Prepare Directories for GLPI

Move necessary files and create required directories:

```bash
sudo mkdir /etc/glpi /var/lib/glpi /var/log/glpi
sudo mv /var/www/html/glpi/config /etc/glpi
sudo mv /var/www/html/glpi/files /var/lib/glpi
sudo mv /var/lib/glpi/_log /var/log/glpi
```

### 7. Configure GLPI Paths

Create `downstream.php`:

```bash
sudo vim /var/www/html/glpi/inc/downstream.php
```

Add the following content:

```php
<?php
define('GLPI_CONFIG_DIR', '/etc/glpi/');
if (file_exists(GLPI_CONFIG_DIR . '/local_define.php')) {
   require_once GLPI_CONFIG_DIR . '/local_define.php';
}
```

Create `local_define.php`:

```bash
sudo vim /etc/glpi/local_define.php
```

Add this content:

```php
<?php
define('GLPI_VAR_DIR', '/var/lib/glpi');
define('GLPI_DOC_DIR', GLPI_VAR_DIR);
define('GLPI_CRON_DIR', GLPI_VAR_DIR . '/_cron');
define('GLPI_DUMP_DIR', GLPI_VAR_DIR . '/_dumps');
define('GLPI_GRAPH_DIR', GLPI_VAR_DIR . '/_graphs');
define('GLPI_LOCK_DIR', GLPI_VAR_DIR . '/_lock');
define('GLPI_PICTURE_DIR', GLPI_VAR_DIR . '/_pictures');
define('GLPI_PLUGIN_DOC_DIR', GLPI_VAR_DIR . '/_plugins');
define('GLPI_RSS_DIR', GLPI_VAR_DIR . '/_rss');
define('GLPI_SESSION_DIR', GLPI_VAR_DIR . '/_sessions');
define('GLPI_TMP_DIR', GLPI_VAR_DIR . '/_tmp');
define('GLPI_UPLOAD_DIR', GLPI_VAR_DIR . '/_uploads');
define('GLPI_CACHE_DIR', GLPI_VAR_DIR . '/_cache');
define('GLPI_LOG_DIR', '/var/log/glpi');
```

### 8. Set Permissions

Ensure proper permissions for the GLPI directories:

```bash
sudo chown root:root /var/www/html/glpi/ -R
sudo chown www-data:www-data /etc/glpi -R
sudo chown www-data:www-data /var/lib/glpi -R
sudo chown www-data:www-data /var/log/glpi -R
sudo find /var/www/html/glpi/ -type f -exec chmod 0644 {} \;
sudo find /var/www/html/glpi/ -type d -exec chmod 0755 {} \;
```

### 9. Configure Apache Web Server

Create an Apache virtual host for GLPI:

```bash
sudo vim /etc/apache2/sites-available/glpi.conf
```

Add the following content:

```apache
<VirtualHost *:80>
   ServerName yourglpi.yourdomain.com
   DocumentRoot /var/www/html/glpi/public
   <Directory /var/www/html/glpi/public>
      Require all granted
      RewriteEngine On
      RewriteCond %{HTTP:Authorization} ^(.+)$
      RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
      RewriteCond %{REQUEST_FILENAME} !-f
      RewriteRule ^(.*)$ index.php [QSA,L]
   </Directory>
</VirtualHost>
```

Enable the site and restart Apache:

```bash
sudo a2dissite 000-default.conf
sudo a2enmod rewrite
sudo a2ensite glpi.conf
sudo systemctl restart apache2
```

### 10. Adjust PHP Configuration

Edit the `php.ini` file:

```bash
sudo vim /etc/php/8.1/apache2/php.ini
```

Update the following settings:

```ini
upload_max_filesize = 20M
post_max_size = 20M
max_execution_time = 60
max_input_vars = 5000
memory_limit = 256M
session.cookie_httponly = On
date.timezone = America/Sao_Paulo
```

### 11. Complete Web Installation

1. Open your browser and navigate to `http://yourglpi.yourdomain.com`.
2. Follow the on-screen instructions to complete the installation.
3. Provide database details:
   - Database name: `glpi`
   - Username: `glpi`
   - Password: Your database user password
4. Finish the setup and begin using GLPI.

---

## Conclusion

You have successfully installed GLPI on your Ubuntu server. You can now start managing your IT assets and services using GLPI.
```

This `README.md` includes all necessary steps for installing and configuring GLPI on an Ubuntu server. Let me know if you need any adjustments!
