FROM php:8.1-fpm-bullseye as php-debian

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends gnupg \
        zlib1g-dev \
        libzip-dev \
        libxml2-dev \
        libicu-dev \
        libwebp-dev \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        libxpm-dev

RUN docker-php-ext-install -j$(nproc) intl exif opcache pdo_mysql \
    && docker-php-ext-configure gd \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-source delete

#https://github.com/moby/moby/issues/2259#issuecomment-48286811
RUN mkdir -p /var/www/public/media/image && chown -R www-data:www-data /var/www/public/media
VOLUME /var/www/public/media

COPY --from=composer/composer:2-bin /composer /usr/bin/composer
COPY ./php.ini       $PHP_INI_DIR/php.ini

WORKDIR /var/www

FROM piotrkardasz/php-distroless:8.1-debug-amd64-debian11 as php-distroless

COPY --from=composer/composer:2-bin /composer /usr/bin/composer
