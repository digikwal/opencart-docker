# OpenCart runtime image with nginx + php-fpm
ARG PHP_VERSION=8.4

FROM php:${PHP_VERSION}-fpm

ENV TZ=UTC

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
      ca-certificates \
      cron \
      nginx \
      supervisor \
      tzdata \
    && rm -rf /var/lib/apt/lists/*

COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/
RUN install-php-extensions \
    @composer \
    apcu \
    gd \
    intl \
    memcached \
    mysqli \
    opcache \
    pdo_mysql \
    pdo_pgsql \
    redis \
    zip

RUN usermod -d /home/www-data -m www-data || true \
    && mkdir -p /home/www-data \
    && chown www-data:www-data /home/www-data

WORKDIR /var/www
COPY upload/ /var/www/upload/
COPY docker/ /var/www/docker/

COPY docker/runtime/nginx/nginx.conf /etc/nginx/nginx.conf
COPY docker/runtime/nginx/default.conf /etc/nginx/conf.d/default.conf
COPY docker/runtime/supervisor/supervisord.conf /etc/supervisor/supervisord.conf
COPY docker/runtime/supervisor/services/ /etc/supervisor/conf.d/services/
COPY docker/php/docker-opencart-entrypoint /usr/local/bin/docker-opencart-entrypoint

RUN chmod +x /usr/local/bin/docker-opencart-entrypoint \
    && mkdir -p /var/log/supervisor /var/run/supervisor \
    && mkdir -p /var/log/nginx /var/cache/nginx \
    && mkdir -p /var/www/storage \
    && chown -R www-data:www-data /var/www/storage

EXPOSE 80

ENTRYPOINT ["docker-opencart-entrypoint"]
CMD ["supervisord", "-c", "/etc/supervisor/supervisord.conf"]
