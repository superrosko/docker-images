FROM php:8.5.7-fpm
LABEL maintainer="Dmitriy Bespalov <dbespalov.rosko@gmail.com>"

ENV XDEBUG_VERSION=3.5.3 \
    REDIS_VERSION=6.3.0 \
    IGBINARY_VERSION=3.2.17RC1 \
    MSGPACK_VERSION=3.0.1 \
    MEMCACHED_VERSION=3.4.0 \
    MONGODB_VERSION=2.3.3

# Set timezone
ENV TZ=Europe/Moscow
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Install dependencies
RUN set -eux; \
    savedAptMark="$(apt-mark showmanual)"; \
    echo "$savedAptMark" > /tmp/saved-apt-mark; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        zip \
        unzip \
        git \
        curl \
        locales \
        procps \
        dnsutils \
        nano \
        libfreetype6 \
        libgmp10 \
        libhashkit2 \
        libicu76 \
        libjpeg62-turbo \
        libmemcached11 \
        libmemcached-dev \
        libpng16-16 \
        libpq5 \
        libsasl2-2 \
        libzip5 \
        zlib1g \
        $PHPIZE_DEPS \
        build-essential \
        libpq-dev \
        libzip-dev \
        libgmp-dev \
        libicu-dev \
        libpng-dev \
        libjpeg-dev \
        libfreetype6-dev \
        libmemcached-dev \
        zlib1g-dev \
    ; \
    rm -rf /var/lib/apt/lists/*

# Install bundled PHP extensions
RUN set -eux; \
    docker-php-ext-configure zip --with-zip; \
    docker-php-ext-configure gmp --with-gmp; \
    docker-php-ext-configure intl --enable-intl; \
    docker-php-ext-install -j$(nproc) \
        pdo_mysql \
        pdo_pgsql \
        bcmath \
        pcntl \
        zip \
        gmp \
        intl \
    ; \
    docker-php-ext-configure gd \
        --with-jpeg \
        --with-freetype; \
    docker-php-ext-install -j$(nproc) gd

# Install PECL extensions
RUN set -eux; \
    pecl install \
        xdebug-${XDEBUG_VERSION} \
        redis-${REDIS_VERSION} \
        igbinary-${IGBINARY_VERSION} \
        msgpack-${MSGPACK_VERSION} \
    ; \
    docker-php-ext-enable xdebug redis igbinary msgpack; \
    pecl install \
        memcached-${MEMCACHED_VERSION} \
        mongodb-${MONGODB_VERSION} \
    ; \
    docker-php-ext-enable memcached mongodb; \
    rm -rf /tmp/pear ~/.pearrc

# Remove build-only dependencies
RUN set -eux; \
    savedAptMark="$(cat /tmp/saved-apt-mark)"; \
    apt-mark auto '.*' > /dev/null; \
    apt-mark manual \
        $savedAptMark \
        zip \
        unzip \
        git \
        curl \
        locales \
        procps \
        dnsutils \
        nano \
        libfreetype6 \
        libgmp10 \
        libhashkit2 \
        libicu76 \
        libjpeg62-turbo \
        libmemcached11 \
        libmemcached-dev \
        libpng16-16 \
        libpq5 \
        libsasl2-2 \
        libzip5 \
        zlib1g \
    ; \
    ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
        | awk '/=>/ { print $3 }' \
        | sort -u \
        | xargs -r dpkg-query -S \
        | cut -d: -f1 \
        | sort -u \
        | xargs -r apt-mark manual; \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
    rm -f /tmp/saved-apt-mark; \
    rm -rf /var/lib/apt/lists/*

# Keep libmemcached runtime libraries after build dependency cleanup
RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends libmemcached-dev; \
    rm -rf /var/lib/apt/lists/*

# Verify installed extensions
RUN set -eux; \
    php -v; \
    php -r '$extensions = ["bcmath", "gd", "gmp", "igbinary", "intl", "memcached", "mongodb", "msgpack", "pcntl", "pdo_mysql", "pdo_pgsql", "redis", "xdebug", "zip"]; foreach ($extensions as $extension) { if (!extension_loaded($extension)) { fwrite(STDERR, "Missing PHP extension: $extension\n"); exit(1); } }'

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
