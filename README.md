![symfony-docker](http://i.imgur.com/vc5ZVqL.png?2)

# Symfony + Nginx + php-fpm
[![Build Status](https://travis-ci.org/kibatic/symfony-docker.svg?branch=master)](https://travis-ci.org/kibatic/symfony-docker)
[![](https://images.microbadger.com/badges/image/kibatic/symfony:latest.svg)](https://microbadger.com/images/kibatic/symfony:latest "Get your own image badge on microbadger.com")
[![](https://images.microbadger.com/badges/version/kibatic/symfony:latest.svg)](https://microbadger.com/images/kibatic/symfony:latest "Get your own version badge on microbadger.com")


Docker for Symfony application, powered by **Nginx** and **php-fpm**.

Based on Debian Jessie.

If you are experiencing some issues, take a look at [TROUBLESHOOTING](TROUBLESHOOTING.md)

### Supported tags and respective `Dockerfile` links

`7`, `7.1` [(7.1/Dockerfile)](https://github.com/kibatic/symfony-docker/blob/master/7.1/Dockerfile)

`7.0` [(7.0/Dockerfile)](https://github.com/kibatic/symfony-docker/blob/master/7.0/Dockerfile)

`5`, `5.6`, `latest` [(5.6/Dockerfile)](https://github.com/kibatic/symfony-docker/blob/master/5.6/Dockerfile)

`5.4` [(5.4/Dockerfile)](https://github.com/kibatic/symfony-docker/blob/master/5.4/Dockerfile)

### Usage

```bash
docker pull kibatic/symfony
```

Then run in your symfony folder

```bash
docker run -v $(pwd):/var/www -p 8080:80 kibatic/symfony
```

Symfony app will be accessible on http://localhost:8080/app.php

### Custom nginx configuration

If you want to replace the default nginx settings, overwrite configuration file at `/etc/nginx/sites-enabled/default`.

```dockerfile
COPY nginx.conf /etc/nginx/sites-enabled/default
```

You may also want to add only some directives in [existing site config](7.1/rootfs/etc/nginx/sites-enabled/default#L5).

```dockerfile
COPY custom-config.conf /etc/nginx/conf.d/docker/custom-config.conf
```

### Logging

A common practice is to log to stdout, but there are major bug in php-fpm wich makes stdout logging not reliable  :

* Logs are truncated when message length exceed 1024 https://bugs.php.net/bug.php?id=69031
* FPM prepend a warning string to worker output https://bugs.php.net/bug.php?id=71880

This image setup a known workaround ([see here](https://github.com/docker-library/php/issues/207)) and expose a log stream as env var **LOG_STREAM**, but **you cannot log to stdout**
For a proper logging you have to configure monolog to log to this stream

```yaml
# app/config_dev.yml
monolog:
    handlers:
        main:
            type:   stream
            path:   '/tmp/stdout'
            level:  debug
```

You can also use symfony `%env(LOG_STREAM)%` if your symfony version is compatible with [this syntax](https://symfony.com/doc/3.4/configuration/external_parameters.html)

We also provide a default dirty solution for standard monolog configuration, **this is not recommended in production**

```bash
tail -q -n 0 -F app/logs/dev.log app/logs/prod.log var/logs/dev.log var/logs/prod.log
```

### Minimal package included

* nginx
* php\*-fpm
* php\*-cli
* php\*-intl
* php\*-mbstring

### Exposed port
* 80 : nginx
