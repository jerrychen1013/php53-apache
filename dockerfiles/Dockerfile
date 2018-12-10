#
# NOTE: THIS DOCKERFILE IS GENERATED VIA "update.sh"
#
# PLEASE DO NOT EDIT IT DIRECTLY.
#

FROM debian:jessie

# dependencies required for running "phpize"
# (see persistent deps below)
RUN set -xe \
	&& DEBIAN_FRONTEND=noninteractive apt-get update -qq \
	&& DEBIAN_FRONTEND=noninteractive apt-get install -qq -y --no-install-recommends --no-install-suggests \
	ca-certificates \
	curl \
	libpcre3 \
	librecode0 \
	libmysqlclient-dev \
	libsqlite3-0 \
	libxml2 \
	# phpize deps
	autoconf \
	file \
	g++ \
	gcc \
	libc-dev \
	make \
	pkg-config \
	re2c \
	xz-utils \
	# other tools
	sudo \
	cron \
	git \
	wget \
	python \
	vim \
	unzip \
	mysql-client \
	zip \
	apt-utils \
	supervisor \
	#====安裝mcrypt====
	libmcrypt-dev \
	#==================
	libpng-dev \
	libmagickwand-dev \
	libldb-dev \
	libldap2-dev \
	libsasl2-dev \
	libgeoip-dev \
	libpq-dev \
	libzip-dev \
	libbz2-dev \
	libgd2-dev \
	libjpeg-dev \
	libgif-dev \
	&& ln -s /usr/lib/x86_64-linux-gnu/libssl.so  /usr/lib \
	&& export PHP_OPENSSL_DIR=yes \
	&& DEBIAN_FRONTEND=noninteractive apt-get purge -qq -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
	&& rm -rf /var/lib/apt/lists/*

# install JP locale
RUN echo "ja_JP.UTF-8 UTF-8" >> /etc/locale.gen \
	locale-gen

ENV PHP_INI_DIR /usr/local/etc/php
RUN mkdir -p $PHP_INI_DIR/conf.d

##<apache2>##
RUN apt-get update && apt-get install -y \
	apache2-bin apache2-dev apache2.2-common \
	--no-install-recommends && rm -rf /var/lib/apt/lists/*

RUN rm -rf /var/www/html && mkdir -p /var/lock/apache2 /var/run/apache2 /var/log/apache2 /var/www/html \ 
	&& chown -R www-data:www-data /var/lock/apache2 /var/run/apache2 /var/log/apache2 /var/www/html

ENV APACHE_CONFDIR /etc/apache2

# Apache + PHP requires preforking Apache for best results
RUN a2dismod mpm_event && a2enmod mpm_prefork

# PHP files should be handled by PHP, and should be preferred over any other file type
RUN { \
	echo '<FilesMatch \.php$>'; \
	echo '\tSetHandler application/x-httpd-php'; \
	echo '</FilesMatch>'; \
	echo; \
	echo 'DirectoryIndex disabled'; \
	echo 'DirectoryIndex index.php index.html'; \
	echo; \
	echo '<Directory /var/www/>'; \
	echo '\tOptions -Indexes'; \
	echo '\tAllowOverride All'; \
	echo '</Directory>'; \
	} | tee -a "/etc/apache2/apache2.conf"


# compile openssl, otherwise --with-openssl won't work
RUN CFLAGS="-fPIC" && OPENSSL_VERSION="1.0.2g" \
	&& cd /tmp \
	&& mkdir openssl \
	&& curl -sL "https://www.openssl.org/source/openssl-$OPENSSL_VERSION.tar.gz" -o openssl.tar.gz \
	&& tar -xzf openssl.tar.gz -C openssl --strip-components=1 \
	&& cd /tmp/openssl \
	&& ./config -fPIC && make && make install \
	&& rm -rf /tmp/* \
	&& export PHP_OPENSSL_DIR=yes

ENV PHP_VERSION 5.3.29
COPY docker-php-* /usr/local/bin/
COPY docker-php-source /usr/local/bin/

RUN buildDeps=" \
	apache2-dev \
	autoconf2.13 \
	libcurl4-openssl-dev \
	libpcre3-dev \
	libreadline6-dev \
	librecode-dev \
	libsqlite3-dev \
	libssl-dev \
	libxml2-dev \
	libpng-dev \
	libfreetype6-dev \
	libjpeg62-turbo-dev \
	xz-utils \
	" \
	&& apt-get update && apt-get install -y $buildDeps --no-install-recommends && rm -rf /var/lib/apt/lists/* \
	&& set -x \
	&& mkdir -p /usr/src/php \
	&& curl -SL "http://php.net/get/php-$PHP_VERSION.tar.xz/from/this/mirror" -o /usr/src/php.tar.xz \
	&& cd /usr/src/ \
	&& docker-php-source extract \
	&& mkdir -p /usr/include/freetype2/freetype \
	&& ln -s /usr/include/freetype2/freetype.h /usr/include/freetype2/freetype/freetype.h \
	&& cd /usr/src/php \
	&& ./configure --disable-cgi \
	$(command -v apxs2 > /dev/null 2>&1 && echo '--with-apxs2=/usr/bin/apxs2' || true) \
	--with-config-file-path="$PHP_INI_DIR" \
	--with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
	--enable-ftp \
	--enable-mbstring \
	--enable-mysqlnd \
	--with-mysql \
	--with-mysqli \
	--with-pdo-mysql \
	--with-curl \
	--with-openssl=/usr/local/ssl \
	--enable-soap \
	--with-png \
	--with-gd \
	--with-readline \
	--with-recode \
	--with-zlib \
	--enable-zip \
	--with-bz2 \
	--with-up-dir \
	--with-pcre-regex \
	&& make -j"$(nproc)" \
	&& make install \
	&& { find /usr/local/bin /usr/local/sbin -type f -executable -exec strip --strip-all '{}' + || true; } \
	# find extensions's so and put it in php extensions folder & "produce ini files" in php/conf.d
	&& docker-php-ext-install -j$(nproc) exif hash sockets xml zip pcntl bcmath gettext \
	&& docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/  \
	&& docker-php-ext-install -j$(nproc) gd \
	&& docker-php-ext-install -j$(nproc) pdo_mysql mysqli \
	&& cd / \
	&& docker-php-source delete \
	&& apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false $buildDeps \
	&& rm -rf /var/lib/apt/lists/*

# RUN echo "default_charset = " > $PHP_INI_DIR/php.ini \
# && echo "date.timezone = America/Sao_Paulo" >> $PHP_INI_DIR/php.ini

#安裝composer
RUN EXPECTED_SIGNATURE=$(wget -q -O - https://composer.github.io/installer.sig); \
	php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"; \
	ACTUAL_SIGNATURE=$(php -r "echo hash_file('SHA384', 'composer-setup.php');"); \
	php composer-setup.php; \
	php -r "unlink('composer-setup.php');"; \
	mv composer.phar /usr/local/bin/composer; 

RUN  pecl install xdebug-2.2.4 \
	#  pecl install redis \
	#  pecl install imagick \
	&& docker-php-ext-enable xdebug


ENTRYPOINT ["docker-php-entrypoint"]
##<autogenerated>##
COPY apache2-foreground /usr/local/bin/
WORKDIR /var/www/html

RUN	echo 'export TERM=xterm-256color' >> ~/.bashrc; \
	echo 'alias ll="ls -al"' >> ~/.bashrc; \
	mkdir -p /var/lib/php/session; \
	chown -R www-data:www-data /var/lib/php/session; \
	a2enmod ssl rewrite 

EXPOSE 80 443 3000 8080
CMD ["apache2-foreground"]
