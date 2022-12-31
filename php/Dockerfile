FROM alireaza/php-fpm:8.2.0

USER root

RUN apk --no-cache --virtual .pdo_pgsql add --update postgresql-dev \
&& apk --no-cache add postgresql-libs \
&& NPROC=$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) \
&& docker-php-ext-install -j${NPROC} pdo pdo_pgsql pgsql \
&& apk del .pdo_pgsql

RUN apk --no-cache --virtual .redis add --update autoconf build-base \
&& pecl install redis && docker-php-ext-enable redis \
&& apk del .redis

RUN apk --no-cache --virtual .kafka add --update autoconf build-base make git bash libpthread-stubs linux-headers python3 librdkafka-dev \
&& apk --no-cache add lz4-libs zstd-dev libsasl \
&& git clone --branch v1.9.2 --depth 1 https://github.com/edenhill/librdkafka.git /tmp/librdkafka \
&& cd /tmp/librdkafka \
&& STATIC_LIB_libzstd=/usr/lib/libzstd.a ./configure --enable-static \
&& NPROC=$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) \
&& make -j${NPROC} \
&& make -j${NPROC} install \
&& rm -rf /tmp/librdkafka \
&& pecl install rdkafka-6.0.3 && docker-php-ext-enable rdkafka \
&& apk del .kafka

RUN apk --no-cache --virtual .pcntl add --update \
&& docker-php-ext-configure pcntl --enable-pcntl \
&& NPROC=$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) \
&& docker-php-ext-install -j${NPROC} pcntl \
&& apk del .pcntl

RUN apk --no-cache --virtual .supervisord add --update \
&& apk --no-cache add supervisor \
&& touch /etc/supervisord.conf \
&& echo "[supervisord]" >> /etc/supervisord.conf \
&& echo "nodaemon=true" >> /etc/supervisord.conf \
&& apk del .supervisord

USER $UNAME

ENTRYPOINT ["/bin/sh", "-c", "php-fpm & supervisord --configuration /etc/supervisord.conf"]
