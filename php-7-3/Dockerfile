FROM php:7.3-fpm
LABEL maintainer="adam.schultz@live.com"

#Upgrade container and add needed tools from debian
RUN DEBIAN_FRONTEND=noninteractive apt-get update -q \
    && DEBIAN_FRONTEND=noninteractive apt-get upgrade -qq -y \
    && DEBIAN_FRONTEND=noninteractive apt-get install -qq -y \
      cron curl git supervisor zip unzip \
    && DEBIAN_FRONTEND=noninteractive apt-get autoremove -y \
    && DEBIAN_FRONTEND=noninteractive apt-get clean

#Install php extension installer from https://github.com/mlocati/docker-php-extension-installer
ADD https://raw.githubusercontent.com/mlocati/docker-php-extension-installer/master/install-php-extensions /usr/local/bin/
RUN chmod 755 /usr/local/bin/install-php-extensions

#Install needed php extensions
RUN install-php-extensions \
      bcmath bz2 calendar exif gd iconv intl mbstring memcached \
      mysqli opcache pdo_mysql redis soap xsl zip sockets apcu \
      pcntl imagick

#Add users/ids to match file system
RUN groupadd -g 996 nginx
RUN useradd -u 997 -g nginx -s /usr/sbin/nologin nginx
RUN groupadd -g 1004 nginx-data
RUN useradd -u 1004 -g nginx-data -s /usr/sbin/nologin nginx-data

#Add logging directory for php-fpm (better if this is mounted to system
RUN install -d -m 0644 -o nginx -g nginx /var/log/php-fpm/

#Copy in supervisor config
COPY ./supervisord.conf /etc/supervisor/conf.d/supervisord.conf

#Copy php-fpm config
COPY ./php-fpm.conf /usr/local/etc/php-fpm.conf
#Note: Pool configurations need to be mounted to /usr/local/etc/php-fpm-d and will define listen port

#Replace upstream default/working director
RUN install -d -m 0644 -o nginx -g nginx /usr/share/nginx/
WORKDIR /usr/share/nginx

#Replace upstream entry point
ENTRYPOINT ["/usr/bin/supervisord","-c","/etc/supervisor/supervisord.conf"]
