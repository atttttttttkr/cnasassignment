FROM mysql:8.0

# Set environment variables (override as needed)
ENV MYSQL_DATABASE=mydb \
    MYSQL_USER=crud_user \
    MYSQL_PASSWORD=crud_pass \
    MYSQL_ROOT_PASSWORD=rootpass

# Copy your SQL script into the auto-init directory
COPY users.sql /docker-entrypoint-initdb.d/
