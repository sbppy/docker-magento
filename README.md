Mark Shust's Docker Configuration for Magento

## Docker Hub

View Dockerfiles:

- [markoshust/magento-nginx (Docker Hub)](https://hub.docker.com/r/markoshust/magento-nginx/)
	- 1.13
		- [`latest`, `1.13`, `1.13-2`](https://github.com/markoshust/docker-magento/tree/master/images/nginx/1.13)
		- [`1.13-1`](https://github.com/markoshust/docker-magento/tree/11.1.5/images/nginx/1.13)
		- [`1.13-0`](https://github.com/markoshust/docker-magento/tree/11.0.0/images/nginx/1.13)
- [markoshust/magento-php (Docker Hub)](https://hub.docker.com/r/markoshust/magento-php/)
	- 7.1
		- [`latest`, `7.1-fpm`, `7.1-fpm-2`](https://github.com/markoshust/docker-magento/tree/master/images/php/7.1)
		- [`7.1-fpm-1`](https://github.com/markoshust/docker-magento/tree/11.1.5/images/php/7.1)
		- [`7.1-fpm-0`](https://github.com/markoshust/docker-magento/tree/11.0.0/images/php/7.1)
	- 7.0
		- [`7.0-fpm`, `7.0-fpm-2`](https://github.com/markoshust/docker-magento/tree/master/images/php/7.0)
		- [`7.0-fpm-1`](https://github.com/markoshust/docker-magento/tree/11.1.5/images/php/7.0)
		- [`7.0-fpm-0`](https://github.com/markoshust/docker-magento/tree/11.0.0/images/php/7.0)
	- 5.6
		- [`5.6-fpm`, `5.6-fpm-2`](https://github.com/markoshust/docker-magento/tree/master/images/php/5.6)
		- [`5.6-fpm-1`](https://github.com/markoshust/docker-magento/tree/11.1.5/images/php/5.6)
		- [`5.6-fpm-0`](https://github.com/markoshust/docker-magento/tree/11.0.0/images/php/5.6)

## Usage

This configuration is intended to be used as a Docker-based development environment for both Magento 1 and Magento 2.

Folders:

- `images`: Docker images for nginx and php
- `compose`: sample setups with Docker Compose

Nginx assumes you are running Magento 2, however you can easily run it with Magento 1 using [the provided configuration file](https://github.com/markoshust/docker-magento/blob/master/images/nginx/1.13/conf/default.magento1.conf). Here is an [example of this setup with Docker Compose](https://github.com/markoshust/docker-magento/tree/master/compose/magento-1).

The PHP images are fairly agnostic to which version of Magento you are running. The PHP 5 images do assume you are running Magento 1, and the PHP 7 images do assume you are running Magento 2, however the main difference is cronjob setup, and they can be easily modified for inverse usage.

## Prerequisites

This setup assumes you are running Docker on a computer with at least 16GB RAM, a quad-core, and an SSD hard drive. [Download & Install Docker Community Edition](https://www.docker.com/community-edition#/download).

This configuration has been tested on Mac, but should also work on Mac, Windows and Linux.

If you are using a Mac, it is strongly recommended for you to apply [these performance tuning changes](http://markshust.com/2018/01/30/performance-tuning-docker-mac) to Docker for Mac before starting.

## Setup a New Magento 2 Project

1. Setup a new project using the Magento 2 compose skeleton:

```
mkdir magento2 && cd $_
git init
git remote add origin git@github.com:markoshust/docker-magento.git
git fetch origin
git checkout origin/master -- compose/magento-2
mv compose/magento-2/* .
rm -rf compose .git
git init
```

2. Download the Magento source code to the `src` folder with: `./bin/download 2.2.2`

3. Setup your ip loopback for proper IP resolution with Docker: `./bin/initloopback`

4. Add an entry to `/etc/hosts` with your custom domain: `10.254.254.254 magento2.test` (assuming the domain  you want to setup is `magento2.test`). Be sure to use a `.test` tld, as `.localhost` and `.dev` will present issues with domain resolution.

5. Start your Docker containers with: `./bin/start`.

6. Run Magento's setup install process with the command: `./bin/setup`. Feel free to edit this file to your liking; at the very least you will probably need to update the `base-url` value to the domain you setup in step 6.

7. You may now access your site at `http://magento2.test` (or whatever domain you setup).

## Existing Magento Project Setup

See the `compose` folder for sample setups for both Magento 1 and Magento 2. Basically your source code should go in the `src` folder, and you can then kick your project off with `./bin/start`. You may have to complete a few of the steps above to get things functioning properly.

## Custom CLI Commands

- `./bin/bash`: Drop into the bash prompt of your Docker container. The `phpfpm` container should be mainly used to access the filesystem within Docker.
- `./bin/cli`: Run any CLI command without going into the bash prompt. Ex. `./bin/cli ls`
- `./bin/composer`: Run the composer binary. Ex. `./bin/composer install`
- `./bin/download`: Download a version of Magento to the `src` directory. Ex. `./bin/download 2.2.2`
- `./bin/fixperms`: This will fix filesystem ownerships and permissions within Docker.
- `./bin/initloopback`: Setup your ip loopback for proper Docker ip resolution.
- `./bin/magento`: Run the Magento CLI. Ex: `./bin/magento cache:flush`
- `./bin/setup`: Run the Magento setup process to install Magento from the source code.
- `./bin/start`: Start the Docker Compose process and your app. Ctrl+C to stop the process.
- `./bin/xdebug`: Disable or enable Xdebug. Ex. `./bin/xdebug enable`

## Misc Info

### Database

- The hostname of each service is the name of the service within the `docker-compose.yml` file. So for example, MySQL's hostname is `db` (not `localhost`) when accessing it from a Docker container.

### PHPStorm & Xdebug

Within Xdebug, create a new `PHPStorm > Preferences > Languages & Frameworks > PHP > CLI Interpreter` and specify `From Docker`. Choose `Docker`, then select the `markoshust/magento-php:7-0-fpm` image name, and the `PHP Executable` to be `php`. Hitting the reload executable button should find the correct PHP Version and Xdebug debugger configuration.

Open `PHPStorm > Preferences > Languages & Frameworks > PHP > Debug` and set:

- IDE key: `PHPSTORM`
- Host: `10.254.254.254`
- Port: `9001`

Create a new server at  `PHPStorm > Preferences > Languages & Frameworks > PHP > Servers`. Set `localhost` as the name and host, check `Shared`, leave port `80`, and debugger `Xdebug`. Check `Use path mappings` and assigned the `src` File/Directory to the absolute path on the server of `/var/www/html`.

Create a new `PHP Remote Debug` configuration at `Run > Edit Configurations`. Name it `localhost`. Check `Filter debug connection by IDE Key`, select server `localhost`, and set IDE key to `PHPSTORM`.

Open up `src/pub/index.php`, and set a breakpoint near the end of the file. Go to `Run > Debug localhost`, and open up a web browser. Be sure to install a plugin like [Xdebug helper](https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc) which sets the IDE key to `PHPStorm` automatically for you. Enable the browser extension and activate it on the site, and reload the site. Xdebug within PHPStorm should now enable the debugger and stop at the toggled breakpoint.

### Composer Authentication

Please first setup Magento Marketplace authentication (details in the [DevDocs](http://devdocs.magento.com/guides/v2.0/install-gde/prereq/connect-auth.html)).

Place your auth token at `~/.composer/auth.json` with the following contents, like so:

```
{
    "http-basic": {
        "repo.magento.com": {
            "username": "MAGENTO_PUBLIC_KEY",
            "password": "MAGENTO_PRIVATE_KEY"
        }
    }
}
```
