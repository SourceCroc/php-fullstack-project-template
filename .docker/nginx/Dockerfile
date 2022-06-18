FROM nginx:latest
ARG UID=1000
ARG GID=1000

RUN usermod -u $UID -o nginx && groupmod -g $GID -o nginx; \
    mkdir -p /var/log/nginx; \
    chown -R $UID:$GID /var/log/nginx;
