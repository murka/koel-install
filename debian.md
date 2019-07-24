# How to install Koel on Debian 9


### Requirements

- **PHP 5.5.6 or > version & extensions**
  - `OpenSSL`
  - `PDO`
  - `Mbstring`
  - `Tokenizer`
  - `XML`
- **MariaDB**
- **NodeJS 8 or > version & yarn**
- **Composer**

### Update system & Install necessary packages.

1. ```apt update && apt upgrade -y```
2. ```apt install -y build-essential sudo dirmngr wget curl vim git```

### Create non-root user & add sudo access
```
adduser murka --gecos "Daniel Murka"
usermod -aG sudo murka
su - murka
```
***NOTE:*** Replace ```murka``` with your username

### Setup timezone

```sudo dpkg-reconfigure tzdata```

### Install PHP

```sudo apt install -y php7.2 php7.2-cli php7.2-fpm php7.2-common php7.2-mbstring php7.2-xml php7.2-mysql php7.2-curl php7.2-zip```

### Install MariaDB

```sudo apt install -y mariadb-server```

```sudo mysql_secure_installation```

```diff 
# Remove anonymous users? [Y/n] Y
# Disallow root login remotely? [Y/n] Y
# Remove test database and access to it? [Y/n] Y
# Reload privilege tables now? [Y/n] Y
```
```diff
sudo mysql -u root -p
# Enter password
```

```
CREATE DATABASE database;
GRANT ALL ON database.* TO 'username' IDENTIFIED BY 'password';
FLUSH PRIVILEGES;
EXIT;
```

### Install Nginx

```sudo apt install -y nginx```

```sudo vim /etc/nginx/sites-available/koel.conf```

***NOTE:*** Replace ```example.com``` with your domain

```
server {
  listen 80;
  server_name example.com;
  root /var/www/koel;
  index index.php;

  if ($request_uri !~ ^/$|index\.php|robots\.txt|api/|public/|remote) {
    return 404;
  }

  location /media/ {
    internal;
    alias $upstream_http_x_media_root;
   }

   location / {
     try_files $uri $uri/ /index.php?$args;
   }

   location ~ \.php$ {
     try_files $uri $uri/ /index.php?$args;
     fastcgi_param PATH_INFO $fastcgi_path_info;
     fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
     fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
     fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
     fastcgi_index index.php;
     fastcgi_split_path_info ^(.+\.php)(/.+)$;
     fastcgi_intercept_errors on;
     include fastcgi_params;
   }
}
```

```sudo ln -s /etc/nginx/sites-available/koel.conf /etc/nginx/sites-enabled/```

```diff
sudo nginx -t
# nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
# nginx: configuration file /etc/nginx/nginx.conf test is successful
```

```sudo systemctl reload nginx.service```

### Install NodeJS

```
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt install -y nodejs
```

### Install Yarn

```
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update && sudo apt install -y yarn
```

### Install Composer

A: ```sudo apt install composer```

B: 
```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer
```

### Install Koel

```
sudo mkdir -p /var/www/koel
cd /var/www/koel
sudo chown -R murka:murka /var/www/koel
git clone https://github.com/phanan/koel.git .
git fetch
git checkout v3.7.2
composer install
```

```
php artisan koel:init
```

Run ```sudo vim .env``` & set ```APP_URL``` to your URL

```
APP_URL=http://example.com
```

```
yarn install
```

Change ownership of the ```var/www/koel``` directory to ```www-data```

```sudo chown -R www-data:www-data /var/www/koel```

Done!


