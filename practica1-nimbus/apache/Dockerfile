FROM debian:bookworm-slim

RUN apt-get update && \
    apt-get install -y --no-install-recommends apache2 && \
    rm -rf /var/lib/apt/lists/*

RUN a2enmod mpm_event

RUN a2dissite 000-default.conf

COPY sites/ /etc/apache2/sites-available/

COPY web/nimbus-legal/ /var/www/nimbus-legal/
COPY web/nimbus-tech/ /var/www/nimbus-tech/

RUN a2ensite nimbus-legal.conf nimbus-tech.conf 000-default.conf

EXPOSE 80

CMD ["apachectl", "-D", "FOREGROUND"]
