Step1: Update and Upgrade the system.


1. Update and Upgrade the Ubuntu Packages:

```
sudo su
```

```
apt update && apt upgrade -y
```



Step2: Install Apache2 and PHP Modules.


2. Install Apache2:

```
apt install apache2 -y
```

3. Install dependencies:

```
sudo apt install php php-common libapache2-mod-php php-bz2 php-gd php-mysql \
php-curl php-mbstring php-imagick php-zip php-common php-curl php-xml \
php-json php-bcmath php-xml php-intl php-gmp zip unzip wget smbclient libmagickcore-6.q16-7-extra ffmpeg -y
```

4. Enable required Apache modules: 

```
a2enmod env rewrite dir mime headers setenvif ssl
```

5. Now, Restart, Enable and Check Apache is Running Properly.

```
systemctl restart apache2
systemctl enable apache2
systemctl status apache2
```

6. Check modules are loaded on Apache (Output omitted):

```
apache2ctl -M
```

Step3: Install and Configure MariaDB Server

7. Install mariadb-server package:

```
apt install mariadb-server -y
```

8. Login to MariaDB, Just type the below command, It will drop you to MySQL Prompt:

```
mysql
```

9. Create Database and User for Nextcloud and Provide User Permissions:


```
CREATE USER 'ncloud'@'localhost' IDENTIFIED BY 'admin123';
CREATE DATABASE ncloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
GRANT ALL PRIVILEGES ON ncloud.* TO 'ncloud'@'localhost';
FLUSH PRIVILEGES;
quit;
```


10. Now, restart and enable MariaDB service:

```
systemctl restart mariadb
systemctl enable mariadb
```

11. Check MariaDB is Running:

```
systemctl status mariadb
```

Step4: Download Nextcloud, Unzip and Permission


12. Download and unzip in the /var/www/html folder:

```
cd /var/www/html
wget https://download.nextcloud.com/server/releases/latest.zip
unzip latest.zip
```

13. Remove the zip file, which is not necessary now:

```
rm -rf latest.zip
```

14. Change the ownership of the nextcloud directory to the HTTP user:

```
chown -R www-data:www-data /var/www/html/nextcloud/
```

Step5: Install Nextcloud From Command Line


15. Run the below command to install nextcloud (provide your own credentials)

```
cd /var/www/html/nextcloud
sudo -u www-data php occ  maintenance:install --database \
"mysql" --database-name "ncloud"  --database-user "ncloud" --database-pass \
'admin123' --admin-user "admin" --admin-pass "password"
```

16. Nextcloud allows access only from localhost, it could through error “Access through untrusted domain”. we need to allow accessing Nextcloud by using ip or domain name:

```
sudo nano /var/www/html/nextcloud/config/config.php

  'trusted_domains' =>
  array (
    0 => 'localhost',
    1 => 'nextcloud.your_domain.ru',   // we Included the Sub Domain
  ),
  'overwritehost' => 'nextcloud.your_domain.ru',
  'overwriteprotocol' => 'https',
  'overwrite.cli.url' => 'nextcloud.your_domain.ru',
  'trusted_proxies' => 
  array (
    0 => '192.168.0.0/16',
    1 => '172.16.0.0/12',
    2 => '10.0.0.0/8',
    3 => 'fc00::/7',
    4 => 'fe80::/10',
    5 => '2001:db8::/32',
   ),
  'default_phone_region' => 'RU',
  'allow_local_remote_servers' => true',
  
```

17. Configure Apache to load Nextcloud from the /var/www/html/nextcloud folder:

