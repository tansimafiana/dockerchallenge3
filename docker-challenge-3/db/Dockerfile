FROM mariadb

# OLD
#COPY docker/db/init/init.sql /docker-entrypoint-initdb.d

# NEW
ENV MARIADB_ROOT_PASSWORD=fiona

COPY ./init/*.sql /docker-entrypoint-initdb.d

RUN echo "DB Initialized"