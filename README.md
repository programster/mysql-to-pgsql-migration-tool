MySQL To PostgreSQL Migration Tool
==================================

This tool aims to make it easy to convert an existing MySQL database over to a 
PostgreSQL one with the help of Docker and 
[PgLoader](https://github.com/dimitri/pgloader).


### Prerequisites
* Docker
* Docker-compose


## Steps
Create a `.env` file from the `.env.example` file. Feel free to change the
values but technically you dont need to. This should not be used for running
anything in production and is only meant to be run locally for generating a
PostgreSQL dump file that you would import into another database that has
different credentials.


Run the following commands to spin up the MySQL and PostgreSQL databases.

```bash
docker-compose up mysql -d
docker-compose up pgsql -d
```

Now import the database by running the following commands, making sure to 
change `MYSQL_DUMP_FILE` to point to wherever your local MySQL dump file is.

```bash
# Import values from .env file.
set -a; . .env; set +a

MYSQL_DUMP_FILE="dump.sql"

mysql \
  -u root \
  -p$MYSQL_ROOT_PASSWORD \
  -h 127.0.0.1 \
  $MYSQL_DB_NAME < $MYSQL_DUMP_FILE
```

That will have imported your mysql database. Now we need to get pgloader to
connect to it and transfer the data over to postgresql.

```bash
set -a; . .env; set +a

docker-compose up pgloader
```

Pgloader will now have moved your data across. Be sure to check the output for 
any issues that may have arisen. You may need to manually change your database 
or dump file in order to resolve any of the issues. E.g. dates with `0000-00-00` 
get converted to null, which may fail to be inserted, due to a `NOT NULL` 
constraint.

Now dump your database with the following commands:

```bash
# load the environment variables
set -a; . .env; set +a

pg_dump  \
  --host pgsql \
  --port "5432" \
  --username $PG_USER \
  --file myPostgresqlDumpFile.sql \
  $PG_DB_NAME
```

That's it! You now have a PostgreSQL dump file that you can import into a
PostgreSQL database with:

```bash
psql \
  --host $HOST \
  --port $PORT \
  --username $USERNAME \
  -d $DATABASE_NAME \
  -f $DUMP_FILEPATH
```