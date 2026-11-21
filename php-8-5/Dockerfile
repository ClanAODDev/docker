FROM composer:2 AS composer
FROM debian:13-slim
LABEL maintainer="adam.schultz@live.com"

ENV DEBIAN_FRONTEND=noninteractive

ARG NGINX_UID=1001
ARG NGINX_GID=1001
ARG NGINX_DATA_UID=1002
ARG NGINX_DATA_GID=1002

# Base utilities and repository support
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        ca-certificates curl gnupg \
        cron git supervisor \
        zip unzip vim nano mariadb-client procps librsvg2-bin \
		less nano && \
    rm -rf /var/lib/apt/lists/*

# Add the third-party PHP APT repository
RUN curl -fsSL \
        https://packages.sury.org/debsuryorg-archive-keyring.deb \
        -o /tmp/debsuryorg-archive-keyring.deb && \
    apt-get install -y /tmp/debsuryorg-archive-keyring.deb && \
    rm -f /tmp/debsuryorg-archive-keyring.deb
RUN . /etc/os-release && \
    echo "deb [signed-by=/usr/share/keyrings/debsuryorg-archive-keyring.gpg] https://packages.sury.org/php/ ${VERSION_CODENAME} main" \
        > /etc/apt/sources.list.d/php.list

# PHP 8.5 and extensions
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        php8.5-fpm \
        php8.5-cli \
        php8.5-bcmath \
        php8.5-bz2 \
        php8.5-calendar \
        php8.5-curl \
        php8.5-exif \
        php8.5-gd \
        php8.5-gmp \
        php8.5-imagick \
        php8.5-intl \
        php8.5-mbstring \
        php8.5-mysql \
        php8.5-mysqlnd \
        php8.5-soap \
        php8.5-sockets \
        php8.5-sysvsem \
        php8.5-xml \
        php8.5-xsl \
        php8.5-zip \
        php8.5-apcu \
        && \
    rm -rf /var/lib/apt/lists/*

# Install Composer 2 from the official image
COPY --from=composer /usr/bin/composer /usr/local/bin/composer

# Match filesystem users
RUN groupadd -g ${NGINX_GID} nginx && \
    useradd -u ${NGINX_UID} -g nginx \
        -s /usr/sbin/nologin nginx && \
    groupadd -g ${NGINX_DATA_GID} nginx-data && \
    useradd -u ${NGINX_DATA_UID} -g nginx-data \
        -s /usr/sbin/nologin nginx-data

# Create nginx writable log dir
RUN install -d -m 0755 -o nginx -g nginx /var/log/php-fpm

# Create nginx working dir
RUN install -d -m 0755 -o nginx -g nginx /usr/share/nginx
WORKDIR /usr/share/nginx

#Copy configuration files
COPY --chown=root:root files/ /tmp/files
RUN set -eux; \
    find /tmp/files -type d -exec chmod 0755 {} +; \
    find /tmp/files -type f -exec chmod 0644 {} +; \
    cp -a /tmp/files/. /; \
    rm -rf /tmp/files;

#Create cache dir
RUN install -d -m 0755 -o nginx -g nginx /var/cache/php-fpm/

# Expose PHP 8.5 using unversioned executable names
RUN update-alternatives --set php /usr/bin/php8.5 && \
    ln -s /usr/sbin/php-fpm8.5 /usr/local/sbin/php-fpm && \
    php -v && \
    php-fpm -v && \
    composer --version && \
    php-fpm -t

WORKDIR /usr/share/nginx

ENTRYPOINT ["/usr/bin/supervisord", \
            "-c", "/etc/supervisor/supervisord.conf"]
