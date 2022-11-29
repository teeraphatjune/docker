FROM --platform=linux/amd64 php:7.4-apache
# --platform=linux/amd64
LABEL maintainer Teeraphat Youngyeun <teeraphat.june@gmail.com>

RUN a2enmod rewrite
RUN apt-get update \
    && apt-get install -y wget \
    unzip \
    gnupg \
    curl \
    zip \
    vim \
    unixodbc \
    unixodbc-dev \
    libmagickwand-dev --no-install-recommends
RUN apt-get install libaio1 

# Install normal extension php
RUN pecl install apcu
RUN docker-php-ext-install \
    intl mysqli pdo pdo_mysql \
    gd \
    && pecl install imagick \
	&& docker-php-ext-enable imagick pdo apcu

## Install sqlsrv driver v18 according to msodbcsql18 below
RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        gnupg \
    ; \
    \
    curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -; \
    curl https://packages.microsoft.com/config/debian/10/prod.list > /etc/apt/sources.list.d/mssql-release.list; \
    \
    apt-get update; \
    ACCEPT_EULA=Y apt-get install -y \
        msodbcsql18
RUN pecl install sqlsrv
RUN pecl install pdo_sqlsrv
RUN docker-php-ext-enable sqlsrv pdo_sqlsrv

## Install odbc driver
RUN set -ex; \
	docker-php-source extract; \
	{ \
        ## Comment source
		echo '# https://github.com/docker-library/php/issues/103#issuecomment-271413933'; \
		echo 'AC_DEFUN([PHP_ALWAYS_SHARED],[])dnl'; \
		echo; \
		cat /usr/src/php/ext/odbc/config.m4; \
	} > temp.m4; \
	mv temp.m4 /usr/src/php/ext/odbc/config.m4; \
	docker-php-ext-configure odbc --with-unixODBC=shared,/usr; \
	docker-php-ext-install odbc; \
	docker-php-source delete
    #apt-get purge -y --auto-remove

# Install Oracle Instantclient
    RUN mkdir /opt/oracle
    ENV ORACLE_HOME /opt/oracle/instantclient
    RUN wget https://download.oracle.com/otn_software/linux/instantclient/185000/instantclient-basic-linux.x64-18.5.0.0.0dbru.zip \
    && wget https://download.oracle.com/otn_software/linux/instantclient/185000/instantclient-sdk-linux.x64-18.5.0.0.0dbru.zip \
    && wget https://download.oracle.com/otn_software/linux/instantclient/185000/instantclient-sqlplus-linux.x64-18.5.0.0.0dbru.zip \
    && unzip instantclient-basic-linux.x64-18.5.0.0.0dbru.zip -d /opt/oracle \
    && unzip instantclient-sdk-linux.x64-18.5.0.0.0dbru.zip -d /opt/oracle \
    && unzip instantclient-sqlplus-linux.x64-18.5.0.0.0dbru.zip -d /opt/oracle \
    && rm -rf *.zip \
    && mv /opt/oracle/instantclient_18_5 /opt/oracle/instantclient
# Add oracle instantclient path to environment
    RUN echo /opt/oracle/instantclient/ > /etc/ld.so.conf.d/oracle.conf \
        && ldconfig
# Install Oracle extensions
    RUN docker-php-ext-configure pdo_oci --with-pdo-oci=instantclient,/opt/oracle/instantclient \
    && echo 'instantclient,/opt/oracle/instantclient/' | pecl install oci8-2.2.0 \
    && docker-php-ext-install \
        pdo_oci \
    && docker-php-ext-enable \
        oci8

## Add
ADD . /var/www/html