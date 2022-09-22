This is an example Wordpress application that works with [Epinio](https://github.com/epinio/epinio).
It is meant to be used as a guide to anyone who wants to deploy Wordpress on Kubernetes using Epinio.

The [wordpress directory](wordpress/) in this repository is the extracted Wordpress zip archive.
In order to deploy Wordpress to Epinio you are going to need:

- The Wordpress sources
- An epinio installation
- A MySQL service running and accessible from your Wordpress app
- A wp-config.php file inside the wordpress directory configured for access to the mysql service
- A [.php.ini.d/extensions.ini](.php.ini.d/extensions.ini)
  file to configure the PHP buildpack for wordpress ([More here](https://github.com/paketo-buildpacks/php-web#configuring-custom-ini-files))

## Step 1 - Create a cluster

```bash
bash> k3d cluster create epinio
```

## Step 2 - Download epinio cli

Find the artifact that matches your OS and architecture by visiting the latest
release here: https://github.com/epinio/epinio/releases

You need to download that binary and put it in your PATH. Something like this
should work on Linux (replace the link with right one for your binary):

```bash
# Download the binary
bash> wget https://github.com/epinio/epinio/releases/download/v1.2.0/epinio-linux-x86_64
# Make the binary executable
bash> chmod +x epinio-linux-x86_64
# Put epinio in your PATH
bash> mv epinio-linux-x86_64 /usr/bin/epinio
# Enable epinio autocompletion
bash> epinio completion bash > comp
bash> source comp
```

## Step 3 - Install epinio

Follow the Epinio installation guide [here](https://docs.epinio.io/installation)

## Step 4 - Create a database for Wordpress

Wordpress needs a database to work. After visiting the route of your deployed application you will have to set the connection details to the database.

You can install a MySQL database on your cluster or use an external one. One option is using a helm chart like this one: https://bitnami.com/stack/mysql/helm

Also you may use the ones already available on our catalog services:

```bash
bash> epinio service catalog

#🚢  Getting catalog...

#✔️  Epinio Services:
#|      NAME      |            CREATED             | VERSION |          DESCRIPTION           |
#|----------------|--------------------------------|---------|--------------------------------|
#| postgresql-dev | 2022-09-22 12:45:05 +0200 CEST | 14.2.0  | A PostgreSQL service that can  |
#|                |                                |         | be used during development     |
#| rabbitmq-dev   | 2022-09-22 12:45:05 +0200 CEST | 3.9.17  | A RabbitMQ service that can be |
#|                |                                |         | used during development        |
#| mysql-dev      | 2022-09-22 12:45:05 +0200 CEST | 8.0.29  | A MySQL service that can be    |
#|                |                                |         | used during development        |
#| mongodb-dev    | 2022-09-22 12:45:05 +0200 CEST | 6.0.1   | A MongoDB service that can be  |
#|                |                                |         | used during development        |
#| redis-dev      | 2022-09-22 12:45:05 +0200 CEST | 6.2.7   | A Redis service that can be    |
#|                |                                |         | used during development        |
```
More info about our Service Catalog here: https://docs.epinio.io/references/customization/catalog

For this example, we proceed with `mysql-dev`:
```bash
bash> epinio service create mysql-dev mydb 
```
## Step 5 - Download and prepare Wordpress

Since you are using this repository, you can skip this step. The wordpress
sources are already in the [wordpress directory](wordpress).

If you want to start from scratch you can get the latest Wordpress here:

https://wordpress.org/download/#download-install

## Step 6 - Prepare the source code to work with Epinio

You need to enable two PHP plugins that are needed for
Wordpress and for MySQL:  zlib and mysqli

This happens with the [.php.ini.d/extensions.ini](.php.ini.d/extensions.ini) file
in this repository.

Learn more here: https://github.com/paketo-buildpacks/php-web#configuring-custom-ini-files

Finally you need to let Wordpress how to connect you the database you created.
This happens by copying the [wordpress/wp-config-sample.php](wordpress/wp-config-sample.php)
to `wordpress/wp-config.php` and editing the relevant `DB_` settings.
This repository already has a [wordpress/wp-config.php](wordpress/wp-config.php) file
that should work if you didn't change the name of the database in the steps above.

## Step 6 - Create application and bind the database service

You can now create the application:
```bash
# Create the application
bash> epinio apps create wordpress
# Bind the database to the app
bash> epinio service bind mydb wordpress
```

## Step 7 - Push the application

You can now push Wordpress with one command:

```bash
bash> epinio push -n wordpress -e BP_PHP_VERSION=8.0.x -e BP_PHP_SERVER=nginx -e BP_PHP_WEB_DIR=wordpress \
      -e CONFIG_NAME=$(epinio configurations list | grep mydb | awk '{print $2}')
```

## Step 8 - Visit the wordpress route and finish the installation

You can now visit the application's route in your browser and follow the Wordpress
installation wizard to finish the installation.

You may have to accept the self-signed certificate your application is served with.
Epinio can create production level certificates for you automatically but that is
ouside the scope of this guide. Visit the [Epinio repository](https://github.com/epinio/epinio)
if you want to know more.
