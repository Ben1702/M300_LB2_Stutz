# LB03
## Inhaltsverzeichnis

[Einleitung](#Einleitung)
  - [Benötigte Vagrant-System-Änderungen](#benötigte-vagrant-system-änderungen)

[Realisierung](#Realisierung)
  - [Vorbereitung](#vorbereitung)
  - [docker-compose.yml](#docker-composeyml)
    - [MySQL](#mysql)
    - [PHP](#php)
    - [Apache](#apache)
    - [Redis](#redis)
    - [PHPmyAdmin](#phpmyadmin)
    - [Volumes](#volumes)
  - [Apache-Konfiguration](#apache-konfiguration)

[Testen](#testing)

## Einleitung
Für diese LB möchte ich eine Docker-Compose erstellen, welche eine MySQL-Datenbank per MyPHP ins LAN verfügbar stellt. Als Grundlage für Docker nehme ich eine Kopie der VM, welche Herr Berger in seinem M300 Github zur Verfügung stellte.

### Benötigte Vagrant-System-Änderungen
Um die Komposition besser testen zu können, nehme ich die VM in mein LAN auf, was bedeutet, dass es die IP-Adresse *192.168.0.45 /24* bekommt. Vor der Abgabe werde ich dies wieder auf die verlangte Adresse *192.168.60.101 /24* ändern.
<br>

Ebenso füge ich diese Provision ins Vagrantfile ein, um mein Dockerfile und Docker-Compose immer verfügbar zu haben:
```shell
  # Shell Provision
  config.vm.provision "shell", inline: <<-SHELL 
  
   apt-get update
   apt-get install -y docker-compose

   mkdir compose-projekt
   mkdir compose-projekt/docker
   mkdir compose-projekt/docker/apache
   mkdir compose-projekt/docker/apache/certs
   mkdir compose-projekt/docker/mysql
   mkdir compose-projekt/docker/mysql/data
   mkdir compose-projekt/docker/php
   mkdir compose-projekt/docker/www
   sudo wget -P ./compose-projekt/docker https://raw.githubusercontent.com/Ben1702/M300_LB_Stutz/main/lb3/docker-compose.yml
   
  SHELL
```

## Realisierung
### Vorbereitung
Als erstes erstelle ich auf der VM folgende Ordnerstruktur im Home-Ordner:
```shell
compose-projekt/docker
docker/apache/certs
docker/mysql/data
docker/php
docker/www
```
Wie unter den [Vagrant-System-Änderungen](#benötigte-vagrant-system-änderungen) bereits zu sehen war, wird mein im Repository bereits erstelltes docker-compose-File in den Docker-Ordner kopiert. Also so:
```shell
compose-projekt/docker/docker-compose.yml
```
<br>

### docker-compose.yml
Alle folgenden Abschnitte werden in das Docker-Compose-File eingetragen. Das File operiert auf Version 3.

#### MySQL
Für den MYSQL-Container erstelle ich folgenden EIntrag unter services:
```docker
  mysql:
    container_name: "mysql"
    image: mysql:latest
    environment:
      - MYSQL_ROOT_PASSWORD=Welcome$20
      - MYSQL_USER=admin
      - MYSQL_PASSWORD=Welcome$20
    ports:
      - '3306:3306'
    volumes:
      - ./docker/mysql/data:/mysql/data
```
Dies erstellt ein MySQL-Container, welcher bereits mit den Admin-Anmeldedaten gefüttert wurde. Anmelden kann man sich mit dem User **Admin** und dem Passwort **Welcome$20**.

#### PHP
Für PHP selbst besteht dieser Eintrag:
```docker
  php:  
    container_name: "php"
    image: bitnami/php-fpm:7.4
    depends_on:
      - redis
    volumes:
      - ./docker/www:/app:delegated
      - ./docker/php/php.ini:/opt/php/etc/conf.d/php.ini:ro
```
Hierfür benutze ich ein leicht angepasstes php-image, *php-fpm*, vom alternativen Image-Anbieter Bitnami. FPM bietet nützliche Features für alle möglichen PHP-Seiten.
Ebenso hat es eine Dependenz an Redis, einem Datenstruktur-Dienst.

#### Apache
```docker
apache:
    container_name: "apache"
    image: apache:2.4
    ports:
      - '80:8080'
      - '443:8443'
    depends_on:
      - php
    volumes:
      - ./docker/www:/app:delegated
      - ./docker/apache/my_vhost.conf:/vhosts/myapp.conf:ro
      - ./docker/apache/certs:/certs
```
Apache wird recht standardmässig implementiert, einfach mit der Dependenz auf PHP und einem zusätzlichem Port. <br>
"my_vhost.conf" wird als Konfigurationsfile für Apache dienen.

#### Redis
```docker
redis:
    container_name: "redis"
    image: redis:latest
    environment:
      - REDIS_PASSWORD=Welcome$20
```
Redis ist wie bereits in PHP gesagt ein Datenstruktur-Dienst für Server. Dieser wird äusserst Standardmässig aufgebaut, nur mit einem Passwort versehen.

#### PHPmyAdmin
```docker
phpmyadmin:
    container_name: "phpmyadmin"
    image: phpmyadmin:latest
    depends_on:
      - mysql
    ports:
      - '81:8080'
      - '8143:8443'
    environment:
      - DATABASE_HOST=host.docker.internal
```
Auch PHPmyAdmin wird ziemlich dem Standard getreu aufgebaut, nur mit einer Dependenz auf MySQL und leicht veränderten Ports.

#### Volumes
Als letztes im Compose-File kommen noch Volumes dran.
```docker
volumes:
    mysql:
      driver: local
```
Das Docker-Compose-File kann nun geschlossen und gespeichert werden.
### Apache-Konfiguration
Als nächstes muss Apache konfiguriert werden. Dafür wird als erstes ein SSL-Zertifikat benötigt, welches durch diesen Command erstellt werden kann:
```shell
openssl req -x509 -newkey rsa:4096 -sha256 -nodes -keyout server.key -out server.crt -subj "/CN=dstack.local" -days 3650
```
Das Zertifikat wird am besten gleich im Ordner *docker/apache/certs* erstellt, da es dort abgespeichert werden soll.

Nun an die eigentliche Konfig. Dafür wird unter *docker/apache* das File **my_vhost.conf** erstellt, mit folgendem Inhalt:
```apache
<VirtualHost *:8080>
  DocumentRoot "/app"
  ProxyPassMatch ^/(.*\.php(/.*)?)$ fcgi://php:9000/app/$1
  <Directory "/app">
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
    DirectoryIndex index.html index.php
  </Directory>
</VirtualHost>

<VirtualHost *:8443>
  SSLEngine on  
  SSLCipherSuite ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP:+eNULL  
  SSLCertificateFile "/certs/server.crt"  
  SSLCertificateKeyFile "/certs/server.key"  

  DocumentRoot "/app"
  ProxyPassMatch ^/(.*\.php(/.*)?)$ fcgi://php:9000/app/$1
  <Directory "/app">
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
    DirectoryIndex index.html index.php
  </Directory>
</VirtualHost>
```
Dies erstellt die Konfigurationen für beide supporteten Ports.
Diese Konfig-datei ist ebenfalls hier im Repository zu finden.

Als letztes für Apache wird noch im Ordner *docker/www* die Datei **info.php** erstellt, welche folgenden Inhalt hat:
```php
<?php
phpinfo();
?>
```
Datei ebenfalls im Repository vorhanden.
### MySQL-Konfiguration
MySQL benötigt nur den bereits erstellten, leeren Ordner *docker/mysql/data*. Den Rest übernimmt das Compose-File.

### PHP-Konfiguration
Als PHP-Konfig wird im Ordner *docker/php* die Datei **php.ini** mit diesem Inhalt erstellt:
```php
display_errors = On
expose_php = off

max_execution_time = 360
max_input_time = 360
memory_limit = 256M
upload_max_filesize = 1G
post_max_size = 1G

opcache.enable = 1
opcache.revalidate_freq = 2
opcache.validate_timestamps = 1
opcache.interned_strings_buffer = 32
opcache.memory_consumption = 256

extension=imagick.so
zend_extension = "/opt/bitnami/php/lib/php/extensions/xdebug.so"

[Xdebug]
xdebug.remote_autostart=1
xdebug.remote_enable=1
xdebug.default_enable=0
xdebug.remote_host=host.docker.internal
xdebug.remote_port=9000
xdebug.remote_connect_back=0
xdebug.profiler_enable=0
xdebug.remote_log="/tmp/xdebug.log"
```
Diese Konfig bestimmt Limitationen des Servers und Standorten von benötigten Dateien.<br>
Auch diese Konfig ist im Repository zu finden.

### PHPmyAdmin-Konfiguration
PHPmyAdmin benötigt ebenfalls keine zusätzliche Konfiguration und sollte nach Aufstarten der Container auf *[IP-Adresse]:81* / *[IP-Adresse]:8143* erreichbar sein.

## Testen
