FROM alpine:3.10

LABEL maintainer="KSIDC <rlathwls@ksidc.net>"

ENV BUILD_VERS=3.0

ENV PHPIZE_DEPS \
		autoconf \
		file \
		re2c \
		g++ \
                gcc \
                libc-dev \
                make \
		curl

ENV PHP_URL "https://www.php.net/distributions/php-7.3.12.tar.xz"
ENV PHP_INI_DIR /etc

ENV PHP_CFLAGS="-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64"
ENV PHP_CPPFLAGS="$PHP_CFLAGS"
ENV PHP_LDFLAGS="-Wl,-O1 -Wl,--hash-style=both -pie"

# script copy
COPY docker-php-source docker-php-ext-* docker-php-entrypoint /usr/local/bin/ 

ENTRYPOINT ["docker-php-entrypoint"]

RUN set -eux; \
	apk add --no-cache --virtual .build-deps \
	$PHPIZE_DEPS \
	curl-dev \
	geoip-dev \
        libedit-dev \
	libxml2-dev \
	libsodium-dev \
	openssl-dev \
	jpeg-dev \
	libpng-dev \
	freetype-dev \
	gd-dev \
	libxpm-dev \
	xmlrpc-c-dev \
	gnu-libiconv-dev \
        libzip-dev \
	; \
	\
	# php7 download
	docker-php-source install; \
	docker-php-source extract; \
	\
        # php7 install
	cd /usr/src/php/; \
	./configure \
		--prefix=/usr/local \
		--sysconfdir=/etc \
                --with-config-file-path="$PHP_INI_DIR" \
                --with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
                --with-mhash \
                --enable-ftp \
                --enable-mbstring \
                --enable-mysqlnd \
                --with-mysqli=mysqlnd \
                --with-sodium=shared \
                --with-curl \
                --with-libedit \
                --with-openssl \
                --with-zlib \
                --with-libxml-dir=/usr \
                --with-iconv-dir=/usr \
                --with-jpeg-dir=/usr \
                --with-png-dir=/usr \
                --with-freetype-dir=/usr \
                --with-xpm-dir=/usr \
                --enable-mbregex \
                --enable-sigchild \
                --enable-sockets \
                --enable-inline-optimization \
                --enable-sysvsem \
                --enable-sysvshm \
                --enable-wddx \
                --enable-bcmath \
                --enable-exif \
                --with-xmlrpc \
                --enable-calendar \
                --with-imap-ssl= \
                --with-kerberos= \
                --enable-zip \
                --with-zlib-dir=/usr \
                --enable-soap \
                --with-gd=/usr \
                --with-libdir=/lib \
                --enable-fpm \
                --with-fpm-user=nobody \
                --with-fpm-group=nobody \
	; \
	make -j 8; \
	make install; \
	\
	find /usr/local/bin /usr/local/sbin -type f -perm +0111 -exec strip --strip-all '{}' + || true; \
	find -type f -name '*.a' -delete; \
	\
	# php.ini conf file setting
	cp php.ini-development /etc/php.ini; \
	\
	# php-fpm conf file setting
        cd /etc; \
        \
	# for php 7.x setting
        if [ -d php-fpm.d ]; then \
                sed 's!=NONE/!=!g' php-fpm.conf.default | tee php-fpm.conf > /dev/null; \
                cp php-fpm.d/www.conf.default php-fpm.d/www.conf; \
        # for php 5.x setting
	else \
                mkdir php-fpm.d; \
                cp php-fpm.conf.default php-fpm.d/www.conf; \
                { \
                        echo '[global]'; \
                        echo 'include=/etc/php-fpm.d/*.conf'; \
                } | tee php-fpm.conf; \
        fi; \
        \
	{ \
                echo '[global]'; \
                echo 'error_log = /dev/stderr'; \
                echo; \
                echo '[www]'; \
                echo '; if we send this to /proc/self/fd/1, it never appears'; \
                echo 'access.log = /dev/stdout'; \
                echo; \
                echo 'clear_env = no'; \
                echo; \
                echo '; Ensure worker stdout and stderr are sent to the main error log.'; \
                echo 'catch_workers_output = yes'; \
        } | tee php-fpm.d/docker.conf; \
        { \
                echo '[global]'; \
                echo 'daemonize = no'; \
                echo; \
                echo '[www]'; \
                echo 'listen = 9000'; \
        } | tee php-fpm.d/zz-docker.conf; \
	\
	\
	mkdir -p $PHP_INI_DIR/conf.d; \
	\
	# sodium modules install
	docker-php-ext-enable sodium; \
	\
	# pcntl modules install
	docker-php-ext-configure pcntl --with-php-config=/usr/local/bin/php-config; \
        docker-php-ext-install pcntl; \
	\
	# geoip module install	
	cd /usr/src; \
	wget https://pecl.php.net/get/geoip-1.1.1.tgz; \
	tar zxvf geoip-1.1.1.tgz; \
	mv geoip-1.1.1/ php/ext/geoip; \
	\
	docker-php-ext-configure geoip --with-php-config=/usr/local/bin/php-config; \
	docker-php-ext-install geoip; \
	\
	# source file delete
	docker-php-source delete; \
	\
	# runtime dependancy packages install & build-deps delete
	runDeps="$( \
	scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
	| tr ',' '\n' \
	| sort -u \
	| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' )"; \
	\
	apk add --no-cache $runDeps; \
	apk del --no-network .build-deps


EXPOSE 9000

STOPSIGNAL SIGQUIT

CMD ["php-fpm"]
