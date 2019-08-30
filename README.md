# What is MediaWiki?

> MediaWiki is an extremely powerful, scalable software and a feature-rich wiki implementation that uses PHP to process and display data stored in a database, such as MySQL.

Pages use wiii's wikitext format, so that users without knowledge of XHTML or CSS can edit them easily.

https://www.mediawiki.org/

# TL;DR;

## Docker Compose

```bash
$ curl -sSL https://raw.githubusercontent.com/bitnami/bitnami-docker-mediawiki/master/docker-compose.yml > docker-compose.yml
$ docker-compose up -d
```

# Why use Bitnami Images?

* Bitnami closely tracks upstream source changes and promptly publishes new versions of this image using our automated systems.
* With Bitnami images the latest bug fixes and features are available as soon as possible.
* Bitnami containers, virtual machines and cloud images use the same components and configuration approach - making it easy to switch between formats based on your project needs.
* All our images are based on [minideb](https://github.com/bitnami/minideb) a minimalist Debian based container image which gives you a small base container image and the familiarity of a leading linux distribution.
* All Bitnami images available in Docker Hub are signed with [Docker Content Trust (DTC)](https://docs.docker.com/engine/security/trust/content_trust/). You can use `DOCKER_CONTENT_TRUST=1` to verify the integrity of the images.
* Bitnami container images are released daily with the latest distribution packages available.

> This [CVE scan report](https://quay.io/repository/bitnami/mediawiki?tab=tags) contains a security report with all open CVEs. To get the list of actionable security issues, find the "latest" tag, click the vulnerability report link under the corresponding "Security scan" field and then select the "Only show fixable" filter on the next page.

# How to deploy MediaWiki in Kubernetes?

Deploying Bitnami applications as Helm Charts is the easiest way to get started with our applications on Kubernetes. Read more about the installation in the [Bitnami MediaWiki Chart GitHub repository](https://github.com/bitnami/charts/tree/master/upstreamed/mediawiki).

Bitnami containers can b
```bash
$ curl -sSL https://raw.githubusercontent.com/bitnami/bitnami-docker-mediawiki/master/docker-compose.yml > docker-compose.yml
$ docker-compose up -d
``` 

### Run the application manually

If you want to run the application manually instead of using docker-compose, these are the basic steps you need to run:

1. Create a new network for the application and the database:

  ```bash
  $ docker network create mediawiki-tier
  ```

2. Create a volume for MariaDB persistence and create a MariaDB container

  ```bash
  $ docker volume create --name mariadb_data
  $ docker run -d --name mariadb \
    -e ALLOW_EMPTY_PASSWORD=yes \
    -e MARIADB_USER=bn_mediawiki \
    -e MARIADB_DATABASE=bitnami_mediawiki \
    --net mediawiki-tier \
    --volume mariadb_data:/bitnami \
    bitnami/mariadb:latest
  ```

  *Note:* You need to give the container a name in order for Mediawiki to resolve the host

3. Create volumes for MediaWiki persistence and launch the container

  ```bash
  $ docker volume create --name mediawiki_data
  $ docker run -d --name mediawiki -p 80:80 -p 443:443 \
    -e ALLOW_EMPTY_PASSWORD=yes \
    -e MEDIAWIKI_DATABASE_USER=bn_mediawiki \
    -e MEDIAWIKI_DATABASE_NAME=bitnami_mediawiki \
    --net mediawiki-tier \
    --volume mediawiki_data:/bitnami \
    bitnami/mediawiki:latest
  ```
Then you can access your application at http://your-ip/

## Persisting your application

If you remove the container all your data and configurations will be lost, and the next time you run the image the database will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed.

For persistence you should mount a volume at the `/bitnami` path. Additionally you should mount a volume for [persistence of the MariaDB data](https://github.com/bitnami/bitnami-docker-mariadb#persisting-your-database).

The above examples define docker volumes namely `mariadb_data` and `mediawiki_data`. The MediaWiki application state will persist as long as these volumes are not removed.

To avoid inadvertent removal of these volumes you can [mount host directories as data volumes](https://docs.docker.com/engine/tutorials/dockervolumes/). Alternatively you can make use of volume plugins to host the volume data.

### Mount host directories as data volumes with Docker Compose

This requires a minor change to the [`docker-compose.yml`](https://github.com/bitnami/bitnami-docker-mediawiki/blob/master/docker-compose.yml) file present in this repository: 

```yaml
services:
  mariadb:
  ...
    volumes:
      - '/path/to/mariadb-persistence:/bitnami'
  ...
  mediawiki:
  ...
    volumes:
      - '/path/to/mediawiki-persistence:/bitnami'
  ...
```

### Mount host directories as data volumes using the Docker command line

In this case you need to specify the directories to mount on the run command. The process is the same than the one previously shown:

1. Create a network (if it does not exist):

  ```bash
  $ docker network create mediawiki-tier
  ```

2. Create a MariaDB container with host volume:

  ```bash
  $ docker run -d --name mariadb \
    -e ALLOW_EMPTY_PASSWORD=yes \
    -e MARIADB_USER=bn_mediawiki \
    -e MARIADB_DATABASE=bitnami_mediawiki \
    --net mediawiki-tier \
    --volume /path/to/mariadb-persistence:/bitnami \
    bitnami/mariadb:latest
  ```

  *Note:* You need to give the container a name in order to MediaWiki to resolve the host

3. Run the MediaWiki container:

  ```bash
  $ docker run -d --name mediawiki -p 80:80 -p 443:443 \
    -e ALLOW_EMPTY_PASSWORD=yes \
    -e MEDIAWIKI_DATABASE_USER=bn_mediawiki \
    -e MEDIAWIKI_DATABASE_NAME=bitnami_mediawiki \
    --net mediawiki-tier \
    --volume /path/to/mediawiki-persistence:/bitnami \
    bitnami/mediawiki:latest
  ```

# Upgrade this application

Bitnami provides up-to-date versions of MariaDB and MediaWiki, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container. We will cover here the upgrade of the Mediawiki container. For the MariaDB upgrade see https://github.com/bitnami/bitnami-docker-mariadb/blob/master/README.md#upgrade-this-image

1. Get the updated images:

  ```bash
  $ docker pull bitnami/mediawiki:latest
  ```

2. Stop your container

 * For docker-compose: `$ docker-compose stop mediawiki`
 * For manual execution: `$ docker stop mediawiki`

3. Take a snapshot of the application state

```bash
$ rsync -a /path/to/mediawiki-persistence /path/to/mediawiki-persistence.bkp.$(date +%Y%m%d-%H.%M.%S)
```

Additionally, [snapshot the MariaDB data](https://github.com/bitnami/bitnami-docker-mariadb#step-2-stop-and-backup-the-currently-running-container)

You can use these snapshots to restore the application state should the upgrade fail.

4. Remove the currently running container

 * For docker-compose: `$ docker-compose rm -v mediawiki`
 * For manual execution: `$ docker rm -v mediawiki`

5. Run the new image

 * For docker-compose: `$ docker-compose up mediawiki`
 * For manual execution ([mount](#mount-persistent-folders-manually) the directories if needed): `docker run --name mediawiki bitnami/mediawiki:latest`

# Configuration

## Environment variables

When you start the mediawiki image, you can adjust the configuration of the instance by passing one or more environment variables either on the docker-compose file or on the docker run command line. If you want to add a new environment variable:

 * For docker-compose add the variable name and value under the application section in the [`docker-compose.yml`](https://github.com/bitnami/bitnami-docker-mediawiki/blob/master/docker-compose.yml) file present in this repository:

```yaml
mediawiki:
  ...
  environment:
    - MEDIAWIKI_PASSWORD=my_password
  ...
```

 * For manual execution add a `-e` option with each variable and value:

  ```bash
  $ docker run -d --name mediawiki -p 80:80 -p 443:443 \
    -e MEDIAWIKI_PASSWORD=my_password \
    --net mediawiki-tier \
    --volume /path/to/mediawiki-persistence:/bitnami \
    bitnami/mediawiki:latest
  ```

Available variables:

##### User and Site configuration

To configure Mediawiki to send email using SMTP you can set the following environment variables:

- `SMTP_HOST`: SMTP host.
- `SMTP_HOST_ID`: SMTP host ID.
- `SMTP_PORT`: SMTP port.
- `SMTP_USER`: SMTP account user.
- `SMTP_PASSWORD`: SMTP account password.

This would be an example of SMTP configuration using a GMail account:

 * Modify the [`docker-compose.yml`](https://github.com/bitnami/bitnami-docker-mediawiki/blob/master/docker-compose.yml) file present in this repository: 

```yaml
  mediawiki:
  ...
    environment:
      - MEDIAWIKI_DATABASE_USER=bn_mediawiki
      - MEDIAWIKI_DATABASE_NAME=bitnami_mediawiki
      - ALLOW_EMPTY_PASSWORD=yes
      - SMTP_HOST=ssl://smtp.gmail.com
      - SMTP_HOST_ID=mydomain.com
      - SMTP_PORT=465
      - SMTP_USER=your_email@gmail.com
      - SMTP_PASSWORD=your_password
  ...
```
 * For manual execution:

  ```bash
  $ docker run -d --name mediawiki -p 80:80 -p 443:443 \
    -e MEDIAWIKI_DATABASE_USER=bn_mediawiki \
    -e MEDIAWIKI_DATABASE_NAME=bitnami_mediawiki \
    -e SMTP_HOST=ssl://smtp.gmail.com \
    -e SMTP_HOST_ID=mydomain.com \
    -e SMTP_PORT=465 \
    -e SMTP_USER=your_email@gmail.com \
    -e SMTP_PASSWORD=your_password \
    --net mediawiki-tier \
    --volume /path/to/mediawiki-persistence:/bitnami \
    bitnami/mediawiki:latest
  ```

# How to install imagemagick in the Bitnami MediaWiki Docker image

If you require better quality thumbnails for your uploaded images, you may want to install imagemagick instead of using GD. To do so you can build your own docker image adding the `imagemagick` system package.

1. Create the following Dockerfile

```
FROM bitnami/mediawiki:latest
RUN install_packages imagemagick
```

2. Build the docker image

```
$ docker build -t bitnami/mediawiki:imagemagick .
```

3. Edit the _docker-compose.yml_ to use the docker image built in the previous step.

4. Finally exec into your MediaWiki container and edit the file _/opt/bitnami/mediawiki/LocalSettings.php_ as described [here](https://www.mediawiki.org/wiki/Manual:Installing_third-party_tools#Image_thumbnailing) in order to start using imagemagick.

# How to migrate from a Bitnami Mediawiki Stack

You can follow these steps in order to migrate it to this container:

1. Export the data from your SOURCE installation: (assuming an installation in `/opt/bitnami` directory)

  ```bash
  $ mysqldump -u root -p bitnami_mediawiki > ~/backup-mediawiki-database.sql
  $ gzip -c ~/backup-mediawiki-database.sql > ~/backup-mediawiki-database.sql.gz
  $ cd /opt/bitnami/apps/mediawiki/htdocs/
  $ tar cfz ~/backup-mediawiki-extensions.tar.gz extensions
  $ tar cfz ~/backup-mediawiki-images.tar.gz images
  $ tar cfz ~/backup-mediawiki-skins.tar.gz skins
  ```

2. Copy the backup files to your TARGET installation:

  ```bash
  $ scp ~/backup-mediawiki-* YOUR_USERNAME@TARGET_HOST:~
  ```

3. Create the Mediawiki Container as described in the section [How to use this Image (Using Docker Compose)](https://github.com/bitnami/bitnami-docker-mediawiki#using-docker-compose)

4. Wait for the initial setup to finish. You can follow it with

  ```bash
  $ docker-compose logs -f mediawiki
  ```

  and press `Ctrl-C` when you see this:

  ```
  nami    INFO  mediawiki successfully initialized
  Starting mediawiki ...
  ```

5. Stop Apache:

  ```bash
  $ docker-compose exec mediawiki nami stop apache
  ```

6. Obtain the password used by Mediawiki to access the database in order avoid reconfiguring it:

  ```bash
  $ docker-compose exec mediawiki bash -c 'cat /opt/bitnami/mediawiki/LocalSettings.php | grep wgDBpassword'
  ```

7. Restore the database backup: (replace ROOT_PASSWORD below with your MariaDB root password)

  ```bash
  $ cd ~
  $ docker-compose exec mariadb mysql -u root -pROOT_PASSWORD
  $ MariaDB [(none)]> drop database bitnami_mediawiki;
  $ MariaDB [(none)]> create database bitnami_mediawiki;
  $ MariaDB [(none)]> grant all privileges on bitnami_mediawiki.* to 'bn_mediawiki'@'%' identified by 'PASSWORD_OBTAINED_IN_STEP_6';
  $ MariaDB [(none)]> exit
  $ gunzip -c ./backup-mediawiki-database.sql.gz | docker exec -i $(docker-compose ps -q mariadb) mysql -u root bitnami_mediawiki -pROOT_PASSWORD
  ```

8. Restore extensions/images/skins directories from backup:

  ```bash
  $ cat ./backup-mediawiki-extensions.tar.gz | docker exec -i $(docker-compose ps -q mediawiki) bash -c 'cd /bitnami/mediawiki/ ; tar -xzvf -'
  $ cat ./backup-mediawiki-images.tar.gz | docker exec -i $(docker-compose ps -q mediawiki) bash -c 'cd /bitnami/mediawiki/ ; tar -xzvf -'
  $ cat ./backup-mediawiki-skins.tar.gz | docker exec -i $(docker-compose ps -q mediawiki) bash -c 'cd /bitnami/mediawiki/ ; tar -xzvf -'
  ```

9. Fix Mediawiki directory permissions:

  ```bash
  $ docker-compose exec mediawiki chown -R daemon:daemon /bitnami/mediawiki
  ```

10. Restart Apache:

  ```bash
  $ docker-compose exec mediawiki nami start apache
  ```

# Customize this image

The Bitnami Mediawiki Docker image is designed to be extended so it can be used as the base image for your custom web applications.

## Extend this image

Before extending this image, please note there are certain configuration settings you can modify using the original image:

- Settings that can be adapted using environment variables. For instance, you can change the ports used by Apache for HTTP and HTTPS, by setting the environment variables `APACHE_HTTP_PORT_NUMBER` and `APACHE_HTTPS_PORT_NUMBER` respectively.
- [Adding custom virtual hosts](https://github.com/bitnami/bitnami-docker-apache#adding-custom-virtual-hosts).
- [Replacing the 'httpd.conf' file](https://github.com/bitnami/bitnami-docker-apache#full-configuration).
- [Using custom SSL certificates](https://github.com/bitnami/bitnami-docker-apache#using-custom-ssl-certificates).

If your desired customizations cannot be covered using the methods mentioned above, extend the image. To do so, create your own image using a Dockerfile with the format below:

```Dockerfile
FROM bitnami/mediawiki
## Put your customizations below
...
```

Here is an example of extending the image with the following modifications:

- Install the `vim` editor
- Modify the Apache configuration file
- Modify the ports used by Apache

```Dockerfile
FROM bitnami/mediawiki
LABEL maintainer "Bitnami <containers@bitnami.com>"

## Install 'vim'
RUN install_packages vim

## Enable mod_ratelimit module
RUN sed -i -r 's/#LoadModule ratelimit_module/LoadModule ratelimit_module/' /opt/bitnami/apache/conf/httpd.conf

## Modify the ports used by Apache by default
# It is also possible to change these environment variables at runtime
ENV APACHE_HTTP_PORT_NUMBER=8181
ENV APACHE_HTTPS_PORT_NUMBER=8143
EXPOSE 8181 8143
```

Based on the extended image, you can use a Docker Compose file like the one below to add other features:

```yaml
version: '2'
services:
  mariadb:
    image: 'bitnami/mariadb:10.3'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - MARIADB_USER=bn_mediawiki
      - MARIADB_DATABASE=bitnami_mediawiki
    volumes:
      - 'mariadb_data:/bitnami'
  mediawiki:
    build: .

We'd love for you to contribute to this container. You can request new features by creating an [issue](https://github.com/bitnami/bitnami-docker-mediawiki/issues), or submit a [pull request](https://github.com/bitnami/bitnami-docker-mediawiki/pulls) with your contribution.

# Issues

If you encountered a problem running this container, you can file an [issue](https://github.com/bitnami/bitnami-docker-mediawiki/issues). For us to provide better support, be sure to include the following information in your issue:

- Host OS and version
- Docker version (`$ docker version`)
- Output of `$ docker info`
- Version of this container (`$ echo $BITNAMI_IMAGE_VERSION` inside the container)
- The command you used to run the container, and any relevant output you saw (masking any sensitive information)

# License

Copyright 2016-2019 Bitnami

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

 <http://www.apache.org/licenses/LICENSE-2.0>

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
