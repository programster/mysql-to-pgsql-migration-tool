FROM ubuntu:20.04

RUN apt-get update && apt-get dist-upgrade -y


RUN apt-get install -y wget gnupg2 \
    && sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt focal-pgdg main" > /etc/apt/sources.list.d/pgdg.list' \
    && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add - \
    && apt-get update && apt-get install -y postgresql-client-13