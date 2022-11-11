ARG NGINX_VERSION=mainline

FROM nginx:${NGINX_VERSION}-alpine

COPY ./conf.d/default.conf /etc/nginx/conf.d/

WORKDIR /var/www
