FROM debian:12-slim
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
        apt-transport-https \
        cron git supervisor \
        zip unzip vim nano && \
    rm -rf /var/lib/apt/lists/*

# Add the third-party PHP APT repository
RUN curl -fsSL \
        https://packages.sury.org/php/apt.gpg \
        -o /usr/share/keyrings/debsuryorg-archive-keyring.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/debsuryorg-archive-keyring.gpg] https://packages.sury.org/php/ bookworm main" \
        > /etc/apt/sources.list.d/php.list

# PHP 7.1 and extensions
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        php7.1-fpm \
        php7.1-cli \
        php7.1-curl \
        php7.1-bcmath \
        php7.1-bz2 \
        php7.1-gd \
        php7.1-intl \
        php7.1-mbstring \
        php7.1-mysql \
        php7.1-opcache \
        php7.1-soap \
        php7.1-xml \
        php7.1-zip \
        php7.1-apcu \
        php7.1-imagick && \
    rm -rf /var/lib/apt/lists/*

# Match filesystem users
RUN groupadd -g ${NGINX_GID} nginx && \
    useradd -u ${NGINX_UID} -g nginx \
        -s /usr/sbin/nologin nginx && \
    groupadd -g ${NGINX_DATA_GID} nginx-data && \
    useradd -u ${NGINX_DATA_UID} -g nginx-data \
        -s /usr/sbin/nologin nginx-data

RUN install -d -m 0755 -o nginx -g nginx \
        /var/log/php-fpm /usr/share/nginx

COPY ./supervisord.conf \
    /etc/supervisor/conf.d/supervisord.conf

# Debian PHP-FPM configuration paths differ
# from those in the official PHP image.
# Adapt your existing configuration accordingly.
COPY ./php-fpm.conf \
    /etc/php/7.1/fpm/php-fpm.conf

# Expose PHP 7.1 using unversioned executable names
RUN update-alternatives --set php /usr/bin/php7.1 && \
    ln -s /usr/sbin/php-fpm7.1 /usr/local/sbin/php-fpm && \
    php -v && \
    php-fpm -v

WORKDIR /usr/share/nginx

ENTRYPOINT ["/usr/bin/supervisord", \
            "-c", "/etc/supervisor/supervisord.conf"]
