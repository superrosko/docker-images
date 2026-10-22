FROM php:8.3.36-fpm
LABEL maintainer="Dmitriy Bespalov <dbespalov.rosko@gmail.com>"

ENV XDEBUG_VERSION=3.5.3 \
    REDIS_VERSION=6.3.0 \
    IGBINARY_VERSION=3.2.17RC1 \
    MSGPACK_VERSION=3.0.1 \
    MEMCACHED_VERSION=3.4.0 \
    MONGODB_VERSION=2.3.3 \
    TZ=Europe/Moscow

RUN set -eux; \
    ln -snf "/usr/share/zoneinfo/$TZ" /etc/localtime; \
    echo "$TZ" > /etc/timezone; \
    savedAptMark="$(apt-mark showmanual)"; \
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
        $PHPIZE_DEPS \
        libpq-dev \
        libzip-dev \
        libgmp-dev \
        libicu-dev \
        libpng-dev \
        libjpeg-dev \
        libfreetype6-dev \
        libmemcached-dev \
        libssl-dev \
        zlib1g-dev \
    ; \
    docker-php-ext-configure zip; \
    docker-php-ext-configure gmp; \
    docker-php-ext-configure intl; \
    docker-php-ext-configure gd \
        --with-jpeg \
        --with-freetype; \
    docker-php-ext-install -j"$(nproc)" \
        opcache \
        pdo_mysql \
        pdo_pgsql \
        bcmath \
        pcntl \
        zip \
        gmp \
        intl \
        gd \
    ; \
    install_php_extension() { \
        repository="$1"; \
        tag="$2"; \
        extension="$3"; \
        source_url="${4:-https://codeload.github.com/${repository}/tar.gz/refs/tags/${tag}}"; \
        archive="/tmp/${extension}.tgz"; \
        source_dir="/tmp/${extension}-source"; \
        echo "Installing PHP extension: ${extension} (${repository}@${tag})"; \
        curl -fsSL --connect-timeout 15 --max-time 180 \
            --retry 5 --retry-delay 2 --retry-all-errors \
            -o "$archive" "$source_url"; \
        mkdir "$source_dir"; \
        tar -xzf "$archive" -C "$source_dir" --strip-components=1; \
        cd "$source_dir"; \
        phpize; \
        ./configure; \
        make -j"$(nproc)"; \
        make install; \
        cd /; \
        rm -rf "$archive" "$source_dir"; \
    }; \
    install_php_extension xdebug/xdebug "$XDEBUG_VERSION" xdebug; \
    install_php_extension igbinary/igbinary "$IGBINARY_VERSION" igbinary; \
    install_php_extension msgpack/msgpack-php "v$MSGPACK_VERSION" msgpack; \
    docker-php-ext-enable xdebug igbinary msgpack; \
    install_php_extension phpredis/phpredis "$REDIS_VERSION" redis; \
    install_php_extension php-memcached-dev/php-memcached "v$MEMCACHED_VERSION" memcached; \
    install_php_extension mongodb/mongo-php-driver "$MONGODB_VERSION" mongodb \
        "https://github.com/mongodb/mongo-php-driver/releases/download/$MONGODB_VERSION/mongodb-$MONGODB_VERSION.tgz"; \
    docker-php-ext-enable redis memcached mongodb; \
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
    ; \
    ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
        | awk '/=>/ { print $3 }' \
        | sort -u \
        | xargs -r readlink -f \
        | xargs -r dpkg-query -S \
        | cut -d: -f1 \
        | sort -u \
        | xargs -r apt-mark manual; \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
    rm -rf /tmp/pear ~/.pearrc /var/lib/apt/lists/*

RUN set -eux; \
    php -v; \
    php -r '$extensions = ["bcmath", "gd", "gmp", "igbinary", "intl", "memcached", "mongodb", "msgpack", "Zend OPcache", "pcntl", "pdo_mysql", "pdo_pgsql", "redis", "xdebug", "zip"]; foreach ($extensions as $extension) { if (!extension_loaded($extension)) { fwrite(STDERR, "Missing PHP extension: $extension\n"); exit(1); } }'

RUN curl -fsSL https://getcomposer.org/installer \
    | php -- --install-dir=/usr/local/bin --filename=composer

