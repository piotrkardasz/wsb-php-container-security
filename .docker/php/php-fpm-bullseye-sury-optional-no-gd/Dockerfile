ARG PHP_VERSION=8.1
ARG APCU_VERSION=5.1.22

FROM debian:bullseye-slim

ARG PHP_VERSION
ARG APCU_VERSION

ENV PHP_DEPS \
		php8.1-common \
		php8.1-cli \
		php8.1-fpm \
		php8.1-apcu \
		php8.1-opcache \
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
