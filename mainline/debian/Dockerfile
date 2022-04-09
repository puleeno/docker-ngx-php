FROM debian:bullseye-slim as ngx-php

ENV NGINX_VERSION=1.21.6
ENV PHP_LIB=/usr/lib

RUN apt update \
  && apt install -y curl gnupg2 software-properties-common

RUN curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor | tee /usr/share/keyrings/nginx-archive-keyring.gpg
RUN apt-key add /usr/share/keyrings/nginx-archive-keyring.gpg

RUN add-apt-repository "deb http://nginx.org/packages/debian `lsb_release -cs` nginx"

RUN apt update \
  && apt install -y nginx

RUN curl https://packages.sury.org/php/apt.gpg -o /etc/apt/trusted.gpg.d/php.gpg \
  && echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/php.list

RUN apt update \
  && apt install -y build-essential systemtap-sdt-dev php8.1-dev libphp8.1-embed libxml2-dev libsodium-dev libpcre3 libpcre3-dev zlib1g zlib1g-dev libargon2-dev \
  && apt install -y php8.1-mysqlnd php8.1-gd php8.1-zip php8.1-sqlite3 php8.1-xml

RUN cd / && curl -L -k https://github.com/rryqszq4/ngx-php/tarball/master -o ngx-php.tgz \
  && tar -xvf ngx-php.tgz \
  && mv rryqszq4-ngx-php* ngx-php

RUN cd /ngx-php \
  && curl -sL -o nginx-${NGINX_VERSION}.tar.gz http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz \
  && tar -xf nginx-${NGINX_VERSION}.tar.gz \
  && cd nginx-${NGINX_VERSION} \
  && ./configure --with-compat --add-dynamic-module=../ \
  && make modules \
  && mv objs/ngx_http_php_module.so /usr/lib/nginx/modules


FROM nginx:1.21.6
RUN apt update \
  && apt install -y curl gnupg2 software-properties-common

RUN curl https://packages.sury.org/php/apt.gpg -o /etc/apt/trusted.gpg.d/php.gpg \
  && echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/php.list

RUN rm -rf /etc/nginx/modules && mkdir /etc/nginx/modules

RUN apt update && apt install -y libphp8.1-embed \
  && sed -i "s|events {|include /etc/nginx/modules/\*.conf;\n\nevents {|" /etc/nginx/nginx.conf \
  && sed -i "s#index  index.html index.htm;#content_by_php_block {\n            phpinfo();\n        }#" /etc/nginx/conf.d/default.conf \
  && echo 'load_module "/usr/lib/nginx/modules/ngx_http_php_module.so";' > /etc/nginx/modules/php.conf

COPY --from=ngx-php /usr/lib/nginx/modules/ /usr/lib/nginx/modules/
