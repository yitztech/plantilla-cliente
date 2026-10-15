# Web de inicio: una página estática servida por nginx en el puerto 80.
# Sustituir por la aplicación real (mismo servicio `web`, o ajustar el upstream del gateway),
# manteniendo /healthz para su healthcheck.
FROM nginx:1.30-alpine

RUN rm /etc/nginx/conf.d/default.conf
COPY nginx/nginx.conf /etc/nginx/nginx.conf
COPY web/default.conf /etc/nginx/conf.d/web.conf
COPY web/public/ /usr/share/nginx/html/

USER nginx
EXPOSE 80
