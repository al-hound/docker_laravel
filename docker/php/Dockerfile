FROM php:8.3-fpm-alpine

ENV COMPOSER_HOME=/tmp/composer
ENV PATH="${COMPOSER_HOME}/vendor/bin:${PATH}"

RUN apk add --no-cache \
    bash \
    curl \
    git \
    nodejs \
    npm \
    icu-dev \
    libzip-dev \
    oniguruma-dev \
    unzip \
    zip \
    $PHPIZE_DEPS \
    && docker-php-ext-install \
        bcmath \
        intl \
        mbstring \
        opcache \
        pdo_mysql \
        zip \
    && pecl install redis \
    && docker-php-ext-enable redis \
    && apk del $PHPIZE_DEPS

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

RUN composer global require laravel/installer

WORKDIR /var/www/html