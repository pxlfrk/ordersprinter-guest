[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/misery/ordersprinter/blob/main/LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com)
[![Open Source Love svg1](https://badges.frapsoft.com/os/v1/open-source.svg?v=103)](https://github.com/ellerbrock/open-source-badges/)

📢 Note: This project only deploys an optional component of ordersprinter, not the required core-system. Use the regular ubuntu installation or the [ordersprinter-project](https://github.com/misery/ordersprinter) to deploy the core-system via docker.

---

# Ordersprinter-Gastbestellsystem in Docker

This repository can be used to simplify the deployment and installation of [Ordersprinter Gastbestellsystem](https://www.ordersprinter.de/gastbestellung.php), an optional component of [Ordersprinter ](https://www.ordersprinter.de) via docker compose.

## Deployment

Based on the [official documentation](https://www.ordersprinter.de/gastbestellung.php), the guestorder-system should be installed separately to the core-system with his own webserver and database. Therefore i recommend deploying the guestorder-system as a separate docker stack in addition to the core-system ordersprinter.

You can either deploy this stack manually or use an container/stack management solution like portainer etc.

### Best Practice

Since this application is required to be internet-facing, so that your customers can reach it from their mobile devices, you should apply additional safety measures.

> [!WARNING]  
> I stronlgy recommend adding a revery proxy and ssl certificates in front of it. In this example (and also in my personal use-case), the `nginx`-Container is added to specific docker network `npm`, in which also the [nginxproxymanager](https://nginxproxymanager.com/) as my preferred reverse proxy is located. But of course, you can choose whichever solution fits your need the best. The web is full of tutorials and guides how to set this up, so i won't cover this here.

> [!NOTE]  
> For easier access during installation, you can modify the `compose.yaml` to your needs and at a ports-mapping to the nginx-container, so that the web-ui is reachable via a dedicated port, e.g.

``` yaml
nginx:
    ...
    ports: # disable in production and use revery-proxy instead!
    - '8080:80'
`````

### Manual Deployment

1. Checkout out this repository and create an `.env` file in the directory- or rename the existing `.env.template`-File and adapt it to your needs.

``` env
MYSQL_DB=ordersprintguest_db
MYSQL_USER=ordersprintguest
MYSQL_PASSWORD=pleaseUseASuperSecurePasswordReplaceMe
MYSQL_ROOT_PASSWORD=pleaseUseASuperSecurePasswordReplaceMe
DB=mysql
CODE=123456
COMPOSE_PROJECT_NAME=ordersprintguest # will be prefixed to your container-names!
```

1. Provide your own database settings via ``MYSQL_`` variable. This will be used during initial deployment.

> [!NOTE]  
> Technically it isn't necessary to use a database as storage option. If you set the value of `DB` to `file`, the system will save your orders in the `db` inside of the container. **Make sure to persist this data by mounting a docker volume, otherwise your data will be lost when the container is recreated or deleted.**

> [!NOTE]  
> Since the image does **NOT** contain the sources of Ordersprinter it will download the newest version and create a volume for this.

**DO NOT** remove MariaDB/MySQL volume! Otherwise you should have a backup somewhere else!

1. Start up: `docker compose up -d --build`

The Web-Interface is now accessible via `http://YOUR-DOMAIN`.

### Portainer

If you use [Portainer](https://github.com/portainer/portainer) for your Docker host, you can create a stack via a git repository, too.

`Stacks -> Add stack -> Use a git repository`

Now you can provide the following information.

- Name: `ordersprinter-guestordersystem` (or whatever you like)
- Repository URL: `https://github.com/pxlfrk/ordersprinter-guest`
- Compose path: `docker-compose.yml`
- Repository reference: `refs/heads/main`
- Environment variables: See ``.env`` file settings above
- Enable relative path volumes: `true/enabled`

> [!NOTE]  
> [Enable relative path volumes](https://docs.portainer.io/user/docker/stacks/add#relative-path-volumes) required the Portainer Business Edition, yes - BUT you can get yourself a free license for up to three nodes 👉 [here](https://www.portainer.io/take-3). 🎉

> [!NOTE]  
> Switch the Environment-Variables to the Advances mode, paste the content of the `.env.template` and change the values to your fit - this will save you some time!

> [!NOTE]  
> If you run Portainer as a container you need to bind mount the data directory.
> Otherwise Docker cannot access the local files from the git repository for the new container.
> [Tell me more!](https://github.com/misery/ordersprinter#portainer-as-container)

## Setup

1. After the compose stack is running, head to your installation - either via IP & Port or your configured Domain (via a revery proxy) or any other alternatives
1. Go to the URL `domain.tld/`**install.php** and you should see the setup-screen.
1. Enter your CODE (specified in your `.env`)
1. After success, remove the `install.php` file from your installation.

## Updating

If you want to update, delete/empty the `data`-volume and re-deploy the stack. It will automatically
fetch the newest version. After that you'll need to run through the setup process again, so make sure to save your credentials entered in the `.env` file.
As long as you don't delete/drop your database, all order-specific informations will be kept.

> [!WARNING]  
> If you don't use the database as storage backend, copy all files in the `db` folder if you don't want to loose them!

## Special thanks to 🎉

- [André Klitzing (aka. misery)](https://github.com/misery/) for the inspiration and extensive work in his [ordersprinter project](https://github.com/misery/ordersprinter)
- Stefan Pichel for creating and open-sourcing ordersprinter under [Creative-Commons BY-NC-ND](https://www.ordersprinter.de/lizenz_en.php)
