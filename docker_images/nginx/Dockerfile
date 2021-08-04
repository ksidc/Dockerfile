FROM alpine:3.10

LABEL maintainer="KSIDC <rlathwls@ksidc.net>"

# nginx install
RUN set -eux; \
	apk add --no-cache --virtual .build-deps \
	git \
	gcc \
	make \
	zlib-dev \
	perl-dev \
	libc-dev \
	openssl \
	pcre-dev \
  	; \
	\
	# openssl
	mkdir /usr/local/certificates; \
	openssl req \
	-x509 \
	-newkey rsa:2048 \
	-keyout /usr/local/certificates/key.pem \
	-out /usr/local/certificates/cert.pem \
	-days 365 \
	-nodes \
	-subj /CN=localhost \
	; \
	rm -rf /var/cache/apk/*; \
	\
	cd /tmp; \
	wget https://www.openssl.org/source/openssl-1.0.2t.tar.gz; \
	wget http://zlib.net/zlib-1.2.11.tar.gz; \
	tar zxvf openssl-1.0.2t.tar.gz; \
	tar zxvf zlib-1.2.11.tar.gz; \
	\
	# nginx install
	wget https://github.com/ksidc/nginx/archive/nginx-ksidc-1.17.6.tar.gz; \
	tar zxvf nginx-ksidc-1.17.6.tar.gz; \
	cd nginx-nginx-ksidc-1.17.6/; \
 	chmod +x ./configure; \
	addgroup -g 101 -S nginx; \
	adduser -S -D -H -u 101 -h /usr/local -s /sbin/nologin -G nginx -g nginx nginx; \
	\
	./configure \
	--prefix=/usr/local \
	--with-zlib=/tmp/zlib-1.2.11 \
	--with-pcre \
	--with-openssl=/tmp/openssl-1.0.2t \
	--with-http_mp4_module \
	--with-http_stub_status_module \
	--user=nginx \
	--group=nginx \
	--with-http_ssl_module \
	--with-http_v2_module \
	--with-http_gzip_static_module \
	; \
	make -j 8; \
	make install; \
	find /usr/local/bin /usr/local/sbin -type f -perm +0111 -exec strip --strip-all '{}' + || true; \
	cd /; \
	rm -rf /tmp/*; \
	apk del .build-deps; \
	\
	runDeps="$( \
	scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
	| tr ',' '\n' \
	| sort -u \
	| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' )"; \
	\
	apk add --no-cache $runDeps;\
	\
	cd /usr/local; \
	mkdir conf.d


EXPOSE 80 443

STOPSIGNAL SIGTERM

CMD ["/usr/local/sbin/nginx", "-g", "daemon off;"]

