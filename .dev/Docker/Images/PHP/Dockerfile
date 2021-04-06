ARG VERSION
FROM php:${VERSION}-cli

RUN apt update && apt upgrade -y && \
    apt install -y libxml2-dev wget

# Copy over our script to check if the extension is already installed
COPY ./docker-php-ext-install-check.sh /opt/php-ext-check.sh

RUN apt-get install -y \
    libzip-dev \
    zip

# Check and/or Install core modules
RUN /opt/php-ext-check.sh bcmath ctype fileinfo json mbstring openssl tokenizer xml zip curl

# Install PHP Database Extensions
RUN /opt/php-ext-check.sh pdo

## Install PHP MySQL Extensions
RUN /opt/php-ext-check.sh mysql mysqli pdo_mysql

## Install PHP ODBC Extensions
RUN set -ex; \
        docker-php-source extract; \
        { \
                echo '# https://github.com/docker-library/php/issues/103#issuecomment-271413933'; \
                echo 'AC_DEFUN([PHP_ALWAYS_SHARED],[])dnl'; \
                echo; \
                cat /usr/src/php/ext/odbc/config.m4; \
        } > temp.m4; \
        mv temp.m4 /usr/src/php/ext/odbc/config.m4; \
        apt-get update; \
        apt-get install -y --no-install-recommends unixodbc-dev; \
        docker-php-ext-configure odbc --with-unixODBC=shared,/usr; \
        docker-php-ext-install odbc; \
        docker-php-source delete; \
        apt-get purge -y --auto-remove

## Install PHP MSSQL Extensions
RUN pecl install sqlsrv pdo_sqlsrv

# Install Composer
# e3e43bde99447de1c13da5d1027545be81736b27 = 04DEC2020
RUN wget https://raw.githubusercontent.com/composer/getcomposer.org/e3e43bde99447de1c13da5d1027545be81736b27/web/installer -O - -q | php -- --quiet
RUN mv composer.phar /usr/local/bin/composer

# Install Xdebug
RUN pecl install xdebug \
    && echo "zend_extension=$(find /usr/local/lib/php/extensions/ -name xdebug.so)" > /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_enable=on" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_autostart=off" >> /usr/local/etc/php/conf.d/xdebug.ini

WORKDIR var/www/html
