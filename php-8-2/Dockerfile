FROM php:8.2-fpm
LABEL maintainer="adam.schultz@live.com"

ARG NGINX_UID="1001"
ARG NGINX_GID="1001"
ARG NGINX_DATA_UID="1002"
ARG NGINX_DATA_GID="1002"

#Upgrade container and add needed tools from debian
RUN DEBIAN_FRONTEND=noninteractive apt-get update -q \
	&& DEBIAN_FRONTEND=noninteractive apt-get upgrade -qq -y \
	&& DEBIAN_FRONTEND=noninteractive apt-get install -qq -y \
		cron curl git supervisor zip unzip vim nano mariadb-client \
	&& DEBIAN_FRONTEND=noninteractive apt-get autoremove -y \
	&& DEBIAN_FRONTEND=noninteractive apt-get clean

#Install php extension installer from https://github.com/mlocati/docker-php-extension-installer
ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/
RUN chmod 755 /usr/local/bin/install-php-extensions

#Install needed php extensions
RUN install-php-extensions \
	bcmath bz2 calendar exif gd iconv intl mbstring \
	mysqli opcache pdo_mysql soap xsl zip sockets apcu \
	pcntl imagick gmp mysqlnd sysvsem
#removed memcached, we're not running the daemon in supervisor anyway
#removed redis

RUN touch /usr/local/etc/php/php.ini
RUN echo "extension=apcu.so" >> /usr/local/etc/php/php.ini
RUN echo "apc.enable_cli=1" >> /usr/local/etc/php/php.ini
RUN echo "apc.enable=1" >> /usr/local/etc/php/php.ini

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

#Add users/ids to match file system
RUN groupadd -g ${NGINX_GID} nginx
RUN useradd -u ${NGINX_UID} -g nginx -s /usr/sbin/nologin nginx
RUN groupadd -g ${NGINX_DATA_GID} nginx-data
RUN useradd -u ${NGINX_DATA_UID} -g nginx-data -s /usr/sbin/nologin nginx-data

#Add logging directory for php-fpm (better if this is mounted to system
RUN install -d -m 0644 -o nginx -g nginx /var/log/php-fpm/

#Copy in supervisor config
COPY ./supervisord.conf /etc/supervisor/supervisord.conf
COPY ./php.supervisord.conf /etc/supervisor/default/php.supervisord.conf
COPY ./cron.supervisord.conf /etc/supervisor/default/cron.supervisord.conf

#Copy php-fpm config
COPY ./php-fpm.conf /usr/local/etc/php-fpm.conf
#Note: Pool configurations need to be mounted to /usr/local/etc/php-fpm-d and will define listen port

#Replace upstream default/working director
RUN install -d -m 0644 -o nginx -g nginx /usr/share/nginx/
WORKDIR /usr/share/nginx

#Replace upstream entry point
ENTRYPOINT ["/usr/bin/supervisord","-c","/etc/supervisor/supervisord.conf"]
