# Use the official Ubuntu image as the base image
FROM ubuntu:24.04

RUN echo "Acquire::http::Pipeline-Depth 0;" > /etc/apt/apt.conf.d/99custom && \
    echo "Acquire::http::No-Cache true;" >> /etc/apt/apt.conf.d/99custom && \
    echo "Acquire::BrokenProxy    true;" >> /etc/apt/apt.conf.d/99custom

ENV DEBIAN_FRONTEND noninteractive
ENV TZ=UTC
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
# Install necessary packages
RUN apt-get update && apt-get install -y \
    software-properties-common curl ca-certificates zip unzip git supervisor sqlite3 libcap2-bin libpng-dev dnsutils librsvg2-bin fswatch ffmpeg nano nginx\
    && add-apt-repository ppa:ondrej/php \
    && apt-get update && apt-get install -y \
    php8.2-fpm \
    php8.2-cli \
    php8.2-mysql \
    php8.2-xml \
    php8.2-mbstring \
    php8.2-curl \
    php8.2-zip \
    php8.2-gd \
    php8.2-redis \
    && apt-get clean

# Get latest Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

COPY nginx/nginx.conf /etc/nginx/sites-available/default

COPY php/fpm-pool.conf /etc/php/8.2/fpm/pool.d/www.conf

RUN service php8.2-fpm restart
RUN service nginx restart

# Create a supervisord configuration file to run both PHP-FPM and Nginx
RUN echo '[supervisord]\nnodaemon=true\n' > /etc/supervisor/supervisord.conf \
    && echo '[program:php-fpm]\ncommand=/usr/sbin/php-fpm8.2\n' >> /etc/supervisor/supervisord.conf \
    && echo '[program:nginx]\ncommand=/usr/sbin/nginx -g "daemon off;"\n' >> /etc/supervisor/supervisord.conf

EXPOSE 80/tcp
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]