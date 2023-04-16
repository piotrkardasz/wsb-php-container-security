ARG PHP_VERSION=8.1
ARG APCU_VERSION=5.1.22

FROM piotrkardasz/php-distroless-2023:${PHP_VERSION}-nonroot-amd64-debian11

COPY --from=composer/composer:2-bin /composer /usr/bin/composer
COPY ./php.ini /etc/php/${PHP_VERSION}/fpm/
COPY ./php.ini /etc/php/${PHP_VERSION}/cli/
