#### So installieren und verwenden Sie Docker

##### Inhaltsverzeichnis
+ [Installation Ubuntu](#installation-ubuntu)<br>
    + [install docker-compose](#install-docker-compose)<br>
+ [Installation Windows](#installation-windows)<br>
+ [Vorbereitung](#vorbereitung)<br>
    + [Ordnerstruktur](#ordnerstruktur)<br>
    + [Das Dockerfile](#das-dockerfile)<br>
        + [Step1 - download and build a image](#step1-download-and-build-a-image)<br>
        + [Step2 - Tag image](#step2-tag-image)<br>
            + [Step2.1 - List all images](#step21-list-all-images)<br>
            + [Step2.2 - Tag image](#step22-tag-image)<br>
    + [Anlegen der Config Dateien](#anlegen-der-config-dateien)<br>
    + [Die docker-compose.yml](#docker-composeyml)<br>
+ [Ausführung](#ausführung)<br>
    + [StartDockerAmpStack as /bin/bash](#startdockerampstack-as-binbash)<br><br>
+ [Docker Cheat Sheet](#docker-cheat-sheet)<br>
+ [Ubuntu Mate 19.10 mit Docker/amp .iso](#ubuntu-mate-1910-mit-dockeramp-iso)<br>


##### Installation Ubuntu
```sh
$ sudo apt-get update
```

```sh
$ sudo apt-get install \
      apt-transport-https \
      ca-certificates \
      curl \
      gnupg-agent \
      software-properties-common
```

```sh
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

```sh
$ sudo apt-key fingerprint 0EBFCD88
```

```sh
$ sudo add-apt-repository \
     "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
     $(lsb_release -cs) \
     stable"
```

```sh
$ sudo apt-get install docker-ce
```

###### install docker-compose
```sh
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.25.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
```sh
$ sudo chmod +x /usr/local/bin/docker-compose
```
```sh
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

##### Installation Windows
`https://hub.docker.com/editions/community/docker-ce-desktop-windows/`<br>
`https://download.docker.com/win/stable/Docker%20Desktop%20Installer.exe`<br>
<br>

##### Vorbereitung 
Der einfachste Weg euch eine lokale Entwicklungsumgebung aufzusetzten (Apache, MariaDB, PHP) ist über ein Dockerfile.

###### Ordnerstruktur
`/home/$USER/dockerfiles/ampstack/`<br>
`/home/$USER/docker-compose/`<br>
`/home/$USER/htdocs`<br>
`/home/$USER/apache`<br>

###### Das Dockerfile

`/home/$USER/dockerfiles/ampstack/Dockerfile`<br>
`(Dateiname: Dockerfile hat keine Dateiendung)`

```text
FROM php:7.3.11-apache

RUN apt-get update && ACCEPT_EULA=Y && apt-get install -y \
        nano 

RUN apt-get install -y --no-install-recommends apt-utils
        
RUN \
    apt-get install libldap2-dev -y && \
    rm -rf /var/lib/apt/lists/* && \
    docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ && \
    docker-php-ext-install ldap
    
RUN docker-php-ext-install mysqli
RUN docker-php-ext-install pdo_mysql
RUN docker-php-ext-install pdo
```
`(Dateiname: Dockerfile hat keine Dateiendung)`<br><br>

+ ###### Step1 - download and build a image
    ```sh
    $ cd /home/$USER/dockerfiles/ampstack/
    ```
    ```sh
    $ sudo docker build - < Dockerfile
    ```
+ ###### Step2 - tag image
    + ###### Step2.1 - List all images
        ```sh
        $ sudo docker images
        ```
    Die Ausgabe sollte ungefähr so ausschauen.<br>
    Bitte die `IMAGE ID` von dem <none> Repository kopieren
    ```bash
  REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
  <none>                 <none>              ee9ee7fe9a47        50 secounds ago     434MB
  php                    7.3.11-apache       d04b0f5fdc60        4 months ago        434MB
    ```
   ###### Step2.2 - Tag image
    ```bash
    $ sudo docker tag ee9ee7fe9a47 ampstack2:new
    ```
    Die Ausgabe sollte jetzt so ausschauen<br>
    ```bash
  REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
  ampstack               new                 ee9ee7fe9a47        7 minutes ago     434MB
  php                    7.3.11-apache       d04b0f5fdc60        4 months ago        434MB
    ```
<br>

##### Anlegen der Config Dateien
+ Apache2 Config<br>
    | `/home/$USER/apache/apache2.conf`<br>
    | `http://gitlab.dev.local/smeiser/howto/blob/master/Docker-local-setup/apache2.conf`
    
+ php.ini Datei<br>
    | `/home/$USER/apache/php.ini`<br>
    | `http://gitlab.dev.local/smeiser/howto/blob/master/Docker-local-setup/php.ini`


###### docker-compose.yml
`/home/$USER/docker-compose/docker-compose.yml`<br>
```yaml
version: '3.7'

services:
    web:
        image: ampstack:new
        container_name: amp
        ports:
            - "80:80"
            - "443:443"
        volumes:
            - "/home/$USER/htdocs/:/var/www/html/"
#            - "/home/smeiser/apache/httpd.conf:/usr/local/apache2/conf/httpd.conf"
            - "/home/$USER/apache/apache2.conf:/etc/apache2/apache2.conf"
            - "/home/$USER/apache/php.ini:/usr/local/etc/php/conf.d/custom.ini"
#            - "/home/smeiser/apache/000-default.conf:/etc/apache2/sites-available/000-default.conf"
        networks:
            - default
        links:
            - mariadb

    mariadb:
        image: mariadb:10.1
        ports:
            - "3306:8000"
        volumes:
            - mariadb:/var/lib/mysql
        environment:
            TZ: "Europe/Berlin"
            MYSQL_ALLOW_EMPTY_PASSWORD: "no"
            MYSQL_ROOT_PASSWORD: "rootpwd"
            MYSQL_USER: 'testuser'
            MYSQL_PASSWORD: 'testpassword'
            MYSQL_DATABASE: 'testdb'

volumes:
    mariadb:
```
`($USER << bitte anpassen)`<br>
`(Die Volumes Pfade bitte anpassen)`
<br><br>


##### Ausführung
+ Erstmalige Ausführung:<br>
    `$ cd /home/$USER/docker-compose`
    <br>
    `$ sudo docker-compose up`
    <br>
    Docker lädt jetzt noch MariaDB runter und sollte dann Apache und die MariaDB starten<br>
    Im Browser könnten wir schon mal http://localhost/ eingeben und sollten einen "Index of /" bekommen

###### StartDockerAmpStack as /bin/bash
`/home/$USER/Schreibtisch/StartDockerAmpStack.sh`<br>
```bash
#!/bin/bash

cd /home/$USER/docker-compose/;
sudo docker-compose up
exit 0;
```
`$ cd /home/$USER/Schreibtisch`<br>
`$ sudo chmod +x StartDockerAmpStack.sh`<br>

##### docker Cheat Sheet
> docker Status
> ```sh
> $ sudo systemctl status docker
> ```

> runterladen von images
>```sh
> $ sudo docker pull httpd
> ```

> show all downloaded images
> ```sh
> $ sudo docker images
> ```

> How do I get into a Docker container's shell?
> ```sh
> $ sudo docker exec -it <mycontainer> bash
> ```

___

> List all container
> ```sh
> $ sudo docker ps -a
> ```

> To remove docker containers
> ```sh
> $ sudo docker container rm $containerID$
> ```

___

> List all images
> ```sh
> $ sudo sudo docker images
> ```

> To remove docker images:
> ```sh
> $ sudo docker image rm $ImageID$
> ```
___

> To stop all containers:
> ```sh
> $ docker stop $(docker ps -aq)
> ```

> To clear containers:
> ```sh
> $ docker rm -f $(docker ps -a -q)
> ```

> To clear images:
> ```sh
> $ docker rmi -f $(docker images -a -q)
> ```

> To clear volumes:
> ```sh
> $ docker volume rm $(docker volume ls -q)
> ```

> To clear networks:
> ```sh
> $ docker network rm $(docker network ls | tail -n+2 | awk '{if($2 !~ /bridge|none|host/){ print $1 }}')
> ```

##### Ubuntu Mate 19.10 mit Docker/amp .iso
