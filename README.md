MySQL To PostgreSQL Migration Tool
==================================

This tool aims to make it easy to convert an existing MySQL database over to a
PostgreSQL one with the help of Docker and
[PgLoader](https://github.com/dimitri/pgloader).


### Prerequisites
* Docker
* Docker-compose


## Steps

### Create Environment File
Create a `.env` file from the `.env.example` file. Feel free to change the
values but technically you dont need to. This should not be used for running
anything in production and is only meant to be run locally for generating a
PostgreSQL dump file that you would import into another database that has
different credentials.


### Spin Up Temporary Databases
Run the following commands to spin up the MySQL and PostgreSQL databases.

```bash
docker-compose up -d mysql
docker-compose up -d pgsql
```

### Import MySQL Dump
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

### Migrate With PGLoader
That will have imported your mysql database. Now we need to get pgloader to
connect to it and transfer the data over to postgresql.

```bash
set -a; . .env; set +a

docker-compose up pgloader
```

PGLoader will now have copied your data across. Be sure to check the output for
any issues that may have arisen. You may need to manually change your database
or dump file in order to resolve any of the issues. E.g. dates with `0000-00-00`
get converted to null, which may fail to be inserted, due to a `NOT NULL`
constraint.

### Create PostgreSQL Dump-file
Now dump your database by running the pgdumper service:

```bash
docker-compose up pgdumper
```

Note: we created a service for this, to prevent users from having to install the postgresql client,
and also to prevent any issues with mismatched versions.

### Import PostgreSQL Database Dump
You now have a PostgreSQL dump file in the `output/` folder, which you can import into a
PostgreSQL database with:

```bash
psql \
  --host $HOST \
  --port $PORT \
  --username $USERNAME \
  -d $DATABASE_NAME \
  -f $DUMP_FILEPATH
```


### A Note About PGLoader Created Schema
Note: the PGLoader tool will have created a schema with the same name as the MySQL database. 
You *may* wish to keep this. Alternatively, you may wish to perform the following operation after 
importing your database:

```sql
DROP SCHEMA 'public';
RENAME SCHEMA '$MYSQL_DB_NAME' TO 'public';
```

Alternatively, you may wish to 
[set your search path](https://www.postgresql.org/docs/9.6/ddl-schemas.html#DDL-SCHEMAS-PATH), 
so that you can continue to access these tables using *unqualified names*:

```sql
SET search_path TO '$MYSQL_DB_NAME',public;
```


### Cleanup
Now you can "clean up" by running the following command to spin-down and remove the running MySQL
and PostgreSQL containers.

```bash
docker-compose down
```
