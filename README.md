## !!!This repository is in WIP state

This is an example Wordpress application that works with [Epinio](https://github.com/epinio/epinio).
It is meant to be used as a guide to anyone who wants to deploy Wordpress on Kubernetes using Epinio.

## Step 1 - Create a cluster

- k3d cluster create epinio -p 80:80@server[0] -p 443:443@server[0] --k3s-server-arg --disable --k3s-server-arg traefik

## Step 2 - Download epinio cli

- wget it
- make it executable
- put it in the path
- setup autocompletion

## Step 3 - Install epinio

- epinio install --system-domain your_domain

## Step 4 - Download and prepare Wordpress

NOTE: https://github.com/paketo-buildpacks/php-web#configuring-custom-ini-files

Update wp-config.php based on the [wp-config.php.sample file](wp-config.php.sample).

## Step 5 - Create a database for Wordpress

- Enable incluster services
- List plans
- Provision Mariadb

## Step 6 - Push wordpress and bind the database service

## Step 7 - Visit the wordpress route and finish the installation

## Step 8 - Logging and scaling
