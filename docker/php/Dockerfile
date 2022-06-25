FROM php:8.1-fpm AS production

# Update PHP and its dependencies
RUN apt-get update && apt-get install -y \
    openssl \
    git \
    unzip \
    libzip-dev \
    libmagickwand-dev \
    gnupg2 \
    rsync \
    vim

# Install PHP extensions
RUN docker-php-ext-install -j$(nproc) \
    pdo \
    pdo_mysql \
    opcache \
    zip

# install imagick
RUN pecl install imagick \
    && docker-php-ext-enable imagick

# Install intl extension for symfony requirements and gd
RUN apt-get install -y libicu-dev \
    && docker-php-ext-configure intl \
    && docker-php-ext-install intl \
    && docker-php-ext-install gd

# Install Redis
RUN pecl install -o -f redis \
    && rm -rf /tmp/pear \
    && docker-php-ext-enable redis

# Install Composer
COPY --from=composer /usr/bin/composer /usr/bin/composer

# Install NodeJS
RUN curl -sL https://deb.nodesource.com/setup_16.x | bash
RUN apt-get update && apt-get install -y nodejs && apt-get clean

# Install Yarn
RUN curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt-get update && apt-get install -y yarn

# Install Dependencies with Composer
# Prevent the reinstallation of vendors at every change in the source code
COPY composer.json composer.lock symfony.lock ./
RUN set -eux; \
	composer install --no-scripts --no-progress --no-interaction --prefer-dist --optimize-autoloader; \
	composer clear-cache

# Copy code files
COPY assets assets/
COPY bin bin/
COPY config config/
COPY public public/
COPY src src/
COPY templates templates/
COPY translations translations/
COPY .env package.json postcss.config.js webpack.config.js yarn.lock ./

# Install assets
RUN yarn install --pure-lockfile && yarn build

# Copy PHP config
COPY docker/php/conf.d/php.ini "$PHP_INI_DIR/php.ini"


FROM production AS development

# Install XDebug
RUN pecl install xdebug \
    && docker-php-ext-enable xdebug

COPY docker/php/conf.d/xdebug.ini /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini