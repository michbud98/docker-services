FROM nginx:1.23.1

RUN mkdir -p /var/www/html
WORKDIR /var/www/html

ADD ./Wireshark-WPA-PSK-Generator/ ./Wireshark-WPA-PSK-Generator/
COPY Wireshark-WPA-PSK-Generator.html .

#CMD /bin/sh
ENTRYPOINT [ "nginx", "-g", "daemon off;" ]
