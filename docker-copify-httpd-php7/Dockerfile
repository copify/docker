FROM debian:jessie

MAINTAINER Rob McVey https://github.com/robmcvey

ENV PHP_VERSION 7.0.2
ENV PHP_FILENAME php-7.0.2.tar.xz
ENV PHP_INI_DIR /usr/local/etc/php
ENV PHP_SHA256 556121271a34c442b48e3d7fa3d3bbb4413d91897abbb92aaeced4a7df5f2ab2
ENV GPG_KEYS 1A4E8B7277C42E53DBA9C7B9BCAA30EA9C0D5763
ENV WWW_ROOT /var/www/html

RUN apt-get update && apt-get install -y \
	ca-certificates \
	curl \
	librecode0 \
	libxml2 \
	autoconf \
    zlib1g-dev \
	file \
	libcurl4-openssl-dev \
	libreadline6-dev \
	librecode-dev \
	libssl-dev \
	libicu-dev	\
	libmcrypt-dev	\
	libxml2-dev \
	xz-utils \
	g++ \
    git-core \
	gcc \
	libc-dev \
	make \
	pkg-config \
	re2c \
    apache2-dev \
	apache2-bin \
	apache2.2-common \
	--no-install-recommends && rm -r /var/lib/apt/lists/*

RUN rm -rf /var/www/html
RUN mkdir -p /var/lock/apache2 /var/run/apache2 /var/log/apache2 /var/www/html
RUN chown -R www-data:www-data /var/lock/apache2 /var/run/apache2 /var/log/apache2 /var/www/html
RUN a2dismod mpm_event && a2enmod mpm_prefork && a2enmod rewrite
RUN mv /etc/apache2/apache2.conf /etc/apache2/apache2.conf.dist
RUN rm /etc/apache2/conf-enabled/* /etc/apache2/sites-enabled/*

# Do this BEFORE php install as PHP compile steps look for this
COPY apache2.conf /etc/apache2/apache2.conf

RUN mkdir -p $PHP_INI_DIR/conf.d
RUN set -xe \
	&& for key in $GPG_KEYS; do \
		gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
	done
RUN set -x \
	&& curl -fSL "http://php.net/get/$PHP_FILENAME/from/this/mirror" -o "$PHP_FILENAME" \
	&& echo "$PHP_SHA256 *$PHP_FILENAME" | sha256sum -c - \
	&& curl -fSL "http://php.net/get/$PHP_FILENAME.asc/from/this/mirror" -o "$PHP_FILENAME.asc" \
	&& gpg --verify "$PHP_FILENAME.asc" \
	&& mkdir -p /usr/src/php \
	&& tar -xf "$PHP_FILENAME" -C /usr/src/php --strip-components=1 \
	&& rm "$PHP_FILENAME"* \
	&& cd /usr/src/php \
	&& ./configure \
		--with-config-file-path="$PHP_INI_DIR" \
		--with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
		--disable-cgi \
		--enable-mysqlnd \
		--enable-intl \
        --enable-zip \
		--enable-mbstring \
		--with-curl \
		--with-openssl \
		--with-readline \
		--with-mcrypt \
		--with-recode \
		--with-zlib \
	&& make -j"$(nproc)" \
	&& make install \
	&& { find /usr/local/bin /usr/local/sbin -type f -executable -exec strip --strip-all '{}' + || true; } \
	&& make clean

RUN curl -sS https://getcomposer.org/installer | php
RUN mv composer.phar /usr/local/bin/composer

COPY apache2-foreground /usr/local/bin/
WORKDIR $WWW_ROOT

CMD ["apache2-foreground"]
