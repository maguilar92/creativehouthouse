FROM ubuntu:16.04

RUN apt-get update \
    && apt-get install --no-install-recommends -y software-properties-common locales supervisor \
    && apt-get update \
    && locale-gen en_US.UTF-8

ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US:en
ENV LC_ALL=en_US.UTF-8

RUN add-apt-repository ppa:nginx/stable
RUN add-apt-repository ppa:ondrej/php
RUN apt-get update
RUN apt-get install -y sudo
RUN apt-get install --yes curl
RUN apt-get install --no-install-recommends -y \
        wget git zip \
        automake \
        make \ 
        graphviz \
        python-sphinx \
        python-sphinxcontrib.phpdomain \
        nginx \
        libpng-dev \ 
        git \
        php7.1-fpm \
        php7.1-cli \
        php7.1-xdebug \
        php7.1-pdo \
        php7.1-pdo-mysql \
        php7.1-sqlite3 \
        php7.1-xml \
        php7.1-mbstring \
        php7.1-tokenizer \
        php7.1-zip \
        php7.1-mcrypt \
        php7.1-gd \
        php7.1-curl
RUN git config --global user.name "CreativeHotHouse"
RUN mkdir /run/php
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
RUN version=$(php -r "echo PHP_MAJOR_VERSION.PHP_MINOR_VERSION;") \
    && curl -A "Docker" -o /tmp/blackfire-probe.tar.gz -D - -L -s https://blackfire.io/api/v1/releases/probe/php/linux/amd64/$version \
    && tar zxpf /tmp/blackfire-probe.tar.gz -C /tmp \
    && mv /tmp/blackfire-*.so $(php -r "echo ini_get('extension_dir');")/blackfire.so \
    && printf "extension=blackfire.so\nblackfire.agent_socket=tcp://blackfire:8707\n" > /etc/php/7.1/mods-available/blackfire.ini \
    && phpenmod blackfire
RUN curl --silent --location https://deb.nodesource.com/setup_8.x | sudo bash -
RUN apt-get install --yes nodejs
RUN apt-get install --yes build-essential
RUN apt-get clean
RUN apt-get autoremove -y
RUN rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY default /etc/nginx/sites-enabled/default
COPY php.ini /etc/php/7.1/fpm/php.ini
COPY php-fpm.conf /etc/php/7.1/fpm/php-fpm.conf
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

COPY laravel.ini /etc/php/7.1/fpm/conf.d/laravel.ini
#COPY disabled-xdebug.ini /etc/php/7.1/mods-available/xdebug.ini
COPY enabled-xdebug.ini /etc/php/7.1/mods-available/xdebug.ini

RUN /etc/init.d/php7.1-fpm restart

RUN mkdir /tmp/certgen
WORKDIR /tmp/certgen
RUN openssl genrsa -des3 -passout pass:x -out server.pass.key 2048 \
    && openssl rsa -passin pass:x -in server.pass.key -out server.key \
    && rm server.pass.key \
    && openssl genrsa -out server.pem 2048 \
    && openssl req -new -key server.key -out server.csr -subj "/CN=creativehothouse.com" \
    && openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt \
    && cp server.pem /etc/ssl/certs/ \
    && cp server.crt /etc/ssl/certs/ \
    && cp server.key /etc/ssl/private/ \
    && rm -rf /tmp/certgen

EXPOSE 80
EXPOSE 443

WORKDIR /var/www/html

CMD ["supervisord"]
