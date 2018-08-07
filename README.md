#Nginx PHP MySQL Build Status GitHub version
Docker running Nginx, PHP-FPM, Composer, MySQL and PHPMyAdmin.

Overview
Install prerequisites

Before installing project make sure the following prerequisites have been met.

Clone the project

We’ll download the code from its repository on GitHub.

Configure Nginx With SSL Certificates [Optional]

We'll generate and configure SSL certificate for nginx before running server.

Configure Xdebug [Optional]

We'll configure Xdebug for IDE (PHPStorm or Netbeans).

Run the application

By this point we’ll have all the project pieces in place.

Use Makefile [Optional]

When developing, you can use Makefile for doing recurrent operations.

Use Docker Commands

When running, you can use docker commands for doing recurrent operations.

Install prerequisites
For now, this project has been mainly created for Unix (Linux/MacOS). Perhaps it could work on Windows.

All requisites should be available for your distribution. The most important are :

Git
Docker
Docker Compose
Check if docker-compose is already installed by entering the following command :

which docker-compose
Check Docker Compose compatibility :

Compose file version 3 reference
The following is optional but makes life more enjoyable :

which make
On Ubuntu and Debian these are available in the meta-package build-essential. On other distributions, you may need to install the GNU C++ compiler separately.

sudo apt install build-essential
Images to use
Nginx
MySQL
PHP-FPM
Composer
PHPMyAdmin
Generate Certificate
You should be careful when installing third party web servers such as MySQL or Nginx.

This project use the following ports :

Server	Port
MySQL	8989
PHPMyAdmin	8080
Nginx	8000
Nginx SSL	3000
Clone the project
To install Git, download it and install following the instructions :

git clone https://github.com/nanoninja/docker-nginx-php-mysql.git
Go to the project directory :

cd docker-nginx-php-mysql
Project tree
.
├── Makefile
├── README.md
├── data
│   └── db
│       ├── dumps
│       └── mysql
├── doc
├── docker-compose.yml
├── etc
│   ├── nginx
│   │   ├── default.conf
│   │   └── default.template.conf
│   ├── php
│   │   └── php.ini
│   └── ssl
└── web
    ├── app
    │   ├── composer.json.dist
    │   ├── phpunit.xml.dist
    │   ├── src
    │   │   └── Foo.php
    │   └── test
    │       ├── FooTest.php
    │       └── bootstrap.php
    └── public
        └── index.php
Configure Nginx With SSL Certificates
You can change the host name by editing the .env file.

If you modify the host name, do not forget to add it to the /etc/hosts file.

Generate SSL certificates

source .env && sudo docker run --rm -v $(pwd)/etc/ssl:/certificates -e "SERVER=$NGINX_HOST" jacoelho/generate-certificate
Configure Nginx

Do not modify the etc/nginx/default.conf file, it is overwritten by etc/nginx/default.template.conf

Edit nginx file etc/nginx/default.template.conf and uncomment the SSL server section :

# server {
#     server_name ${NGINX_HOST};
#
#     listen 443 ssl;
#     fastcgi_param HTTPS on;
#     ...
# }
Configure Xdebug
If you use another IDE than PHPStorm or Netbeans, go to the remote debugging section of Xdebug documentation.

For a better integration of Docker to PHPStorm, use the documentation.

Get your own local IP address :

sudo ifconfig
Edit php file etc/php/php.ini and comment or uncomment the configuration as needed.

Set the remote_host parameter with your IP :

xdebug.remote_host=192.168.0.1 # your IP
Run the application
Copying the composer configuration file :

cp web/app/composer.json.dist web/app/composer.json
Start the application :

sudo docker-compose up -d
Please wait this might take a several minutes...

sudo docker-compose logs -f # Follow log output
Open your favorite browser :

http://localhost:8000
https://localhost:3000 (HTTPS not configured by default)
http://localhost:8080 PHPMyAdmin (username: dev, password: dev)
Stop and clear services

sudo docker-compose down -v
Use Makefile
When developing, you can use Makefile for doing the following operations :

Name	Description
apidoc	Generate documentation of API
clean	Clean directories for reset
code-sniff	Check the API with PHP Code Sniffer (PSR2)
composer-up	Update PHP dependencies with composer
docker-start	Create and start containers
docker-stop	Stop and clear all services
gen-certs	Generate SSL certificates for nginx
logs	Follow log output
mysql-dump	Create backup of all databases
mysql-restore	Restore backup of all databases
phpmd	Analyse the API with PHP Mess Detector
test	Test application with phpunit
Examples
Start the application :

sudo make docker-start
Show help :

make help
Use Docker commands
Installing package with composer
sudo docker run --rm -v $(pwd)/web/app:/app composer require symfony/yaml
Updating PHP dependencies with composer
sudo docker run --rm -v $(pwd)/web/app:/app composer update
Generating PHP API documentation
sudo docker-compose exec -T php php -d memory_limit=256M -d xdebug.profiler_enable=0 ./app/vendor/bin/apigen generate app/src --destination ./app/doc
Testing PHP application with PHPUnit
sudo docker-compose exec -T php ./app/vendor/bin/phpunit --colors=always --configuration ./app
Fixing standard code with PSR2
sudo docker-compose exec -T php ./app/vendor/bin/phpcbf -v --standard=PSR2 ./app/src
Checking the standard code with PSR2
sudo docker-compose exec -T php ./app/vendor/bin/phpcs -v --standard=PSR2 ./app/src
Analyzing source code with PHP Mess Detector
sudo docker-compose exec -T php ./app/vendor/bin/phpmd ./app/src text cleancode,codesize,controversial,design,naming,unusedcode
Checking installed PHP extensions
sudo docker-compose exec php php -m
Handling database
MySQL shell access
sudo docker exec -it mysql bash
and

mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD"
Creating a backup of all databases
mkdir -p data/db/dumps
source .env && sudo docker exec $(sudo docker-compose ps -q mysqldb) mysqldump --all-databases -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" > "data/db/dumps/db.sql"
Restoring a backup of all databases
source .env && sudo docker exec -i $(sudo docker-compose ps -q mysqldb) mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" < "data/db/dumps/db.sql"
Creating a backup of single database
Notice: Replace "YOUR_DB_NAME" by your custom name.

source .env && sudo docker exec $(sudo docker-compose ps -q mysqldb) mysqldump -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" --databases YOUR_DB_NAME > "data/db/dumps/YOUR_DB_NAME_dump.sql"
Restoring a backup of single database
source .env && sudo docker exec -i $(sudo docker-compose ps -q mysqldb) mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" < "data/db/dumps/YOUR_DB_NAME_dump.sql"
Connecting MySQL from PDO
<?php
    try {
        $dsn = 'mysql:host=mysql;dbname=test;charset=utf8;port=3306';
        $pdo = new PDO($dsn, 'dev', 'dev');
    } catch (PDOException $e) {
        echo $e->getMessage();
    }
?>
Help us
Any thought, feedback or (hopefully not!)
