# FROM mysql:8.0
# COPY db/ /docker-entrypoint-initdb.d
# ENV MYSQL_ROOT_PASSWORD="RoboShop@1"

# created part of K8 init containers
FROM mysql:8.0
COPY db/ /docker-entrypoint-initdb.d
COPY script.sh /usr/local/bin/script.sh
RUN chmod +x /usr/local/bin/script.sh
ENTRYPOINT ["/usr/local/bin/script.sh"]