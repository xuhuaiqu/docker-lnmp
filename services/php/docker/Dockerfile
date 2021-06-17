FROM php:7.0.33-fpm

#解决debain升级停止维护造成的库不存在的问题
RUN sed -i '/jessie-updates/d' /etc/apt/sources.list

# 系统的库
RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        librabbitmq-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libldap2-dev \
        libicu-dev \
        libxml2-dev \
        libbz2-dev \
        libpcre3-dev \
        curl \
        libcurl3-dev \
        git \
        wget \
        gdb \
        vim \
        --no-install-recommends

# PHP核心扩展
RUN docker-php-ext-install bcmath mcrypt zip intl gettext soap sockets pcntl sysvsem sysvshm mbstring mysqli pdo_mysql xmlrpc gd

#PECL扩展
RUN pecl install mongodb yar amqp redis \
    && rm -rf /tmp/pear
RUN docker-php-ext-enable mongodb yar amqp redis 

RUN apt-get purge -y g++ \
    && apt-get autoremove -y \
    && rm -r /var/lib/apt/lists/* \
    && rm -rf /tmp/*

#php composer
RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer \
    && chmod +x /usr/local/bin/composer

RUN usermod -u 1000 www-data

EXPOSE 9000
CMD ["php-fpm"]
