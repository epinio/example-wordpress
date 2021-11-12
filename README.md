This is an example Wordpress application that works with [Epinio](https://github.com/epinio/epinio).
It is meant to be used as a guide to anyone who wants to deploy Wordpress on Kubernetes using Epinio.

The [wordpress directory](wordpress/) in this repository is the extracted Wordpress zip archive.
In order to deploy Wordpress to Epinio you are going to need:

- The Wordpress sources
- An epinio installation
- A mariadb service running and accessible from your Wordpress app
- A wp-config.php file inside the wordpress directory configured for access to the Mariadb service
- A [buildpack.yml](buildpack.yml) and a [.php.ini.d/extensions.ini](.php.ini.d/extensions.ini)
  file to configure the PHP buildpack for wordpress ([More here](https://github.com/paketo-buildpacks/php-web#configuring-custom-ini-files))

## Step 1 - Create a cluster

```bash
bash> k3d cluster create epinio -p 80:80@server[0] -p 443:443@server[0] --k3s-server-arg --disable --k3s-server-arg traefik
```

## Step 2 - Download epinio cli

Find the artifact that matches your OS and architecture by visiting the latest
release here: https://github.com/epinio/epinio/releases

You need to download that binary and put it in your PATH. Something like this
should work on Linux (replace the link with right one for your binary):

```bash
# Download the binary
bash> wget https://github.com/epinio/epinio/releases/download/v0.1.6/epinio-linux-amd64
# Make the binary executable
bash> chmod +x epinio-linux-amd64
# Put epinio in your PATH
bash> mv epinio-linux-amd64 /usr/bin/epinio
# Enable epinio autocompletion
bash> epinio comp bash > comp
bash> source comp
```

## Step 3 - Install epinio

```bash
bash> epinio install
```

## Step 4 - Create a database for Wordpress

```bash
# Enable in-cluster services (Minibroker)
bash> epinio enable services-incluster
# Wait until the following command returns some Mariadb plans
bash> watch epinio service list-plans mariadb # CTRL+C when ready
# Create a service instance of Mariadb called "wordpress" with a database "wordpress"
# automatically created.
# This is the actual value we are setting behind the scenes:
# https://github.com/helm/charts/blob/2771293/stable/mariadb/values.yaml#L134
bash> epinio service create wordpress mariadb 10-3-22 --data '{ "db": { "name": "wordpress" }}'
```

## Step 5 - Download and prepare Wordpress

Since you are using this repository, you can skip this step. The wordpress
sources are already in the [wordpress directory](wordpress).

If you want to start from scratch you can get the latest Wordpress here:

https://wordpress.org/download/#download-install

## Step 6 - Prepare the source code to work with Epinio

You are going to need to configure the PHP buildpack to work with Epinio. This is
done with the [buildpack.yml](buildpack.yml) file in this repository which defines
what script the app should start from, which web server to use, where the application
files live and what PHP version to use.
The other thing you need to do is to enable two PHP plugins that are needed for
Wordpress and for Mariadb:  zlib and mysqli

This happens with the [.php.ini.d/extensions.ini](.php.ini.d/extensions.ini) file
in this repository.

Learn more here: https://github.com/paketo-buildpacks/php-web#configuring-custom-ini-files

Finally you need to let Wordpress how to connect you the database you created.
This happens by copying the [wordpress/wp-config-sample.php](wordpress/wp-config-sample.php)
to `wordpress/wp-config.php` and editing the relevant `DB_` settings.
This repository already has a [wordpress/wp-config.php](wordpress/wp-config.php) file
that should work if you didn't change the name of the database in the steps above.

## Step 6 - Push wordpress and bind the database service

You can now push Wordpress with one command:

```bash
bash> epinio push -n wordpress -b wordpress
```

## Step 7 - Visit the wordpress route and finish the installation

You can now visit the application's route in your browser and follow the Wordpress
installation wizard to finish the installation.

You may have to accept the self-signed certificate your application is served with.
Epinio can create production level certificates for you automatically but that is
ouside the scope of this guide. Visit the [Epinio repository](https://github.com/epinio/epinio)
if you want to know more.
