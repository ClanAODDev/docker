FROM mariadb:10.4
LABEL maintainer="adam.schultz@live.com"

#Add users/ids to match file system
RUN userdel mysql
RUN groupdel mysql || true
RUN groupdel sudo || true
RUN groupadd -g 27 mysql
RUN useradd -u 27 -g mysql -s /usr/sbin/nologin mysql

#Add logging directory for php-fpm (better if this is mounted to system
RUN install -d -m 0644 -o mysql -g mysql /var/log/mysql/
