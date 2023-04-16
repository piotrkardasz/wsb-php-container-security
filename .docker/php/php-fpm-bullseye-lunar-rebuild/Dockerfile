ARG PHP_VERSION=8.1
ARG APCU_VERSION=5.1.22

FROM piotrkardasz/sylius-distroless-2023:php-fpm-lunar-base

ARG APCU_VERSION

ENV PHP_DYNAMIC_DEPS \
		libfreetype6 \
		libicu72 \
		libjpeg-turbo8 \
		libpng16-16 \
		libwebp7 \
		libxpm4 \
		libzip4

RUN set -eux; \
	\
	savedAptMark="$(apt-mark showmanual) $(echo $PHP_DYNAMIC_DEPS)"; \
	apt-get update && apt-get upgrade -y; \
	apt-get install -y --no-install-recommends \
		libfreetype6-dev \
		libicu-dev \
		libjpeg-turbo8-dev \
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
