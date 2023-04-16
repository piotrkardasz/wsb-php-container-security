ARG PHP_VERSION=8.1
ARG APCU_VERSION=5.1.22

FROM php:${PHP_VERSION}-fpm-alpine3.17

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
