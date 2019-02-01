# Pull base image.
FROM php:7.3.1-apache

RUN apt-get clean && apt-get update && apt-get install --fix-missing wget apt-transport-https lsb-release ca-certificates gnupg -y
RUN echo "deb http://packages.dotdeb.org stretch all" >> /etc/apt/sources.list
RUN echo "deb-src http://packages.dotdeb.org stretch all" >> /etc/apt/sources.list
RUN cd /tmp && wget https://www.dotdeb.org/dotdeb.gpg && apt-key add dotdeb.gpg
RUN wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
RUN echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list

RUN apt-get clean && apt-get update && apt-get install --fix-missing -y \
  ruby-dev \
  rubygems \
  imagemagick \
  graphviz \
  sudo \
  git \
  vim \
  memcached \
  libmemcached-tools \
  libmemcached-dev \
  libpng-dev \
  libjpeg62-turbo-dev \
  libmcrypt-dev \
  libxml2-dev \
  libxslt1-dev \
  mysql-client \
  zip \
  libzip-dev \
  wget \
  linux-libc-dev \
  libyaml-dev \
  zlib1g-dev \
  libicu-dev \
  libpq-dev \
  bash-completion \
  htop \
  libwebp-dev \
  libxpm-dev \
  libfreetype6-dev \
  libssl-dev
  #postgresql-client-9.6

# Install memcached
RUN pecl install memcached-3.1.3
RUN docker-php-ext-enable memcached
COPY core/memcached.conf /etc/memcached.conf

# Install PHP ext lib
RUN docker-php-ext-configure gd --with-gd --with-webp-dir --with-jpeg-dir \
    --with-png-dir --with-zlib-dir --with-xpm-dir --with-freetype-dir \
    --enable-gd-native-ttf && docker-php-ext-install gd
RUN docker-php-ext-install mbstring zip soap pdo_mysql mysqli xsl opcache calendar intl exif pgsql pdo_pgsql ftp bcmath

# Opcache custom
RUN ( \
  echo "opcache.memory_consumption=128"; \
  echo "opcache.interned_strings_buffer=8"; \
  echo "opcache.max_accelerated_files=20000"; \
  echo "opcache.revalidate_freq=5"; \
  echo "opcache.fast_shutdown=1"; \
  echo "opcache.enable_cli=1"; \
  ) >> /usr/local/etc/php/conf.d/docker-php-ext-opcache.ini

# Create new web user for apache and grant sudo without password
RUN useradd web -d /var/www -g www-data -s /bin/bash
RUN usermod -aG sudo web
RUN echo 'web ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# Install YAML extension
RUN pecl install yaml-2.0.4
RUN docker-php-ext-enable yaml

# Install APCu extension
RUN pecl install apcu-5.1.16
RUN docker-php-ext-enable apcu
RUN echo "apc.enabled=1" >> /usr/local/etc/php/conf.d/docker-php-ext-apcu.ini

# SASS and Compass installation
RUN gem install compass

# Installation node.js
RUN curl -sL https://deb.nodesource.com/setup_10.x | bash - && \
	apt-get update && apt-get install -y nodejs && \
	npm install npm@latest -g

# Installation of LESS
RUN npm install -g less && npm install -g less-plugin-clean-css

# Installation of Grunt
RUN npm install -g grunt-cli

# Installation of Gulp
RUN npm install -g gulp

# Installation of Bower
RUN npm install -g bower

# Installation of Composer
RUN cd /usr/src && curl -sS http://getcomposer.org/installer | php
RUN cd /usr/src && mv composer.phar /usr/bin/composer

# Installation of drush
RUN git clone https://github.com/drush-ops/drush.git /usr/local/src/drush
RUN cd /usr/local/src/drush && git checkout 8.1.18
RUN ln -s /usr/local/src/drush/drush /usr/bin/drush
RUN cd /usr/local/src/drush && composer update && composer install

# Install xdebug extension (using beta is mandatory for php 7.3)
RUN pecl install xdebug-beta
RUN docker-php-ext-enable xdebug
RUN echo xdebug.remote_enable=1 >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini &&\
    echo xdebug.remote_autostart=0 >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini &&\
    echo xdebug.remote_connect_back=0 >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini &&\
    echo xdebug.remote_port=9000 >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini &&\
    echo xdebug.remote_host=docker_host >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini &&\
    echo xdebug.remote_log=/tmp/php7-xdebug.log >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# Installation of PHP_CodeSniffer with Drupal coding standards.
