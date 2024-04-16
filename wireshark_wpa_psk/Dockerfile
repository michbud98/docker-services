FROM nginx:1.25

RUN mkdir -p /var/www/html
WORKDIR /var/www/html

COPY ./Wireshark-WPA-PSK-Generator/ ./Wireshark-WPA-PSK-Generator/
COPY Wireshark-WPA-PSK-Generator.html .
COPY config/nginx.conf /etc/nginx/conf.d/default.conf

#CMD /bin/sh
ENTRYPOINT [ "nginx", "-g", "daemon off;" ]
