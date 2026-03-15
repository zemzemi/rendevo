FROM php:8.4-cli

# Install system dependencies
RUN apt-get update && apt-get install -y \
    git curl zip unzip \
    libpng-dev libonig-dev libxml2-dev libzip-dev \
    && rm -rf /var/lib/apt/lists/*

# Install PHP extensions
RUN docker-php-ext-install pdo pdo_mysql mbstring exif pcntl bcmath gd zip

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/html

# Install dependencies first (layer cache)
COPY composer.json composer.lock ./
RUN composer install --no-scripts --no-autoloader --prefer-dist

# Copy application files
COPY . .

# Create required Laravel directories and set permissions
RUN mkdir -p bootstrap/cache storage/framework/{sessions,views,cache/data} storage/logs \
    && chmod -R 775 bootstrap/cache storage

# Generate optimized autoload (without post-autoload scripts — run at startup)
RUN composer dump-autoload --optimize --no-scripts

EXPOSE 8000
