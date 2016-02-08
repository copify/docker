FROM debian:jessie

MAINTAINER Rob McVey https://github.com/robmcvey

ENV WWW_ROOT /var/www/html

RUN apt-get update && apt-get install -y \
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

COPY apache2.conf /etc/apache2/apache2.conf
COPY apache2-foreground /usr/local/bin/

WORKDIR $WWW_ROOT

CMD ["apache2-foreground"]
