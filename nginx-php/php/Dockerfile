FROM php:7.4-fpm
# ARG UNAME=teeraphat
# ARG UGROUP=_www
# ARG UID=1000
# ARG GID=1001
# RUN usermod  --uid $UNAME
# RUN groupmod --gid $UGROUP

RUN docker-php-ext-install mysqli
RUN apt-get update && apt-get install -y \
		libfreetype6-dev \
		libjpeg62-turbo-dev \
		libpng-dev \
	&& docker-php-ext-configure gd --with-freetype --with-jpeg \
	&& docker-php-ext-install -j$(nproc) gd