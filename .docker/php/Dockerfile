ARG UID
ARG GID

FROM php:8-fpm-alpine as base
ARG UID=${UID:-1000}
ARG GID=${GID:-1000}

WORKDIR /app

RUN apk update && apk add --no-cache libzip-dev icu-dev curl-dev tzdata; \
    if [ ! $(getent group ${GID}) ]; then addgroup -g ${GID} app-group; fi; \
    mkdir -p /home/app; \
    adduser -D -h /home/app -s /bin/ash -u ${UID} -G $(getent group ${GID} | cut -d: -f1) app-user; \
    chown -R ${UID}:${GID} /home/app; \
    echo "user = ${UID}" >> /usr/local/etc/php-fpm.d/www.conf; \
    echo "group = ${GID}" >> /usr/local/etc/php-fpm.d/www.conf;

RUN apk add --no-cache git ssh wget; \
    docker-php-ext-install pdo_mysql zip intl curl; \
    apk del gcc g++;

RUN chown -R $UID:$GID /app

FROM base as dev
ARG XDEBUG="false"

COPY --from=composer:2 /usr/bin/composer /usr/local/bin/composer

RUN if [ $XDEBUG = "true" ]; then \
        pecl install xdebug; \
        docker-php-ext-enable xdebug; \
        touch /var/log/xdebug.log; \
        echo "error_reporting = E_ALL" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
        echo "display_startup_errors = On" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
        echo "display_errors = On" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
        echo "xdebug.mode = develop,debug,coverage" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
        echo "xdebug.start_with_request=yes" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
        echo "xdebug.client_host=$XDEBUG_CLIENT" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
        echo "xdebug.client_port=9003" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
        echo "xdebug.log_level = 7" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
        echo "xdebug.log=/var/log/xdebug.log" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini; \
    fi; \
    # Set system timezone to Europe/Amsterdam
    rm /etc/localtime; ln -s /usr/share/zoneinfo/Europe/Amsterdam /etc/localtime

FROM base as builder

COPY . .
COPY --from=composer:2 /usr/bin/composer /usr/local/bin/composer

RUN composer install
