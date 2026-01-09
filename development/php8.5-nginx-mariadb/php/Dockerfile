FROM php:8.5-fpm

# Install php libs

RUN set -ex; \
    \
    aptMarkList="$(apt-mark showmanual)"; \
    \
    apt-get update; \
    # Install php libs
    apt-get install -y --no-install-recommends \
    libpq-dev \
    libpng-dev \
    libjpeg-dev \
    libwebp-dev \
    libfreetype6-dev \
    libzip-dev \
    libxml2-dev \
    libldap2-dev \
    libzmq5-dev \
    zlib1g-dev \
    ; \
    \
    # Install php-zmq
    cd /usr; \
    curl -fSL https://github.com/zeromq/php-zmq/archive/94920ac64398901175dc4372a4297817122c52ce.tar.gz -o php-zmq.tar.gz; \
    tar -zxf php-zmq.tar.gz; \
    cd php-zmq*; \    
    phpize && ./configure; \
    make; \
    make install; \
    cd .. && rm -rf php-zmq*; \
    # END: Install php-zmq
    \
    # Install ext-ev
    pecl install ev; \
    docker-php-ext-enable ev; \
    # END: Install ext-ev
    \
    docker-php-ext-configure ldap \
    --with-libdir="lib/$(dpkg-architecture --query DEB_BUILD_MULTIARCH)" \
    ; \
    docker-php-ext-configure gd \
    --with-freetype \
    --with-jpeg=/usr \
    --with-webp \
    ; \
    \
    docker-php-ext-install -j$(nproc) \
    gd \
    pdo_pgsql \
    pdo_mysql \
    zip \
    ldap \
    exif \
    pcntl \
    posix \
    bcmath \
    ; \
    docker-php-ext-enable \
    zmq \
    ; \
    \
    # reset a list of apt-mark
    apt-mark auto '.*' > /dev/null; \
    apt-mark manual $aptMarkList; \
    ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
    | awk '/=>/ { print $3 }' \
    | sort -u \
    | xargs -r realpath | xargs -r dpkg-query --search \
    | cut -d: -f1 \
    | sort -u \
    | xargs -rt apt-mark manual; \
    \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
    # Install required libs
    apt-get install -y --no-install-recommends \
    unzip \
    libldap-common \
    ; \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# php.ini
RUN { \
    echo 'expose_php = Off'; \
    echo 'display_errors = On'; \
    echo 'display_startup_errors = On'; \
    echo 'log_errors = On'; \
    echo 'memory_limit=256M'; \
    echo 'max_execution_time=180'; \
    echo 'max_input_time=180'; \
    echo 'post_max_size=30M'; \
    echo 'upload_max_filesize=30M'; \
    echo 'date.timezone=UTC'; \
    } > ${PHP_INI_DIR}/conf.d/espocrm.ini

VOLUME /var/www/html

COPY docker-daemon.sh /usr/local/bin/docker-daemon.sh
COPY docker-websocket.sh /usr/local/bin/docker-websocket.sh

RUN chmod +x /usr/local/bin/docker-daemon.sh /usr/local/bin/docker-websocket.sh

CMD ["php-fpm"]
