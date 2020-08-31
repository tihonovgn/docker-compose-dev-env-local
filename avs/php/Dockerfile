FROM php:7.4-fpm

RUN apt-get update && apt-get install -y \
        curl \
        wget \
        git \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libzip-dev \
        libicu-dev \
        librabbitmq-dev \
        g++ libcurl4-gnutls-dev libpq-dev libxml2-dev unzip zlib1g-dev \
    && docker-php-ext-install -j$(nproc) iconv mysqli pdo_mysql zip intl bcmath \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd \
    && printf "\n" | pecl install amqp \
    && printf "\n" | pecl install xdebug-2.9.0 \
    && printf "\n" |docker-php-ext-enable xdebug

#consul
ENV CONSUL_TEMPLATE_VERSION 0.16.0
ENV CONSUL_TEMPLATE_CHECKSUM 064b0b492bb7ca3663811d297436a4bbf3226de706d2b76adade7021cd22e156

RUN curl --retry 7 --fail -Lks "https://releases.hashicorp.com/consul-template/${CONSUL_TEMPLATE_VERSION}/consul-template_${CONSUL_TEMPLATE_VERSION}_linux_amd64.zip" -o /tmp/consul-template.zip \
    && echo "${CONSUL_TEMPLATE_CHECKSUM}  /tmp/consul-template.zip" | sha256sum -c \
    && unzip /tmp/consul-template.zip -d /usr/local/bin \
    && rm /tmp/consul-template.zip

RUN echo 'date.timezone="Europe/Astrakhan"' >>  /usr/local/etc/php/conf.d/date.ini

ENV CONTAINERPILOT_VERSION 2.4.3
ENV CONTAINERPILOT_CHECKSUM 2c469a0e79a7ac801f1c032c2515dd0278134790
ENV CONTAINERPILOT file:///etc/containerpilot.json

RUN curl --retry 7 --fail -Lks "https://github.com/joyent/containerpilot/releases/download/${CONTAINERPILOT_VERSION}/containerpilot-${CONTAINERPILOT_VERSION}.tar.gz" -o /tmp/containerpilot.tar.gz \
    && echo "${CONTAINERPILOT_CHECKSUM}  /tmp/containerpilot.tar.gz" | sha1sum -c \
    && tar zxf /tmp/containerpilot.tar.gz -C /usr/local/bin \
    && rm /tmp/containerpilot.tar.gz

COPY config/ /etc
COPY scripts/ /usr/local/bin

RUN chmod +x /usr/local/bin/reload.sh
#end consul

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

ADD php.ini /usr/local/etc/php/conf.d/40-custom.ini

WORKDIR /var/www/

# Set the locale
RUN apt-get clean && apt-get update && apt-get install -y locales
RUN locale-gen ru_RU.UTF-8
ENV LANG ru_RU.UTF-8
ENV LANGUAGE ru_RU:en
ENV LC_ALL ru_RU.UTF-8

# Запускаем контейнер
# Из документации: The main purpose of a CMD is to provide defaults for an executing container. These defaults can include an executable,
# or they can omit the executable, in which case you must specify an ENTRYPOINT instruction as well.
CMD [ "/usr/local/bin/containerpilot", \
    "php-fpm", \
        "--nodaemonize"]