```

sudo nano /etc/apache2/sites-enabled/000-default.conf

sudo nano /etc/apache2/sites-enabled/000-default.conf

<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html/nextcloud
        
        <Directory /var/www/html/nextcloud>
            Options Indexes FollowSymLinks
            AllowOverride All
            Require all granted
	      </Directory>
	      
	      ServerName nextcloud.yourdomain.ru
        <IfModule mod_headers.c>
            Header always set Strict-Transport-Security "max-age=15552000; includeSubDomains"
           </IfModule>
        
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

18. Now restart Apache:

```
systemctl restart apache2
```

19. Now, Go to the Browser and type http://[ip or fqdn] of the server, The below Nextcloud login page will appear.


Step6: Install and Configure PHP-FPM with Apache


20. Install PHP-FPM: #check actual version

```
apt install php8.3-fpm 
```

21. Check the PHP-FPM is running, its version and Socket is created:

```
service php8.3-fpm status
php-fpm8.3 -v
ls -la /var/run/php/php8.3-fpm.sock
```

22. Disable mod_php and prefork module:

```
a2dismod php8.3
a2dismod mpm_prefork
```

23. Enable PHP-FPM:

```
a2enmod mpm_event proxy_fcgi setenvif
a2enconf php8.3-fpm
```

24. Restart Apache to reload all the modules and configurations:

```
systemctl restart apache2
```

Now, for file upload size and performance settings, we need to tweak some php.ini settings listed below in the /etc/php/8.4/fpm/php.ini file. You can assign your own values depending on your environment.

```
upload_max_filesize = 64M
post_max_size = 96M
memory_limit = 512M
max_execution_time = 600
max_input_vars = 3000
max_input_time = 1000
```

25. Check the current values:

```
grep -E "upload_max_filesize|post_max_size|memory_limit|max_execution_time|max_input_vars|max_input_time" /etc/php/8.3/fpm/php.ini
```

26. Instead of manual change, you can execute the below command to change at once. it will save time:

```
sed -i 's/^upload_max_filesize.*/upload_max_filesize = 64M/; s/^post_max_size.*/post_max_size = 96M/; s/^memory_limit.*/memory_limit = 512M/; s/^max_execution_time.*/max_execution_time = 600/; s/^;max_input_vars.*/max_input_vars = 3000/; s/^max_input_time.*/max_input_time = 1000/' /etc/php/8.4/fpm/php.ini
```

Now, we need update PHP-FPM pool Configurations at /etc/php/8.4/fpm/pool.d/www.conf, below are some optimum values, but you should assign your own values.

```
pm.max_children = 64
pm.start_servers = 16
pm.min_spare_servers = 16
pm.max_spare_servers = 32
```

27. Check current values:

```
grep -E "pm.max_children|pm.start_servers|pm.min_spare_servers|pm.max_spare_servers" /etc/php/8.3/fpm/pool.d/www.conf
```

28. Change all the values at once with the below command:

```
sed -i 's/^pm.max_children = .*/pm.max_children = 64/; s/^pm.start_servers = .*/pm.start_servers = 16/; s/^pm.min_spare_servers = .*/pm.min_spare_servers = 16/; s/^pm.max_spare_servers = .*/pm.max_spare_servers = 32/' /etc/php/8.4/fpm/pool.d/www.conf
```

29. Now, restart PHP-FPM to apply all the changes:

```
service php8.3-fpm restart
```

Now, Insert the below code to apache’s default site configuration file /etc/apache2/sites-enabled/000-default.conf, it will direct apache to handover the php file processing to PHP-FPM.

	```
    <FilesMatch ".php$">
         SetHandler "proxy:unix:/var/run/php/php8.4-fpm.sock|fcgi://localhost/"
	</FilesMatch>
    ```

30. After providing the code, apache’s default site configuration will look like the below snippet:

```
nano /etc/apache2/sites-enabled/000-default.conf 
```
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
        DocumentRoot /var/www/html/nextcloud

        <Directory /var/www/html/nextcloud>
            Options Indexes FollowSymLinks
            AllowOverride All
            Require all granted
              </Directory>
           
        <FilesMatch ".php$"> 
            SetHandler "proxy:unix:/var/run/php/php8.3-fpm.sock|fcgi://localhost/"
              </FilesMatch>   
            
        ServerName nextcloud.your_domain.ru
        <IfModule mod_headers.c>
            Header always set Strict-Transport-Security "max-age=15552000; includeSubDomains"
           </IfModule>

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

31. Now, Restart Apache to take the change:

```
systemctl restart apache2
```


Step7: Create info.php Page for PHP feature check


32. Create info.php Page for PHP feature check:

Create an info.php page, it will show us whether PHP-FPM, OPCache, APCu are enabled with the PHP.

`
cd /var/www/html/nextcloud
`

```
nano info.php
```
```
    <?php phpinfo(); ?>
```

Now, Browse [URL]/info.php. if the PHP-FPM is enabled on the PHP. it will show “Server API FPM/FastCGI”


Step8: Enable OPCache in PHP

33. Enable OPCache in PHP:

Check it is running or not from the [URL]/info.php file previously we created.

Opcache JIT (Just-In-Time) compilation is an important feature. JIT compilation improves PHP performance by compiling code into machine language at runtime, rather than interpreting it each time it’s executed. This can significantly enhance the performance of CPU-intensive tasks. So it will be very effective to enable it to increase nextcloud performance.

```
nano /etc/php/8.4/fpm/conf.d/10-opcache.ini

zend_extension=opcache.so
opcache.enable=1
opcache.enable_cli=1
opcache.interned_strings_buffer=64
opcache.max_accelerated_files=12000
opcache.memory_consumption=512
opcache.save_comments=1
opcache.revalidate_freq=60
opcache.jit=on
opcache.jit = 1255
opcache.jit_buffer_size = 256M
```

34. Restart PHP-FPM to take effect the change:

```
service php8.3-fpm restart
```

Step9: Enable APCu in PHP

35. Enable APCu in PHP:

```
apt install php8.3-apcu
```

```
nano /etc/php/8.3/fpm/conf.d/20-apcu.ini
```

```
extension=apcu.so
apc.enable_cli=1
```

Now, Restart PHP-FPM and Apache

```
sudo systemctl restart php8.3-fpm
sudo systemctl restart apache2
```

Now, Check the [URL]/info.php again, it will show the “APCu support Enabled”

36. Configure Nextcloud to use APCu for memory caching:

```
nano /var/www/html/nextcloud/config/config.php
```

```
'memcache.local' => '\OC\Memcache\APCu',
```


Step10: Install and Configure Redis Cache

37. Install and Configure Redis Cache

In Nextcloud, Redis is used for local and distributed caching as well as transactional file locking. We used APCu for Local Caching which is faster than Redis. We will use Redis for File locking. Nextcloud’s Transactional File Locking mechanism locks files to avoid file corruption during normal operation.

38. Install Redis Server and Redis php extension

```
sudo apt install redis-server php-redis -y
```


39. Start and Enable the Redis service.

```
systemctl start redis-server
systemctl enable redis-server
systemctl status redis-server
```

40. Configure Redis to use Unix Socket than ports

```
sudo nano /etc/redis/redis.conf
```

```
port 0
unixsocket /var/run/redis/redis.sock
unixsocketperm 770
```

41. Add Apache user to the Redis group

```
usermod -a -G redis www-data
```

42. Configure Nextcloud for using Redis for File Locking

```
sudo nano /var/www/html/nextcloud/config/config.php
```

```
'filelocking.enabled' => 'true',
'memcache.distributed' => '\\OC\\Memcache\\Redis',
'memcache.locking' => '\\OC\\Memcache\\Redis',
'redis' => [
     'host'     => '/var/run/redis/redis.sock',
     'port'     => 0,
     'dbindex'  => 0,
     'password' => '',
     'timeout'  => 1.5,
],
```

43. Enable Redis session locking in PHP

```
sudo nano /etc/php/8.3/fpm/php.ini
```

```
redis.session.locking_enabled=1
redis.session.lock_retries=-1
redis.session.lock_wait_time=10000
```

44. Restart Redis, PHP-FPM and Apache

```
sudo systemctl restart redis-server
sudo systemctl restart php8.3-fpm
sudo systemctl restart apache2
```

45. You can check the features are enabled on PHP

```
nextcloud-ubuntu-24.04-redis
Redis is Installed for Nextcloud
```

https://help.nextcloud.com/t/installing-redis-for-memcache/162599/6


46. I assume that you use traefik as reverse proxy and you don't need to use certbot. That's why we inserted appropriate values in /etc/apache2/sites-enabled/000-default.conf

47. Enable Pretty URL’s:

```
sudo nano /var/www/html/nextcloud/config/config.php
```

```
'htaccess.RewriteBase' => '/',
```

This command will update the .htaccess file for the redirection

```
sudo -u www-data php --define apc.enable_cli=1 /var/www/html/nextcloud/occ maintenance:update:htaccess
```

48. Other neccessary steps


```
sudo -u www-data php /var/www/html/nextcloud/occ maintenance:repair --include-expensive
```
```
sudo -u www-data php /var/www/html/nextcloud/occ db:add-missing-indices
```
 
```
sudo -u www-data php /var/www/html/nextcloud/occ config:system:set maintenance_window_start --type=integer --value=1
```
 
```
sudo crontab -u www-data -e
```

```
*/5  *  *  *  * php -f /var/www/html/nextcloud/cron.php
```
 

