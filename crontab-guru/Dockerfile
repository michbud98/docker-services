FROM nginx:1.25

RUN apt-get update && apt-get install wget -y

RUN mkdir -p /var/www/html
WORKDIR /var/www/html

# Copy crontab.guru
RUN wget -m https://crontab.guru/ 

COPY config/nginx.conf /etc/nginx/conf.d/default.conf

ENTRYPOINT [ "nginx", "-g", "daemon off;" ]
