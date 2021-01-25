FROM php:7.4-apache

# Install composer 
ADD install-composer.sh /root/
RUN /root/install-composer.sh

COPY mi_api/ /var/www/html/
COPY .htaccess /var/www/html/

RUN set -eux; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
        unzip \
        git; \
    rm -rf /var/lib/apt/lists/*
RUN composer install

RUN a2enmod rewrite