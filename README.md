# DockerWeb

A docker-powered bash script for shared web hosting management. The ultimate Docker LAMP/LEMP Stack.

## Requirements

- A fresh installation of latest Ubuntu LTS x64 with the FQDN set.
	- DigitalOcean provides Cloud VPS Servers at affordable prices. You can sign up via [this referral link](https://m.do.co/c/fb7b38f12520) to get a free $100 bonus in credit.
- At least 512MB of system memory. (1GB recommended)
- A domain name pointed at the host's IP address. (Optional but highly recommended)

## Installing

After you changed the default root password and ssh port, to install the latest stable release, you can run the following commands as root:

```
git clone https://github.com/fffaraz/dockerweb.git /opt/dockerweb
export PATH=$PATH:/opt/dockerweb
```

To add `/opt/dockerweb` to your $PATH permanently run `echo 'export PATH=$PATH:/opt/dockerweb' >> ~/.profile`

To create swap space on a regular file run `docweb install:swapfile SIZE_IN_MB`

If you don't already have docker installed on your host run `docweb install:docker`

To create the network bridge used by dockerweb containers run `docweb install:network`

Finally to build dockerweb images run `docweb build:all`

Optionally, to install some useful packages run `docweb install:extras` and `docweb install:aliases`

### Upgrading

To upgrade you just need to run `docweb upgrade` first and then if necessary rebuild all the images with `docweb build:all` and re-run all the containers to use new images built.

## How to run services

### The DNS Server

[MicroDNS](https://github.com/fffaraz/microdns) is a tiny DNS server that is used to (almost) always return your host's IP address for any query sent to it. Hence you will just need to set your domains' dns setting to point to your host's IP address and you don't have to also add them in a config file on the host any more.

```
docweb microdns:run [--ttl 3600] [--log]
docweb microdns:log
docweb stop microdns
```

### The Reverse Proxy Server

This is the main web server listening on ports 80 and 443 and proxying requests to the apps based on their domain names.
It also provides TLS/SSL encryption using valid certificates signed by [Let's Encrypt](https://letsencrypt.org/) and is responsible for output gzip compression.
You can add your apps to its config file at `/home/proxy/websites.conf`.
You need to run the update command each time you edit the config file.
Make sure all your apps are running before updating the proxy server.

```
docweb proxy:run [--debug] [SERVER_NAME]
docweb stop proxy
docweb proxy:update
docweb proxy:status
```

Logs Directory: `/home/proxy/log/nginx/`

Config File Format: `CONTAINER CATCHALL WILDCARD SSLCERT DOMAIN1 [DOMAINS...]`

* `CONTAINER` : The name of App's container.
* `CATCHALL` : Set to 1 for the default app. Proxies all unmatched domain names to that container. Optional. One app only.
* `WILDCARD` : Whether to also proxy all subdomains of app's domain names to that container.
* `SSLCERT` : Request for a valid TLS/SSL certificate from letsencrypt.
* `HTTPSONLY` : Redirect all HTTP requests to HTTPS.
* `DOMAIN1` : Primary domain name for the app.
* `[DOMAINS...]` : Optional additional domain names.

### MySQL Server

```
docweb mysql:run [--debug] [--direct PORT] [--memory M] NAME MYSQL_ROOT_PASSWORD [mariadb|mysql|mysql/mysql-server]
docweb mysql:stop NAME
docweb mysql:client NAME
docweb mysql:status NAME
docweb mysql:optimize NAME
docweb mysql:backup NAME [--gzip] [DATABASES...]
docweb mysql:import NAME [--gzip] FILENAME [DATABASE]
docweb mysql:createuser NAME USERNAME PASSWORD
docweb mysql:createdb NAME DB
docweb mysql:listusers NAME
docweb mysql:listproc NAME
docweb mysql:listdbs NAME
docweb mysql:upgrade NAME
docweb mysql:tuner NAME MYSQL_ROOT_PASSWORD
```

Default Image: `mariadb`

MySQL Backup / Import Directory: `/home/NAME/backup`

### PostgreSQL Server

```
docweb postgres:run [--debug] NAME POSTGRES_PASSWORD
docweb stop NAME
```

### Redis Server

```
docweb redis:run [--debug] NAME
docweb stop NAME
```

### Mail Forwarder

```
docweb mailfwd:run
```

Based on [zixia/docker-simple-mail-forwarder](https://github.com/zixia/docker-simple-mail-forwarder). Config File: `/home/mailfwd/mail.conf`

### Web Apps

```
docweb run [--debug|--bash] [--direct PORT] IMAGE NAME [WEBUSER_PASSWORD SSH_PORT]
docweb stop NAME
docweb logs NAME
docweb update NAME
docweb exec [--root] NAME
docweb clean NAME
docweb backup NAME
docweb ps NAME
docweb status NAME
docweb log:nginx NAME [--error]
docweb log:php NAME [--error]
```

WWW (public_hmtl) root directory: `/home/NAME/www/public`

Domains config file: `/home/NAME/domains.conf`

Optional project init script: `/home/NAME/project.sh`

List of availabe images: ` php7nginx php7apache php5apache python3 nodejs aspnet4 aspnet5 `

### phpMyAdmin

```
docweb pma:run [--debug] [--direct PORT] MYSQLSERVER MYSQLROOTPASSWORD [pma status] [CaptchaPublic CaptchaPrivate]
docweb stop MYSQLSERVER_pma
```

Then add the following line to your `/home/proxy/websites.conf` and then run `dockweb proxy:update`.

```
MYSQLSERVER_pma 0 0 0 0 pma.example.com
```

phpMyAdmin can be accessed from `http://pma.example.com/pma/`

Status page `http://pma.example.com/status/`

### ClamAV

```
docweb clamav:run
```

### Etc.

* `docweb rm` To remove stopped containers, orphaned images and orphaned volumes.
* `docweb rm:all` To remove all containers and volumes.
* `docweb rm:img` To remove all images.
* `docweb build NAME` To build an image.
* `docweb stats` Prints host status.
* `docweb volume:create NAME SIZE`

## Examples

### PHP 7.1 + NGINX + MySQL Stack

To create a new PHP website with NGINX webserver and MariaDB database server and phpMyAdmin run the following commands:

```
docweb mysql:run db1 mysqlrootpassword
docweb mysql:createuser db1 example.com db_password
docweb pma:run db1 mysqlrootpassword
docweb run php7nginx example.com
docweb proxy:run
```

Then add the following lines to `/home/proxy/websites.conf`

```
db1_pma 0 0 0 0 pma.example.com
example.com 0 0 0 0 example.com
```

Then update the proxy server with `docweb proxy:update`

## See Also

* [docker-webserver](https://github.com/nazar-pc/docker-webserver) - WebServer (MariaDB, PHP-FPM, Nginx) composed from several separate containers linked together.
* [shipping-docker](https://github.com/shipping-docker)

<!---
## Donation

If this helps you [why not buy me a beer](https://www.paypal.me/frz288) :beer:

If this project help you reduce time to develop and you would like to donate to this cause, feel free to send money my way via one of the links below:

[![PayPal](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.me/frz288)
[![Venmo](https://img.shields.io/badge/Donate-Venmo-green.svg)](https://venmo.com/?txn=pay&audience=public&recipients=fffaraz&note=DockerWeb)
[![Square](https://img.shields.io/badge/Donate-Square-green.svg)](https://cash.me/$fffaraz)
[![SayThanks](https://img.shields.io/badge/SayThanks.io-%E2%98%BC-1EAEDB.svg)](https://saythanks.io/to/fffaraz)
-->
