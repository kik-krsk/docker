#Запуск
# docker run --name php-56 -p 127.0.0.1:9156:9000 -v /var/www:/var/www --add-host=host.docker.internal:host-gateway  -d php-with-extensions:5.6-fpm
# или docker-compose up -d

FROM php:5.6-fpm
LABEL description="Образ php-fpm 5.6 с расширениями"

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
    libicu-dev \
    libmagickwand-dev \
    libmagickcore-dev

RUN pecl install xdebug-2.5.5 \
    docker-php-ext-enable xdebug-2.5.5 \
    && pecl install imagick \
    && docker-php-ext-enable imagick \
    && docker-php-ext-install -j$(nproc) iconv mbstring mysqli pdo_mysql zip intl\
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd

# Install Oracle Instantclient
RUN apt-get install -y \
    libaio1 \libaio-dev
RUN mkdir /opt/oracle
RUN wget  https://raw.githubusercontent.com/caffeinalab/php-fpm-oci8/master/oracle/instantclient-basic-linux.x64-12.2.0.1.0.zip \
    && wget https://raw.githubusercontent.com/caffeinalab/php-fpm-oci8/master/oracle/instantclient-sdk-linux.x64-12.2.0.1.0.zip \
    && wget https://raw.githubusercontent.com/caffeinalab/php-fpm-oci8/master/oracle/instantclient-sqlplus-linux.x64-12.2.0.1.0.zip \
    && unzip instantclient-basic-linux.x64-12.2.0.1.0.zip -d /opt/oracle \
    && unzip instantclient-sdk-linux.x64-12.2.0.1.0.zip -d /opt/oracle \
    && unzip instantclient-sqlplus-linux.x64-12.2.0.1.0.zip -d /opt/oracle \
    && rm -rf *.zip \
    && mv /opt/oracle/instantclient_12_2 /opt/oracle/instantclient \
    && ln -s /opt/oracle/instantclient/libclntsh.so.12.1 /opt/oracle/instantclient/libclntsh.so \
    && ln -s /opt/oracle/instantclient/libocci.so.12.1 /opt/oracle/instantclient/libocci.so \
    && echo /opt/oracle/instantclient >> /etc/ld.so.conf.d/oic.conf \
    && ldconfig \
#  2) OCI8
    && echo 'instantclient,/opt/oracle/instantclient' | pecl install oci8-2.0.12 \
#  2) PDO_OCI
    && pecl channel-update pear.php.net \
    && cd /tmp \
    && pecl download pdo_oci \
    && tar xvf /tmp/PDO_OCI-1.0.tgz -C /tmp \
    && sed 's/function_entry/zend_function_entry/' -i /tmp/PDO_OCI-1.0/pdo_oci.c \
    && sed 's/10.1/12.1/' -i /tmp/PDO_OCI-1.0/config.m4 \
    && cd /tmp/PDO_OCI-1.0 \
    && phpize \
    && docker-php-ext-configure pdo_oci --with-pdo-oci=instantclient,/opt/oracle/instantclient,12.1 \
    && docker-php-ext-install pdo_oci \
    && docker-php-ext-enable oci8 \
    && apt clean \
    && rm -rf /tmp/*
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
