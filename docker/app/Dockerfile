FROM ubuntu:22.04

LABEL maintainer="<group>"

ENV DEBIAN_FRONTEND=noninteractive

# Setting timezone
RUN apt-get update \
    && apt-get install -y gnupg tzdata \
    && echo "UTC" | tee /etc/timezone \
    && unlink /etc/localtime \
    && ln -s /usr/share/zoneinfo/UTC /etc/localtime \
    && dpkg-reconfigure -f noninteractive tzdata

# Making php to install properly
RUN apt-get install -y software-properties-common \
    && add-apt-repository ppa:ondrej/php \
    && add-apt-repository ppa:ondrej/nginx

RUN apt-get update \
    && apt-get install -y curl zip unzip git supervisor sqlite3 nginx \
    && which nginx

# Installing PHP plugins and composer, removing cache
RUN apt-get update \
    && add-apt-repository universe \
    && apt-get install -y \
       php-fpm php-mysql \
       php-mbstring php-xml php-bcmath \
       php-msgpack php-igbinary \
       php-curl \
    && php -r "readfile('http://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer \
    && mkdir /run/php \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
    && echo "daemon off;" >> /etc/nginx/nginx.conf

# Making sure php uses php 8.1
RUN  update-alternatives --set php /usr/bin/php8.1 \
    && update-alternatives --set phar /usr/bin/phar8.1 \
    && update-alternatives --set phar.phar /usr/bin/phar.phar8.1 

# Output logs from Nginx correctly
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log

# Setup virtual host for Nginx 
ADD default /etc/nginx/sites-available/default

# Making sure nginx process runs consistently, monitored by supervisord
ADD supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Some custom configuration for making php-fpm (Nginx) work correctly.
ADD php-fpm.conf /etc/php/8.0/fpm/php-fpm.conf

# Custom bash script to run supervisord
ADD start-container /usr/bin/start-container
RUN chmod +x /usr/bin/start-container

ENTRYPOINT ["start-container"]
