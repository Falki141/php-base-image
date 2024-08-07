# Use the official php image
FROM php:8.2-fpm AS build

ENV TZ=Europe/Berlin \
    NGINX_PORT=80

# Add dependencies
RUN apt update \
    && apt dist-upgrade -y \
    && apt install -y --no-install-recommends \
        ca-certificates \
        nginx \
        unzip \
        zip \
        cron \
        supervisor \
        libfreetype6-dev \
        libfontconfig1-dev \
        libxml++2.6-dev \
        libjpeg-dev \
        libssl-dev \
        libzip-dev \
        libmagickwand-dev \
        libwebp-dev \
    && apt-get clean \
    && apt-get autoremove -y \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Install php extensions
RUN docker-php-ext-configure gd \
        --with-jpeg=/usr/include/ \
        --with-freetype=/usr/include/ \
        --with-webp=/usr/include/ \
    && docker-php-ext-install -j$(nproc) \
        dom \
        ctype \
        exif \
        gd \
        intl \
        opcache \
        pdo \
        pdo_mysql \
        session \
        phar \
        xml \
        fileinfo \
        simplexml \
        sockets \
        zip \
    && pecl install imagick \
    && pecl install redis \
    && docker-php-ext-enable \
        dom \
        ctype \
        exif \
        gd \
        imagick \
        intl \
        opcache \
        pdo \
        pdo_mysql \
        session \
        phar \
        xml \
        fileinfo \
        simplexml \
        sockets \
        zip \
        redis

RUN apt-get purge -y --auto-remove gcc g++ make \
    && apt-get clean \
    && apt-get autoremove -y \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Set timezone
RUN cp /usr/share/zoneinfo/${TZ} /etc/localtime \
    && echo ${TZ} > /etc/timezone \
    && useradd -s /bin/false nginx

# PHP configuration 
RUN ln -s /usr/bin/php8 /usr/bin/php \ 
    && mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"
COPY ./config/fpm/pool.d/app.conf  /usr/local/etc/php-fpm.d/www.conf
COPY ./config/fpm/conf.d/custom.ini "$PHP_INI_DIR/conf.d/custom.ini"

# Nginx configuration
COPY ./config/nginx/default /etc/nginx/conf.d/default.conf
COPY ./config/nginx/nginx.conf /etc/nginx/

# www-data crontab for laravel artisan schedule
COPY --chown=root:crontab ./config/cron/www-data /var/spool/cron/crontabs/root
RUN chmod 0600 /var/spool/cron/crontabs/root \
    && rm /usr/local/etc/php-fpm.d/docker.conf \
    && rm /usr/local/etc/php-fpm.d/zz-docker.conf \
    && rm /etc/nginx/sites-enabled/default

# Add Supervisord config
COPY ./config/supervisord/ /etc/

# Nginx port
EXPOSE $NGINX_PORT

WORKDIR /app

# Entrypoint and command
ENTRYPOINT ["/usr/bin/supervisord", "-c"] 
CMD ["/etc/supervisor/supervisord.conf"]
