# FROM mysql:8.0
# COPY db/ /docker-entrypoint-initdb.d
# ENV MYSQL_ROOT_PASSWORD="RoboShop@1"

# created part of K8 init containers
FROM mysql:8.0
COPY db/ /docker-entrypoint-initdb.d
COPY entrypoint.sh /usr/local/bin/entrypoint.sh
RUN chmod +x c
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]