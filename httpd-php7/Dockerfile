FROM debian:jessie

MAINTAINER Rob McVey https://github.com/robmcvey

ENV PHP_VERSION 7.1.1
ENV PHP_FILENAME php-7.1.1.tar.xz
ENV PHP_INI_DIR /usr/local/etc/php
ENV PHP_SHA256 b3565b0c1441064eba204821608df1ec7367abff881286898d900c2c2a5ffe70
ENV GPG_KEYS A917B1ECDA84AEC2B568FED6F50ABC807BD5DCD0 528995BFEDFBA7191D46839EF9BA0ADA31CBD89E
ENV REDIS_FILENAME redis-3.1.0.tar.gz
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
    locales \
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
    mysql-client \
    vim \
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
        --with-apxs2=/usr/bin/apxs \
        --with-config-file-path="$PHP_INI_DIR" \
        --with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
        --disable-cgi \
        --enable-mysqlnd \
        --with-pdo-mysql \
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

RUN curl -fSL "https://codeload.github.com/phpredis/phpredis/tar.gz/3.1.0" -o "$REDIS_FILENAME" \
    && mkdir redis \
    && tar -xvzf "$REDIS_FILENAME" -C redis \
    && rm "$REDIS_FILENAME" \
    && cd redis/phpredis-3.1.0 \
    && phpize \
    && ./configure \
    && make && make install

RUN echo "extension=redis.so" > /usr/local/etc/php/conf.d/redis.ini

COPY apache2-foreground /usr/local/bin/
WORKDIR $WWW_ROOT

CMD ["apache2-foreground"]
