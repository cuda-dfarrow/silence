FROM mysql:5.6
MAINTAINER farrow.daveb@gmail.com

COPY setup.sql /docker-entrypoint-initdb.d/
COPY mysql-root /run/secrets/
