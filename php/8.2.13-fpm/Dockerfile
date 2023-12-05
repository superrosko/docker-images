FROM php:8.2.13-fpm
MAINTAINER Dmitriy Bespalov <dbespalov.rosko@gmail.com>

# Set timezone
ENV TZ=Europe/Moscow
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Install dependencies
RUN apt-get update && apt-get install -y \
    zip \
    unzip \
    git\
    curl \
    build-essential \
    locales \
    procps \
    dnsutils \
    nano \
    libpq-dev\
    libzip-dev \
    libmpc-dev \
    libicu-dev \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    libmemcached-dev


# PHP Extensions
RUN apt-get update && apt-get install -y \
    && docker-php-ext-configure zip \
    && docker-php-ext-configure gmp \
    && docker-php-ext-configure intl \
    && docker-php-ext-install -j$(nproc) \
        opcache \
        pdo_mysql \
        pdo_pgsql \
        bcmath \
        pcntl\
        zip \
        gmp \
        intl

RUN apt-get update && apt-get install -y \
    && pecl install xdebug && docker-php-ext-enable xdebug \
    && pecl install redis && docker-php-ext-enable redis \
    && pecl install memcached && docker-php-ext-enable memcached \
    && pecl install mongodb && docker-php-ext-enable mongodb


RUN apt-get update && apt-get install -y \
    && docker-php-ext-configure gd \
        --with-jpeg\
        --with-freetype \
    && docker-php-ext-install -j$(nproc) \
        gd
# PHP Extensions


# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*
RUN apt-get autoremove -y

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
