FROM php:7.4.13-fpm-alpine3.12

# 修改系统源为阿里云
RUN echo 'http://mirrors.aliyun.com/alpine/v3.11/main' > /etc/apk/repositories \
    && echo 'http://mirrors.aliyun.com/alpine/v3.11/community' >> /etc/apk/repositories \
    && echo 'http://mirrors.aliyun.com/alpine/edge/testing' >> /etc/apk/repositories \
    && apk update

# 修改系统时区为CST
RUN ln -snf /usr/share/zoneinfo/"Asia/Shanghai" /etc/localtime \
    && echo "Asia/Shanghai" > /etc/timezone

# 安装系统依赖包
RUN apk add --no-cache \
        ca-certificates \
        tzdata \
        libpng-dev \
        freetype-dev \
        jpeg-dev \
        zlib-dev \
        nginx \
        supervisor \
        grpc

# 安装编译安装的依赖包（virtual 临时）
RUN set -eux; \
	apk add --no-cache --virtual .build-deps \
        autoconf \
        dpkg-dev dpkg \
        file \
        g++ \
        gcc \
        libc-dev \
        make \
        pkgconf \
        re2c

# 安装扩展（pecl扩展）
RUN set -eux; \
    pecl update-channels; \
    pecl install apcu redis mongodb skywalking;

# 安装扩展（系统扩展）
RUN set -eux; \
    docker-php-ext-configure gd --with-freetype --with-jpeg; \
    docker-php-ext-install gd bcmath pdo_mysql opcache;

# 启用扩展
RUN set -eux; \
    docker-php-ext-enable apcu redis mongodb opcache;

# 清理依赖包
RUN set -eux; \
    apk del --no-network .build-deps; \
    docker-php-source delete;

# nginx初始化
RUN set -eux; \
    deluser --remove-home www-data; \
    addgroup -g 1000 -S www; \
    adduser -u 1000 -D -S -G www www;

RUN set -eux; \
    mkdir -p /var/www; \
    mkdir -p /var/log/php; \
    chown -R www:www /var/www; \
    chown -R www:www /var/log/php; \
    chown -R www:www /var/log/nginx; \
    chown -R www:www /var/lib/nginx; \
    rm -rf /usr/local/etc/php-fpm.d/*; 

# 拷贝配置文件
COPY config/nginx/nginx.conf /etc/nginx/nginx.conf
COPY config/nginx/conf.d   /etc/nginx/conf.d
COPY config/fpm/php-fpm.conf /usr/local/etc/php-fpm.conf
COPY config/fpm/php-fpm.d /usr/local/etc/php-fpm.d
COPY config/php/php.ini /usr/local/etc/php/php.ini
COPY config/supervisor/supervisord.conf /etc/supervisord.conf
COPY config/supervisor/supervisor.d /etc/supervisor.d
COPY html /var/www/html
COPY startup.sh /opt/

# 暴露端口
EXPOSE 80 443

# 守护进程
CMD ["/opt/startup.sh"]
