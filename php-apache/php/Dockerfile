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
RUN docker-php-ext-install \
    intl mysqli pdo pdo_mysql \
    gd \
    && pecl install imagick \
	&& docker-php-ext-enable imagick pdo

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
    RUN mkdir /opt/oracle \
    && cd /opt/oracle 
    ADD instantclient-basic-linux.x64-12.2.0.1.0.zip /opt/oracle
    ADD instantclient-sdk-linux.x64-12.2.0.1.0.zip /opt/oracle
    ADD instantclient-sqlplus-linux.x64-12.2.0.1.0.zip /opt/oracle

    # ENV ORACLE_HOME /opt/oracle/instantclient
    RUN  unzip /opt/oracle/instantclient-basic-linux.x64-12.2.0.1.0.zip -d /opt/oracle \
    && unzip /opt/oracle/instantclient-sdk-linux.x64-12.2.0.1.0.zip -d /opt/oracle \
    && ln -s /opt/oracle/instantclient_12_2/libclntsh.so.12.1 /opt/oracle/instantclient_12_2/libclntsh.so \
    && ln -s /opt/oracle/instantclient_12_2/libclntshcore.so.12.1 /opt/oracle/instantclient_12_2/libclntshcore.so \
    && ln -s /opt/oracle/instantclient_12_2/libocci.so.12.1 /opt/oracle/instantclient_12_2/libocci.so \
    && rm -rf /opt/oracle/*.zip

    ENV LD_LIBRARY_PATH  /opt/oracle/instantclient_12_2:${LD_LIBRARY_PATH}
    # Install Oracle extensions
    RUN echo 'instantclient,/opt/oracle/instantclient_12_2/' | pecl install oci8-2.2.0 \ 
    && docker-php-ext-enable \
        oci8 \ 
    && docker-php-ext-configure pdo_oci --with-pdo-oci=instantclient,/opt/oracle/instantclient_12_2,12.2 \
    && docker-php-ext-install \
        pdo_oci 
# # Add oracle instantclient path to environment
#     RUN echo /opt/oracle/instantclient/ > /etc/ld.so.conf.d/oracle.conf \
#         && ldconfig
# # Install Oracle extensions
#     RUN docker-php-ext-configure pdo_oci --with-pdo-oci=instantclient,/opt/oracle/instantclient \
#     && echo 'instantclient,/opt/oracle/instantclient/' | pecl install oci8-2.2.0 \
#     && docker-php-ext-install \
#         pdo_oci \
#     && docker-php-ext-enable \
#         oci8

## Add
ADD . /var/www/html