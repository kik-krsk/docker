#Запуск
# docker run --name php-72 -p 127.0.0.1:9172:9000 -v /var/www:/var/www --add-host=host.docker.internal:host-gateway -d php-with-extensions:7.2-fpm

FROM php:7.2-fpm
LABEL description="Образ php-fpm 7.2 с oci8 и другими расширениями"

#ID пользователя nginx для доступа к файлам
ARG USER_ID=968
ARG GROUP_ID=965

# RUN выполняет идущую за ней команду в контексте нашего образа.
# В данном случае мы установим некоторые зависимости и модули PHP.
# Для установки модулей используем команду docker-php-ext-install.
# На каждый RUN создается новый слой в образе, поэтому рекомендуется объединять команды.
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    git \
    unzip \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    libonig-dev \
    libzip-dev \
    libmcrypt-dev \
    libicu-dev \
    libmagickwand-dev \
    libmagickcore-dev \
    libsnmp-dev \
    && docker-php-ext-install snmp \
    && pecl install mcrypt-1.0.3 \
    && docker-php-ext-enable mcrypt \
    && pecl install redis \
    && pecl install xdebug \
    && docker-php-ext-enable redis xdebug \
    && pecl install imagick \
    && docker-php-ext-enable imagick \
    && docker-php-ext-install -j$(nproc) iconv mbstring mysqli pdo_mysql zip intl\
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd

# Install Oracle Instantclient
RUN apt-get install -y \
    libaio1 \libaio-dev \
    && apt clean
RUN mkdir /opt/oracle
RUN wget https://download.oracle.com/otn_software/linux/instantclient/216000/instantclient-basic-linux.x64-21.6.0.0.0dbru.zip \
    && wget https://download.oracle.com/otn_software/linux/instantclient/216000/instantclient-sdk-linux.x64-21.6.0.0.0dbru.zip \
    && wget https://download.oracle.com/otn_software/linux/instantclient/216000/instantclient-sqlplus-linux.x64-21.6.0.0.0dbru.zip \
    && unzip instantclient-basic-linux.x64-21.6.0.0.0dbru.zip -d /opt/oracle \
    && unzip instantclient-sdk-linux.x64-21.6.0.0.0dbru.zip -d /opt/oracle \
    && unzip instantclient-sqlplus-linux.x64-21.6.0.0.0dbru.zip -d /opt/oracle \
    && rm -rf *.zip \
    && mv /opt/oracle/instantclient_21_6 /opt/oracle/instantclient

#add oracle instantclient path to environment
RUN echo /opt/oracle/instantclient > /etc/ld.so.conf.d/oic.conf && \
    ldconfig
# Install Oracle extensions. See https://pecl.php.net/package/oci8
RUN docker-php-ext-configure pdo_oci --with-pdo-oci=instantclient,/opt/oracle/instantclient,21.1 \
    && echo 'instantclient,/opt/oracle/instantclient/' | pecl install oci8-2.2.0 \
    && docker-php-ext-install \
    pdo_oci \
    && docker-php-ext-enable \
    oci8
# Куда же без composer'а.
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Добавим свой php.ini, можем в нем определять свои значения конфига
ADD php.ini /usr/local/etc/php/conf.d/40-custom.ini

RUN usermod -u ${USER_ID} www-data && groupmod -g ${GROUP_ID} www-data
# Указываем рабочую директорию для PHP
WORKDIR /var/www

# Запускаем контейнер
# Из документации: The main purpose of a CMD is to provide defaults for an executing container. These defaults can include an executable,
# or they can omit the executable, in which case you must specify an ENTRYPOINT instruction as well.
CMD ["php-fpm"]