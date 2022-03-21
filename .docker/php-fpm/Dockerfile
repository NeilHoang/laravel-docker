# Sử dụng image php:8.0-fpm-alpine
FROM php:8.0-fpm-alpine

LABEL maintainer="DucLS &lt;<a class="__cf_email__" href="/cdn-cgi/l/email-protection">[email protected]</a>&gt;"

ARG DEBIAN_FRONTEND=noninteractive

RUN docker-php-ext-install bcmath pdo_mysql

# Tạo thư mục /var/www/html
RUN mkdir -p /var/www/html

# Copy toàn bộ file trong thư mục ./src ở máy local vào trong thư mục /var/www/html ở trong container
COPY ./src /var/www/html
