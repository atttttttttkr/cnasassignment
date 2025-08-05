# Use the official PHP image as the base image
FROM php:latest
RUN docker-php-ext-install mysqli
#Copy the PHP files to the container
COPY . /var/www/html
# Expose port 8080 for web traffic
EXPOSE 8080
# Set the working directory
WORKDIR /var/www/html

CMD ["php","-S", "0.0.0.0:8080"]
