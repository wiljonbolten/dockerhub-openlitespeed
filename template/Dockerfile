FROM ubuntu:20.04 AS BUILD
LABEL mantainer="Wiljon Bolten <wiljonbolten@gmail.com>" vendor="überhub"

ARG OLS_VERSION
ARG PHP_VERSION

RUN apt-get update && \
    apt-get install \
    wget \ 
    curl \ 
    cron \
    tzdata \ 
    -y

RUN wget https://openlitespeed.org/preuse/openlitespeed-$OLS_VERSION.tgz && \
    tar xzf openlitespeed-$OLS_VERSION.tgz && \
    cd openlitespeed && \
    ./install.sh && \
    echo 'cloud-docker' > /usr/local/lsws/PLAT && \
    rm -rf /openlitespeed && \
    rm /openlitespeed-$OLS_VERSION.tgz

RUN apt-get install \
    mysql-client \
    ls$PHP_VERSION \
    ls$PHP_VERSION-common \
    ls$PHP_VERSION-curl \
    ls$PHP_VERSION-imagick \
    ls$PHP_VERSION-intl \
    ls$PHP_VERSION-memcached \
    ls$PHP_VERSION-mysql \
    ls$PHP_VERSION-opcache \
    ls$PHP_VERSION-redis \
    -y

RUN ["/bin/bash", "-c", "if [[ $PHP_VERSION == php7* ]]; then apt-get install ls$PHP_VERSION-json -y; fi"]

COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

RUN rm -f /etc/openlitespeed/admin/htpasswd

RUN wget -O /usr/local/lsws/admin/misc/lsup.sh \
    https://raw.githubusercontent.com/litespeedtech/openlitespeed/master/dist/admin/misc/lsup.sh && \
    chmod +x /usr/local/lsws/admin/misc/lsup.sh

RUN wget -O -  https://get.acme.sh | sh

EXPOSE 7080

ADD docker.conf /usr/local/lsws/conf/templates/docker.conf
ADD setup_docker.sh /usr/local/lsws/bin/setup_docker.sh
ADD httpd_config.xml /usr/local/lsws/conf/httpd_config.xml

RUN /usr/local/lsws/bin/setup_docker.sh && rm /usr/local/lsws/bin/setup_docker.sh
RUN chown 999:999 /usr/local/lsws/conf -R
RUN cp -RP /usr/local/lsws/conf/ /usr/local/lsws/.conf/
RUN cp -RP /usr/local/lsws/admin/conf /usr/local/lsws/admin/.conf/

#RUN sed -i "s|fcgi-bin/lsphp|/usr/local/lsws/ls$PHP_VERSION/bin/lsphp|g" /usr/local/lsws/conf/httpd_config.conf

RUN ["/bin/bash", "-c", "if [[ $PHP_VERSION == php8* ]]; then ln -sf /usr/local/lsws/ls$PHP_VERSION/bin/lsphp /usr/local/lsws/fcgi-bin/lsphp8; fi"]
RUN ["/bin/bash", "-c", "if [[ $PHP_VERSION == php8* ]]; then ln -sf /usr/local/lsws/fcgi-bin/lsphp8 /usr/local/lsws/fcgi-bin/lsphp; fi"]
RUN ["/bin/bash", "-c", "if [[ $PHP_VERSION == php7* ]]; then ln -sf /usr/local/lsws/ls$PHP_VERSION/bin/lsphp /usr/local/lsws/fcgi-bin/lsphp7; fi"]
RUN ["/bin/bash", "-c", "if [[ $PHP_VERSION == php7* ]]; then ln -sf /usr/local/lsws/fcgi-bin/lsphp7 /usr/local/lsws/fcgi-bin/lsphp; fi"]

COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
WORKDIR /var/www/vhosts/
