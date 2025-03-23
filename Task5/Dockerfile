FROM richarvey/nginx-php-fpm:latest

RUN set -ex; \
	wget "https://wordpress.org/wordpress-5.0.tar.gz"; \
	tar -xzf wordpress-5.0.tar.gz -C /var/www/html/; \
	chown -R www-data:www-data /var/www/html/

COPY ./nginx.conf /etc/nginx/sites-available/default.conf