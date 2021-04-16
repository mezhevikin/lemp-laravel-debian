# Install Nginx, PHP 8, MariaDB, Laravel 8, Certbot(Let's Encrypt) on Debian 10

**Tutorial works for DidgitalOcean,VScale.**

* Create new Debian 10 instance.

* Open terminal and connect to ssh

    ```sh
    ssh root@11.11.111.111
    ```

* Upgrade system

    ```sh
    sudo apt update
    sudo apt upgrade
    ```

* Enter `Y` few times and choise `Keep the Local version currenctly installed`.

* Install Nginx

    ```sh
    sudo apt install nginx
    ```
* Open http://11.11.111.111/ to test Nginx

* Install MariaDB

    ```sh
    sudo apt install mariadb-server
    ```

* Open MariaDB

    ```sh
    sudo mariadb
    ```

* Create database

    ```sh
    create database DATABASE_NAME;
    ```

* Create new db user

    ```sh
    CREATE USER YOUR_DATABASE_USER IDENTIFIED BY 'YOUR_DATABASE_USER_PASSWORD';
    ```
* Grant privileges to this user

    ```sh
    GRANT ALL ON DATABASE_NAME.* TO YOUR_DATABASE_USER;
    ```

* Exit from MariaDB

    ```sh
    exit
    ```

* Add SURY repository, for PHP 8.

    ```sh
    sudo apt -y install lsb-release apt-transport-https ca-certificates 
    ```

    ```sh
    sudo wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
    ```

    ```sh
    echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/php.list
    ```

* Upgrade system

    ```sh
    sudo apt update
    sudo apt upgrade
    ```
* Install PHP and extensions

    ```sh
    sudo apt install php8.0-fpm php8.0-mysql php8.0-mbstring php8.0-xml php8.0-bcmath zip unzip curl git
    ```
* Go to www directory

    ```sh
    cd /var/www
    ```
* Clone your laravel repository

    ```sh
    git clone https://github.com/name/example.git example
    ```

* **If you have private repo**, then [create github token](https://github.com/settings/tokens) and enter it instead of password.

    To save token for next time
    ```sh
    git config --global credential.helper store
    ```  

* Install composer 

    ```sh
    php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
    php -r "if (hash_file('sha384', 'composer-setup.php') === '756890a4488ce9024fc62c56153228907f1545c228516cbf63f885e036d37e9a59d27d63f46af1d4d07ee0f76181c7d3') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
    php composer-setup.php
    php -r "unlink('composer-setup.php');"
    ```

    ```sh
    mv composer.phar /usr/local/bin/composer
    ```
* Install composer dependencies

    ```sh
    composer install
    ```
* Add db settings to `.env`.

    ```sh
    nano .env
    ```

    ```sh
    DB_DATABASE=database
    DB_USERNAME=user
    DB_PASSWORD=password
    ```

* Set permissions to directories

    ```sh
    sudo chown -R www-data.www-data /var/www/example
    sudo chown -R www-data.www-data /var/www/example/storage
    sudo chown -R www-data.www-data /var/www/example/bootstrap/cache
    ```

* Migrate db

    ```sh
    php artisan migrate
    ```
* If you need import existings sql dump:

    ```sh
    mariadb -uroot database -p < /var/www/example.sql
    ```

* Add nginx config to `/etc/nginx/sites-available`

    ```sh
    nano /etc/nginx/sites-available/example.com.conf
    ```
    ```sh
    server {
        listen 80;
        server_name example.com;
        root /var/www/example/public;

        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Content-Type-Options "nosniff";

        index index.html index.htm index.php;

        charset utf-8;

        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }

        location = /favicon.ico { access_log off; log_not_found off; }
        location = /robots.txt  { access_log off; log_not_found off; }

        error_page 404 /index.php;

        location ~ \.php$ {
            fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
            include fastcgi_params;
        }

        location ~ /\.(?!well-known).* {
            deny all;
        }
    }
    ```

* Enable your Nginx config

    ```sh
    sudo ln -s /etc/nginx/sites-available/example.com.conf /etc/nginx/sites-enabled/
    ```
* Test nginx config

    ```sh
    nginx -t
    ```
* Reload nginx

    ```sh
    service nginx reload
    ```
* Install node.js and npm

    ```sh
    curl -fsSL https://deb.nodesource.com/setup_15.x | bash -
    apt-get install -y nodejs
    ```

    ```sh
    cd /var/www/example
    npm install
    npm run prod
    ```
* Add cron task

    ```sh
    crontab -l | { cat; echo "* * * * * php /var/www/example/artisan schedule:run >> /dev/null 2>&1"; } | crontab -
    ```
* Check cron file

     ```sh
    more /var/spool/cron/crontabs/root
    ```
* Install cerbot

     ```sh
    sudo apt install snapd
    sudo snap install core; sudo snap refresh core
    sudo snap install --classic certbot
    sudo ln -s /snap/bin/certbot /usr/bin/certbot
    sudo certbot certonly --nginx
    ```
















