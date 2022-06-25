FROM nginx:alpine

# Delete Default Nginx configuration files
RUN rm -rf /usr/share/nginx/html/* /etc/nginx/conf.d/*

# Copy Sulu site configuration files
COPY docker/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf