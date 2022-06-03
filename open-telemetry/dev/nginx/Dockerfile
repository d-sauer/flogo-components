FROM nginx:1.19.8

COPY config/nginx.conf /etc/nginx/nginx.conf
COPY config/otel-nginx.toml /etc/nginx/otel-nginx.toml
CMD ["/usr/sbin/nginx", "-g", "daemon off;"]


EXPOSE 80 443