# See https://www.drupal.org/node/1419988#coder-composer
RUN composer global require drupal/coder
RUN ln -s ~/.composer/vendor/bin/phpcs /usr/local/bin
RUN ln -s ~/.composer/vendor/bin/phpcbf /usr/local/bin
RUN phpcs --config-set installed_paths ~/.composer/vendor/drupal/coder/coder_sniffer

# installation of ssmtp
RUN DEBIAN_FRONTEND=noninteractive apt-get install --fix-missing -y ssmtp && rm -r /var/lib/apt/lists/*
ADD core/ssmtp.conf /etc/ssmtp/ssmtp.conf
ADD core/php-smtp.ini /usr/local/etc/php/conf.d/php-smtp.ini

# Apache bullshit
RUN rm -rf /var/www/html && \
  mkdir -p /var/lock/apache2 /var/run/apache2 /var/log/apache2 /var/www/html && \
  chown -R web:www-data /var/lock/apache2 /var/run/apache2 /var/log/apache2 /var/www/html
COPY config/apache2.conf /etc/apache2
COPY core/envvars /etc/apache2
COPY core/other-vhosts-access-log.conf /etc/apache2/conf-enabled/
RUN rm /etc/apache2/sites-enabled/000-default.conf

# Enable rewrite and expires
RUN a2enmod rewrite expires && service apache2 restart

# Install Drupal Console for Drupal 8
RUN curl https://drupalconsole.com/installer -L -o drupal.phar && mv drupal.phar /usr/local/bin/drupal && chmod +x /usr/local/bin/drupal

# Install WKHTMLTOPDF 0.12.4
RUN apt-get update && apt-get remove -y libqt4-dev qt4-dev-tools wkhtmltopdf && apt-get autoremove -y
RUN apt-get install openssl build-essential libssl-dev libxrender-dev git-core libx11-dev libxext-dev libfontconfig1-dev libfreetype6-dev fontconfig -y
RUN mkdir /var/wkhtmltopdf
RUN cd /var/wkhtmltopdf && wget https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.4/wkhtmltox-0.12.4_linux-generic-amd64.tar.xz && tar xf wkhtmltox-0.12.4_linux-generic-amd64.tar.xz
RUN cp /var/wkhtmltopdf/wkhtmltox/bin/wkhtmltopdf /bin/wkhtmltopdf && cp /var/wkhtmltopdf/wkhtmltox/bin/wkhtmltoimage /bin/wkhtmltoimage
RUN chown -R web:www-data /var/wkhtmltopdf
RUN chmod +x /bin/wkhtmltopdf && chmod +x /bin/wkhtmltoimage

# Our apache volume
VOLUME /var/www/html

# create directory for ssh keys
RUN mkdir /var/www/.ssh/
RUN chown -R web:www-data /var/www/
RUN chmod -R 600 /var/www/.ssh/

# Set timezone to Europe/Paris
RUN echo "Europe/Paris" > /etc/timezone && dpkg-reconfigure -f noninteractive tzdata

# Expose 80 for apache, 9000 for xdebug
EXPOSE 80 9000

# Add web and root .bashrc config
# When you "docker exec -it" into the container, you will be switched as web user and placed in /var/www/html
RUN echo "exec su - web" > /root/.bashrc && \
    echo ". .profile" > /var/www/.bashrc && \
    echo "alias ll='ls -al'" > /var/www/.profile && \
    echo "cd /var/www/html" >> /var/www/.profile

# BROKEN
# # Install symfony autocomplete for web user
# RUN sudo -u web composer global require bamarni/symfony-console-autocomplete
# RUN echo "export PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/var/www/.composer/vendor/bamarni/symfony-console-autocomplete/bin" >> /var/www/.profile
# RUN echo 'eval "$(symfony-autocomplete)"' >> /var/www/.profile

# Custom php.ini
COPY config/php.ini /usr/local/etc/php/

# Set and run a custom entrypoint
COPY core/docker-entrypoint.sh /
RUN chmod 777 /docker-entrypoint.sh && chmod +x /docker-entrypoint.sh
ENTRYPOINT ["/docker-entrypoint.sh"]
