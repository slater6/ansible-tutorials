FROM nginx:1.12.2

RUN apt-get update && apt-get install -y python2.7 \
    && apt-get clean \
    && rm -rf /var/lib/apt/list/* /tmp/* /var/tmp/* \
    && echo "daemon off;" >> /etc/nginx/nginx.conf

EXPOSE 80

STOPSIGNAL SIGTERM

CMD ["nginx", "-g", "daemon off;"]