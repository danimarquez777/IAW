FROM nginx:stable

RUN rm -f /etc/nginx/conf.d/default.conf

COPY conf.d/ /etc/nginx/conf.d/

COPY certs/ /etc/nginx/certs/

EXPOSE 80 443
