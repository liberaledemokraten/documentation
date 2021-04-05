---
title: 'Multiple PHP Versions'
---

!!! You should generally use the latest PHP version available. So use this to upgrade to a newer version instead of for downgrades if possible.

## Installing PHP
It is of course possible to have different versions of PHP installed on the server. First, we install our target version (here PHP 7.4) and enable it in our Apache httpd server:

```sh
apt-get install install php7.4 php7.4-fpm
systemctl start php7.4-fpm
systemctl enable php7.4-fpm
systemctl reload apache2
service apache2 restart
```

The above lines install and generally enable PHP 7.4 without it being enabled for any specific domain yet. To actually use PHP 7.4, there are two approaches, one is to use Vesta Templates (recommended) and the other one is to modify the configuration files manually.

## Vesta Template
`cd` to `/usr/local/vesta/data/templates/web/apache2`. Now add the following files:

_PHP74-FPM.sh_
```bash
#!/bin/bash
# Adding php pool conf
user="$1"
domain="$2"
ip="$3"
home_dir="$4"
docroot="$5"

pool_conf="[$2]

listen = /run/php/php7.4-fpm-$2.sock
listen.owner = $1
listen.group = $1
listen.mode = 0666

user = $1
group = $1

pm = ondemand
pm.max_children = 8
request_terminate_timeout = 90s
pm.max_requests = 4000
pm.process_idle_timeout = 10s
pm.status_path = /status

php_admin_value[upload_tmp_dir] = /home/$1/tmp
php_admin_value[session.save_path] = /home/$1/tmp
php_admin_value[open_basedir] = $5:/home/$1/tmp:/bin:/usr/bin:/usr/local/bin:/var/www/html:/tmp:/usr/share:/etc/phpmyadmin:/var/lib/phpmyadmin:/etc/roundcube:/var/log/roundcube:/var/lib/roundcube
php_admin_value[upload_max_filesize] = 80M
php_admin_value[max_execution_time] = 30
php_admin_value[post_max_size] = 80M
php_admin_value[memory_limit] = 256M
php_admin_value[sendmail_path] = \"/usr/sbin/sendmail -t -i -f info@$2\"
php_admin_flag[mysql.allow_persistent] = off
php_admin_flag[safe_mode] = off

env[PATH] = /usr/local/bin:/usr/bin:/bin
env[TMP] = /home/$1/tmp
env[TMPDIR] = /home/$1/tmp
env[TEMP] = /home/$1/tmp
"

pool_file_56="/etc/php/5.6/fpm/pool.d/$2.conf"
pool_file_70="/etc/php/7.0/fpm/pool.d/$2.conf"
pool_file_71="/etc/php/7.1/fpm/pool.d/$2.conf"
pool_file_72="/etc/php/7.2/fpm/pool.d/$2.conf"
pool_file_73="/etc/php/7.3/fpm/pool.d/$2.conf"
pool_file_74="/etc/php/7.4/fpm/pool.d/$2.conf"

if [ -f "$pool_file_56" ]; then
    rm $pool_file_56
    service php5.6-fpm restart
fi

if [ -f "$pool_file_70" ]; then
    rm $pool_file_70
    service php7.0-fpm restart
fi

if [ -f "$pool_file_71" ]; then
    rm $pool_file_71
    service php7.1-fpm restart
fi

if [ -f "$pool_file_72" ]; then
    rm $pool_file_72
    service php7.2-fpm restart
fi

write_file=0
if [ ! -f "$pool_file_73" ]; then
  write_file=1
else
  user_count=$(grep -c "/home/$1/" $pool_file_73)
  if [ $user_count -eq 0 ]; then
    write_file=1
  fi
fi
if [ $write_file -eq 1 ]; then
    echo "$pool_conf" > $pool_file_74
    service php7.4-fpm restart
fi
if [ -f "/etc/php/7.4/fpm/pool.d/www.conf" ]; then
    rm /etc/php/7.4/fpm/pool.d/www.conf
fi

if [ -f "$pool_file_73" ]; then
    rm $pool_file_73
    service php7.3-fpm restart
fi

exit 0
```
This will set up the correct `.sock` files. So it is highly important that the `$pool_conf` is written in our target file's `pool_file`. But of course, this is not sufficient and we have to set the templates for the actual httpd configuration.

To do so `cp PHP-FPM-73.tpl PHP-FPM-74.tpl && cp PHP-FPM-73.stpl PHP-FPM-74.stpl`. Now you have a skeleton to work on. All you have to do now is to open `PHP-FPM-74.tpl` and `PHP-FPM-74.stpl` and modify it such that the handler is set as following:

```
<FilesMatch \.php$>
    SetHandler "proxy:unix:/run/php/php7.4-fpm-%domain%.sock|fcgi://localhost/"
</FilesMatch>
```

Lastly, log into Vesta and change the Apache template of your domain under the menu "Web" to `PHP-FPM-74` and save it. Now, your website should be using PHP 7.4 and you can switch any other website to PHP 7.4 too by just changing their Apache template. If you notice any errors, you can of course change back to the previous template and everything should work just as it did before.

That way, you can flexibly change between versions for different domains and remain safe in case you made a mistake while configuring things.

## Classic Approach
! The below lines are a valid configuration but highly disregarded to use in our case. We use Vesta and can and should make full use of Vesta's power of templates instead. It will result in a more clean, easier to maintain and flexible configuration that way.

If you are not utilizing Vesta but managing your own server without a server management software or if your server management software does not offer any easier method, you can go as described below. But be aware, that with this configuration, changing from one version to another will require modifying the configuration files each time. Also, keeping track of which PHP version you are using for which Virtual Host might be a little annoying too.

### Apache Config
go to `/home/{{username}}/conf/web` and open the apache configuration for the domain in question. The enter the following lines within the `VirtualHost` section:

```
<FilesMatch \.php> # Apache 2.4.10+ can proxy to unix socket 
    SetHandler "proxy:unix:/var/run/php/php7.4-fpm.sock|fcgi://localhost/" 
</FilesMatch> 
```

Thenafter restart apache.

### .htaccess
First, have a look at the above apache configuration and make sure `AllowOwerrideAll` is set to `All` within the `Directory` section. By default, it should be, if you are using Vesta CP. Afterwards, open the `.htaccess` file within the webroot of the domain, e.g. `/home/{{username}}/web/{{domain_name}}/public_html` and add the following lines:

```
<FilesMatch \.php>
    SetHandler "proxy:unix:/var/run/php/php7.4-fpm.sock|fcgi://localhost/" 
</FilesMatch> 
```