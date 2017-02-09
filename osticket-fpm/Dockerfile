FROM ubuntu:16.04
MAINTAINER Lee Keitel <lfkeitel@usi.edu>

# setup workdir
RUN mkdir /data
WORKDIR /data

# environment for osticket
ENV OSTICKET_VERSION 1.10
ENV HOME /data

# requirements
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get -y install \
    nano \
    wget \
    unzip \
    msmtp \
    ca-certificates \
    supervisor \
    cron \
    php7.0-cli \
    php7.0-fpm \
    php7.0-imap \
    php7.0-gd \
    php7.0-curl \
    php7.0-ldap \
    php7.0-mbstring \
    php7.0-mysql && \
    rm -rf /var/lib/apt/lists/*

# Download & install OSTicket
RUN wget -nv -O osTicket.zip http://osticket.com/sites/default/files/download/osTicket-v${OSTICKET_VERSION}.zip && \
    unzip osTicket.zip && \
    rm osTicket.zip && \
    chown -R www-data:www-data /data/upload/ && \
    chmod -R a+rX /data/upload/ /data/scripts/ && \
    chmod -R u+rw /data/upload/ /data/scripts/ && \
    mv /data/upload/setup /data/upload/setup_hidden && \
    chown -R root:root /data/upload/setup_hidden && \
    chmod 700 /data/upload/setup_hidden

# Download LDAP plugin
RUN wget -nv -O upload/include/plugins/auth-ldap.phar http://osticket.com/sites/default/files/download/plugin/auth-ldap.phar

# Configure php-fpm & PHP7.0
RUN sed -i -e "s/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/g" /etc/php/7.0/fpm/php.ini && \
    sed -i -e "s/upload_max_filesize\s*=\s*2M/upload_max_filesize = 100M/g" /etc/php/7.0/fpm/php.ini && \
    sed -i -e "s/post_max_size\s*=\s*8M/post_max_size = 100M/g" /etc/php/7.0/fpm/php.ini && \
    sed -i -e 's#;sendmail_path\s*=\s*#sendmail_path = "/usr/bin/msmtp -C /etc/msmtp -t "#g' /etc/php/7.0/fpm/php.ini && \
    sed -i -e "s/;daemonize\s*=\s*yes/daemonize = no/g" /etc/php/7.0/fpm/php-fpm.conf && \
    sed -i -e "s/;catch_workers_output\s*=\s*yes/catch_workers_output = yes/g" /etc/php/7.0/fpm/pool.d/www.conf && \
    sed -i -e "s/listen = \/run\/php\/php7.0-fpm.sock/listen = 9000/g" /etc/php/7.0/fpm/pool.d/www.conf && \
    phpenmod imap && \
    mkdir /run/php

# Configure msmtp log file permissions
RUN touch /var/log/msmtp.log && \
    chown www-data:www-data /var/log/msmtp.log

ADD supervisord-fpm.conf /data/supervisord.conf
ADD msmtp.conf /data/msmtp.conf
ADD bin/ /data/bin

VOLUME "/data/upload"
EXPOSE 9000
CMD ["/data/bin/start.sh"]
