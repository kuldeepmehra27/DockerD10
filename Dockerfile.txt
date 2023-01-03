# Base image
FROM ubuntu:22.04
RUN echo "It is working fine!!"

ENV TZ=Asia/Kolkata DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install tzdata

# Work directory
WORKDIR /var/www/html

# Install Apache
RUN apt-get -y install apache2 
RUN apt-get -y install apache2-utils 
RUN apt-get clean 
EXPOSE 80

# Enable rewrite module
RUN a2enmod rewrite

# Install PHP
RUN apt-get -y install --no-install-recommends php8.1
RUN apt-get -y install php8.1-cli php8.1-common php8.1-mysql php8.1-zip php8.1-gd php8.1-mbstring php8.1-curl php8.1-xml php8.1-bcmath

# Install composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
	php composer-setup.php && \
	mv composer.phar /usr/local/bin/composer && \
	php -r "unlink('composer-setup.php');"
ENV PATH="~/.composer/vendor/bin:${PATH}"

# Remove apache default index html file
RUN rm -rf index.html

# Define path variable
# ARG SITE_PATH=/var/www/html
# ENV PATH=$SITE_PATH

# Create drupal 10 project using composer into current directory
RUN composer create-project drupal/recommended-project:10.0.0 .

# Create files folder
RUN mkdir /var/www/html/web/sites/default/files
# Give permission to files folder
RUN chmod a+w /var/www/html/web/sites/default/files

# Copy settings file
RUN cp /var/www/html/web/sites/default/default.settings.php /var/www/html/web/sites/default/settings.php
# Give permission to settings file
RUN chmod a+w /var/www/html/web/sites/default/settings.php

# Add servername to apache config file
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf

# This is resolve 404 isseue
RUN echo "\n<Directory /var/www/>" >> /etc/apache2/apache2.conf
RUN echo "\t Options Indexes FollowSymLinks" >> /etc/apache2/apache2.conf
RUN echo "\t AllowOverride All" >> /etc/apache2/apache2.conf
RUN echo "\t Require all granted" >> /etc/apache2/apache2.conf
RUN echo "</Directory>" >> /etc/apache2/apache2.conf

RUN echo "Done, please browse to http://127.0.0.1:8000/web to check!"
RUN service apache2 restart


#Install drush
RUN composer global require drush/drush

CMD ["/usr/sbin/apachectl", "-D", "FOREGROUND"]
