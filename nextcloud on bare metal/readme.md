1. Update and Upgrade the Ubuntu Packages:


```
sudo apt update && sudo apt upgrade -y
```


2. Install Apache2:

```
sudo apt install apache2 -y
```

3. Install dependencies:

```
sudo apt install php php-common libapache2-mod-php php-bz2 php-gd php-mysql \
php-curl php-mbstring php-imagick php-zip php-common php-curl php-xml \
php-json php-bcmath php-xml php-intl php-gmp zip unzip wget -y
```

4. Enable required Apache modules: 

```
a2enmod env rewrite dir mime headers setenvif ssl
```

5. Enable required Apache modules:

```
a2enmod env rewrite dir mime headers setenvif ssl
```

6. Now, Restart, Enable and Check Apache is Running Properly.

```
sudo systemctl restart apache2
sudo systemctl enable apache2
```

7. Check modules are loaded on Apache (Output omitted):

```
apache2ctl -M
```

8. Install mariadb-server package:

```
sudo apt install mariadb-server -y
```

9. Login to MariaDB, Just type the below command, It will drop you to MySQL Prompt:

```
mysql
```

10. Create Database and User for Nextcloud and Provide User Permissions:

11. Now, restart and enable MariaDB service:

```
sudo systemctl restart mariadb
sudo systemctl enable mariadb
```

12. Check MariaDB is Running:

```
sudo systemctl status mariadb
```

13. Download and unzip in the /var/www/html folder:

```
cd /var/www/html
sudo wget https://download.nextcloud.com/server/releases/latest.zip
sudo unzip latest.zip
```

14. Remove the zip file, which is not necessary now:

```
sudo rm -rf latest.zip
```

15. Change the ownership of the nextcloud directory to the HTTP user:

```
sudo chown -R www-data:www-data /var/www/html/nextcloud/
```

16. Run the below command to install nextcloud (provide your own credentials)

```
cd /var/www/html/nextcloud
sudo -u www-data php occ  maintenance:install --database \
"mysql" --database-name "ncloud"  --database-user "ncloud" --database-pass \
'username' --admin-user "admin" --admin-pass "password"
```

17. Nextcloud allows access only from localhost, it could through error “Access through untrusted domain”. we need to allow accessing Nextcloud by using ip or domain name:

```
sudo nano /var/www/html/nextcloud/config/config.php

  'trusted_domains' =>
  array (
    0 => 'localhost',
    1 => 'nextcloud.domain.ru',   // we Included the Sub Domain
  ),
  .....
```

18. Configure Apache to load Nextcloud from the /var/www/html/nextcloud folder:

```
sudo nano /etc/apache2/sites-enabled/000-default.conf

<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html/nextcloud
        
        <Directory /var/www/html/nextcloud>
            Options Indexes FollowSymLinks
            AllowOverride All
            Require all granted
	      </Directory>
        
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

19. Now restart Apache:

```
sudo systemctl restart apache2
```

20. Now, Go to the Browser and type http://[ip or fqdn] of the server, The below Nextcloud login page will appear.

21. Install PHP-FPM: #check actual version

```
sudo apt install php8.4-fpm 
```

22. Check the PHP-FPM is running, its version and Socket is created:

```
service php8.3-fpm status
php-fpm8.3 -v
ls -la /var/run/php/php8.3-fpm.sock
```

23. Disable mod_php and prefork module:

```
a2dismod php8.4
a2dismod mpm_prefork
```

24. Enable PHP-FPM:

```
a2enmod mpm_event proxy_fcgi setenvif
a2enconf php8.4-fpm
```

25. Restart Apache to reload all the modules and configurations:

```
sudo systemctl restart apache2
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

26. Check the current values:

```
grep -E "upload_max_filesize|post_max_size|memory_limit|max_execution_time|max_input_vars|max_input_time" /etc/php/8.3/fpm/php.ini
```

27. Instead of manual change, you can execute the below command to change at once. it will save time:

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

28. Check current values:

```
grep -E "pm.max_children|pm.start_servers|pm.min_spare_servers|pm.max_spare_servers" /etc/php/8.4/fpm/pool.d/www.conf
```

29. Change all the values at once with the below command:

