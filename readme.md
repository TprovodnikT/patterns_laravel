            Instruction for docker container creation
                     with nginx and laravel
=================================================================

--First check if you have a docker

--Next, pull ubuntu server
```
    docker pull ubuntu
```
--Create docker network, to assign a static IP to your docker container
```
    docker network create --subnet=172.16.0.0/24 my_dock_net
```
--Run your docker image ubuntu
```
    docker run --net my_dock_net --ip 172.16.0.3 -it ubuntu bash
```
--In your docker container install this packages:
```
apt update                                                                                               #update the container
apt-get install php7.0 php7.0-curl php7.0-mcrypt php7.0-zip php7.0-fpm php7.0-xml php7.0-mbstring -y     #install php7.0
apt-get install mysql-server                                                                             #install mysql
mysql_secure_installation                                                                                #delete unnecessary stuff from mysql
apt-get install nginx                                                                                    #install nginx
apt-get install zip                                                                                      #install zip
```
--Create directory /home/laravel
```
    mkdir /home/laravel
```
--Download composer and do like in instruction
--Create user with group www-data
```
    useradd -m -d /home/laravel/ -G www-data composer
```
--Change owner for directory /home/laravel and permissions:
```
    chown -R composer:www-data /home/laravel
    chmod -R 755 /home/laravel
```
--Login to cli as composer user
```
    su -l composer
    ./composer create-project laravel/laravel patterns                                                   #Create a project with name patterns
    touch /home/laravel/patterns/storage/logs/laravel.log
```
--Exit from user composer by pressing ctrl+d
--Change permissions on folder /home/laravel/patterns/storage to 775
```
    chmod -R 775 /home/laravel/patterns/storage
```
--Change the value cgi.fix_pathinfo to 0 in file /etc/php/7.0/fpm/php.ini
    cgi.fix_pathinfo=0
--Create file laravel in directory /etc/nginx/sites-enabled
```
    touch /etc/nginx/sites-enabled/laravel
```
--Insert into file /etc/nginx/sites-enabled/laravel this:
```
server {
         listen 80;
         listen [::]:80 ipv6only=on;

         # Log files for Debugging
         access_log /var/log/nginx/laravel-access.log;
         error_log /var/log/nginx/laravel-error.log;

         # Webroot Directory for Laravel project
         root /home/laravel/patterns/public;
         index index.php index.html index.htm;

         # Your Domain Name
         server_name patterns.laravel;

         location / {
                 try_files $uri $uri/ /index.php?$query_string;
         }
         # PHP-FPM Configuration Nginx
         location ~ \.php$ {
                 try_files $uri =404;
                 fastcgi_split_path_info ^(.+\.php)(/.+)$;
                 fastcgi_pass unix:/run/php/php7.0-fpm.sock;
                 fastcgi_index index.php;
                 fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                 include fastcgi_params;
         }
 }
```
--Save your changes in docker by creating an image
```
    exit        #exit to your main host
    docker ps   #you will see docker container, which is runing. Choose what you need
    docker commit <your container number> patterns_laravel
```
--Run your docker container with mapping to directory in your host computer and with ip address from your created network
```
    docker run --net my_dock_net --ip 172.16.0.3 -v /home/user/workspace/patterns_laravel/:/home/laravel/ -it patterns_laravel bash
```
