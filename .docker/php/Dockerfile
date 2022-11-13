ARG PHP_VERSION=8.1
ARG APCU_VERSION=5.1.22

#https://github.com/docker-library/php/blob/master/8.1/bullseye/fpm/Dockerfile
FROM php:${PHP_VERSION}-fpm-bullseye as php-debian

ARG APCU_VERSION

RUN set -eux; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	apt-get update && apt-get upgrade -y; \
	apt-get install -y --no-install-recommends \
		libfreetype6-dev \
		libicu-dev \
		libjpeg62-turbo-dev \
		libpng-dev \
		libwebp-dev \
		libxpm-dev \
		libzip-dev \
	; \
	\
 	docker-php-ext-configure gd --with-jpeg --with-webp --with-freetype; \
	docker-php-ext-configure zip --with-zip; \
	docker-php-ext-install -j$(nproc) \
		exif \
		gd \
		intl \
		opcache \
		pdo_mysql \
	 	sockets \
		zip \
		; \
	#https://pear.php.net/manual/en/guide.developers.package2.pecl.php \
	pecl install --configureoptions 'enable-apcu-debug="no"' \
		apcu-${APCU_VERSION} \
	; \
	docker-php-ext-enable \
		apcu --ini-name 10-docker-php-ext-apcu.ini \
	; \
	pecl clear-cache; \
	docker-php-source delete; \
	\
# reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
	apt-mark auto '.*' > /dev/null; \
	[ -z "$savedAptMark" ] || apt-mark manual $savedAptMark; \
	find /usr/local -type f -executable -exec ldd '{}' ';' \
		| awk '/=>/ { print $(NF-1) }' \
		| sort -u \
		| xargs -r dpkg-query --search \
		| cut -d: -f1 \
		| sort -u \
		| xargs -r apt-mark manual \
	; \
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	rm -rf /var/lib/apt/lists/*;

#https://github.com/moby/moby/issues/2259#issuecomment-48286811
RUN mkdir -p /var/www/public/media/image && chown -R www-data:www-data /var/www/public/media
VOLUME /var/www/public/media

COPY --from=composer/composer:2-bin /composer /usr/bin/composer
COPY ./php.ini		$PHP_INI_DIR/php.ini

WORKDIR /var/www


#https://github.com/docker-library/php/blob/master/8.1/alpine3.16/fpm/Dockerfile
FROM php:${PHP_VERSION}-fpm-alpine3.16 as php-alpine

ARG APCU_VERSION

# install gnu-libiconv and set LD_PRELOAD env to make iconv work fully on Alpine image.
# see https://github.com/docker-library/php/issues/240#issuecomment-763112749
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so

RUN set -eux; \
	 apk upgrade; \
	apk add --no-cache --virtual .build-deps \
		$PHPIZE_DEPS \
		freetype-dev \
		icu-dev \
		libjpeg-turbo-dev \
		libpng-dev \
		libtool \
		libwebp-dev \
		libzip-dev \
		zlib-dev \
	; \
	\
 	docker-php-ext-configure gd --with-jpeg --with-webp --with-freetype; \
	docker-php-ext-configure zip --with-zip; \
	docker-php-ext-install -j$(nproc) \
		exif \
		gd \
		intl \
		opcache \
		pdo_mysql \
	 	sockets \
		zip \
		; \
	#https://pear.php.net/manual/en/guide.developers.package2.pecl.php \
	pecl install --configureoptions 'enable-apcu-debug="no"' \
		apcu-${APCU_VERSION} \
	; \
	docker-php-ext-enable \
		apcu --ini-name 10-docker-php-ext-apcu.ini \
	; \
	pecl clear-cache; \
	docker-php-source delete; \
	\
	runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
			| tr ',' '\n' \
			| sort -u \
			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)"; \
	apk add --no-cache $runDeps; \
	\
	apk del --no-network .build-deps;

#https://github.com/moby/moby/issues/2259#issuecomment-48286811
RUN mkdir -p /var/www/public/media/image && chown -R www-data:www-data /var/www/public/media
VOLUME /var/www/public/media

COPY --from=composer/composer:2-bin /composer /usr/bin/composer
COPY ./php.ini $PHP_INI_DIR/php.ini

WORKDIR /var/www


FROM debian:bullseye-slim as php-sury-debian

ARG PHP_VERSION
ARG APCU_VERSION

ENV PHP_DEPS \
		php8.1-common \
		php8.1-cli \
		php8.1-fpm \
		php8.1-apcu \
		php8.1-opcache \
		php8.1-gd \
		php8.1-curl \
		php8.1-zip \
		php8.1-intl \
		php8.1-mbstring \
		php8.1-mysql \
		php8.1-xml

RUN set -eux; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		ca-certificates \
		curl \
		xz-utils \
	; \
	rm -rf /var/lib/apt/lists/*

RUN set -eux; \
	\
	savedAptMark="$(apt-mark showmanual) $(echo $PHP_DEPS)"; \
	apt-get update && apt-get upgrade -y; \
	apt-get install -y --no-install-recommends lsb-release ca-certificates gnupg2 wget; \
	echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/sury-php.list; \
	wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		$PHP_DEPS \
	; \
	## reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
	apt-mark auto '.*' > /dev/null; \
	[ -z "$savedAptMark" ] || apt-mark manual $savedAptMark; \
	find /usr/local -type f -executable -exec ldd '{}' ';' \
		| awk '/=>/ { print $(NF-1) }' \
		| sort -u \
		| xargs -r dpkg-query --search \
		| cut -d: -f1 \
		| sort -u \
		| xargs -r apt-mark manual \
	; \
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	rm -rf /var/lib/apt/lists/*;

RUN ln -sf /usr/sbin/php-fpm${PHP_VERSION} /usr/bin/php-fpm; \
	ln -sf /usr/bin/php${PHP_VERSION} /usr/bin/php;

#https://github.com/moby/moby/issues/2259#issuecomment-48286811
RUN mkdir -p /var/www/public/media/image && chown -R www-data:www-data /var/www/public/media
VOLUME /var/www/public/media

RUN set -eux; \
	mkdir -p /run/php; \
	cd /etc/php/${PHP_VERSION}/fpm/pool.d; \
	{ \
		echo '[global]'; \
		echo 'error_log = /proc/self/fd/2'; \
		echo; echo '; https://github.com/docker-library/php/pull/725#issuecomment-443540114'; echo 'log_limit = 8192'; \
		echo; \
		echo '[www]'; \
		echo '; if we send this to /proc/self/fd/1, it never appears'; \
		echo 'access.log = /proc/self/fd/2'; \
		echo; \
		echo 'clear_env = no'; \
		echo; \
		echo '; Ensure worker stdout and stderr are sent to the main error log.'; \
		echo 'catch_workers_output = yes'; \
		echo 'decorate_workers_output = no'; \
	} | tee docker.conf; \
	{ \
		echo '[global]'; \
		echo 'daemonize = no'; \
		echo; \
		echo '[www]'; \
		echo 'listen = 9000'; \
	} | tee zz-docker.conf

COPY --from=composer/composer:2-bin /composer /usr/bin/composer
COPY ./php.ini /etc/php/${PHP_VERSION}/fpm/
COPY ./php.ini /etc/php/${PHP_VERSION}/cli/

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000

FROM ubuntu:jammy as php-sury-ubuntu

ARG PHP_VERSION
ARG APCU_VERSION

ENV PHP_DEPS \
		php8.1-common \
		php8.1-cli \
		php8.1-fpm \
		php8.1-apcu \
		php8.1-opcache \
		php8.1-gd \
		php8.1-curl \
		php8.1-zip \
		php8.1-intl \
		php8.1-mbstring \
		php8.1-mysql \
		php8.1-xml

RUN set -eux; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		ca-certificates \
		curl \
		xz-utils \
	; \
	rm -rf /var/lib/apt/lists/*

RUN set -eux; \
	\
	savedAptMark="$(apt-mark showmanual) $(echo $PHP_DEPS)"; \
	apt-get update && apt-get upgrade -y; \
	apt install -y software-properties-common; \
    add-apt-repository ppa:ondrej/php -y; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		$PHP_DEPS \
	; \
	## reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
	apt-mark auto '.*' > /dev/null; \
	[ -z "$savedAptMark" ] || apt-mark manual $savedAptMark; \
	find /usr/local -type f -executable -exec ldd '{}' ';' \
		| awk '/=>/ { print $(NF-1) }' \
		| sort -u \
		| xargs -r dpkg-query --search \
		| cut -d: -f1 \
		| sort -u \
		| xargs -r apt-mark manual \
	; \
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	rm -rf /var/lib/apt/lists/*;

RUN ln -sf /usr/sbin/php-fpm${PHP_VERSION} /usr/bin/php-fpm; \
	ln -sf /usr/bin/php${PHP_VERSION} /usr/bin/php;

#https://github.com/moby/moby/issues/2259#issuecomment-48286811
RUN mkdir -p /var/www/public/media/image && chown -R www-data:www-data /var/www/public/media
VOLUME /var/www/public/media

RUN set -eux; \
	 mkdir -p /run/php; \
	cd /etc/php/${PHP_VERSION}/fpm/pool.d; \
	{ \
		echo '[global]'; \
		echo 'error_log = /proc/self/fd/2'; \
		echo; echo '; https://github.com/docker-library/php/pull/725#issuecomment-443540114'; echo 'log_limit = 8192'; \
		echo; \
		echo '[www]'; \
		echo '; if we send this to /proc/self/fd/1, it never appears'; \
		echo 'access.log = /proc/self/fd/2'; \
		echo; \
		echo 'clear_env = no'; \
		echo; \
		echo '; Ensure worker stdout and stderr are sent to the main error log.'; \
		echo 'catch_workers_output = yes'; \
		echo 'decorate_workers_output = no'; \
	} | tee docker.conf; \
	{ \
		echo '[global]'; \
		echo 'daemonize = no'; \
		echo; \
		echo '[www]'; \
		echo 'listen = 9000'; \
	} | tee zz-docker.conf

COPY --from=composer/composer:2-bin /composer /usr/bin/composer
COPY ./php.ini /etc/php/${PHP_VERSION}/fpm/
COPY ./php.ini /etc/php/${PHP_VERSION}/cli/

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000


FROM piotrkardasz/php-distroless:${PHP_VERSION}-debug-amd64-debian11 as php-distroless

COPY --from=composer/composer:2-bin /composer /usr/bin/composer
COPY ./php.ini /etc/php/${PHP_VERSION}/fpm/
COPY ./php.ini /etc/php/${PHP_VERSION}/cli/