```
sed -i 's/^pm.max_children = .*/pm.max_children = 64/; s/^pm.start_servers = .*/pm.start_servers = 16/; s/^pm.min_spare_servers = .*/pm.min_spare_servers = 16/; s/^pm.max_spare_servers = .*/pm.max_spare_servers = 32/' /etc/php/8.4/fpm/pool.d/www.conf
```

30. Now, restart PHP-FPM to apply all the changes:

```
service php8.4-fpm restart
```

Now, Insert the below code to apache’s default site configuration file /etc/apache2/sites-enabled/000-default.conf, it will direct apache to handover the php file processing to PHP-FPM.

	```
    <FilesMatch ".php$">
         SetHandler "proxy:unix:/var/run/php/php8.4-fpm.sock|fcgi://localhost/"
	</FilesMatch>
    ```

31. After providing the code, apache’s default site configuration will look like the below snippet:

```
sudo nano /etc/apache2/sites-enabled/000-default.conf 
```
```
<VirtualHost *:80>

	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html/nextcloud

	<Directory /var/www/html/nextcloud>
          Options Indexes FollowSymLinks
          AllowOverride All
          Require all granted
	</Directory>

	<FilesMatch ".php$"> 
          SetHandler "proxy:unix:/var/run/php/php8.4-fpm.sock|fcgi://localhost/"
	</FilesMatch>

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

32. Now, Restart Apache to take the change:

```
sudo systemctl restart apache2
```

33. Create info.php Page for PHP feature check:

Create an info.php page, it will show us whether PHP-FPM, OPCache, APCu are enabled with the PHP.

`
cd /var/www/html/nextcloud
`

```
sudo nano info.php
```
```
    <?php phpinfo(); ?>
```

Now, Browse [URL]/info.php. if the PHP-FPM is enabled on the PHP. it will show “Server API FPM/FastCGI”

34. Enable OPCache in PHP:

Check it is running or not from the [URL]/info.php file previously we created.

Opcache JIT (Just-In-Time) compilation is an important feature. JIT compilation improves PHP performance by compiling code into machine language at runtime, rather than interpreting it each time it’s executed. This can significantly enhance the performance of CPU-intensive tasks. So it will be very effective to enable it to increase nextcloud performance.

```
sudo nano /etc/php/8.4/fpm/conf.d/10-opcache.ini

zend_extension=opcache.so
opcache.enable_cli=1
opcache.jit=on
opcache.jit = 1255
opcache.jit_buffer_size = 128M
```

35. Restart PHP-FPM to take effect the change:

```
service php8.4-fpm restart
```

36. Enable APCu in PHP:

```
sudo apt install php8.4-apcu
```

```
sudo nano /etc/php/8.4/fpm/conf.d/20-apcu.ini
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

37. Configure Nextcloud to use APCu for memory caching:

```
sudo nano /var/www/html/nextcloud/config/config.php
```

```
'memcache.local' => '\OC\Memcache\APCu',
```

37. Install and Configure Redis Cache

In Nextcloud, Redis is used for local and distributed caching as well as transactional file locking. we used APCu for Local Caching which is faster than Redis. We will use Redis for File locking. Nextcloud’s Transactional File Locking mechanism locks files to avoid file corruption during normal operation.

38. Install Redis Server and Redis php extension

```
sudo apt install redis-server php-redis -y
```


39. Start and Enable the Redis service.

```
systemctl start redis-server
systemctl enable redis-server
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
'memcache.locking' => '\OC\Memcache\Redis',
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

We can check Redis use, (by enabling the Redis port in the Redis configuration) by running the command “redis-cli MONITOR“, during Nextcloud loading it will show live data on the screen.

Now, that we have finished Performance improvement steps. We are going to work for the Security, First of all, we will install an SSL certificate for Nextcloud.

46. I assume that you use traefik as reverse proxy and you don't need to use certbot

47. Enable Pretty URL’s:

```
sudo nano /var/www/html/nextcloud/config/config.php
```

```
sudo nano /var/www/html/nextcloud/config/config.php
```

This command will update the .htaccess file for the redirection

```
sudo -u www-data php --define apc.enable_cli=1 /var/www/html/nextcloud/occ maintenance:update:htaccess
```

