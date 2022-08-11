FROM nginx:1.23.1

RUN apt-get update && apt-get install wget -y

RUN mkdir -p /var/www/html
WORKDIR /var/www/html

# Copy crontab.guru
RUN wget -m https://crontab.guru/

ENTRYPOINT [ "nginx", "-g", "daemon off;" ]
