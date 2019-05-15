Update PostgreSQL data directories.

# Quickstart

1. Get Docker and docker-compose
2. Put a copy of the source data directory in `./old`
3. Add some settings to docker-compose.yml
4. Start the container
5. Replace the source data directory with `./work/new`

# Overview

The migration is done by installing the old and new version of
PostgreSQL in a container and then run `pg_upgrade`. All of the work
happens in the upgrade script; it boils down to:

- install some dependencies
- set the right locale
- install PostgreSQL from the official packages
- set the right permissions so `pg_upgrade` can read the source data
- create a new database via initdb (with the right superuser)
- use `pg_upgrade` to migrate from old to new database
- copy configuration files
- start new PostgreSQL version with migrated database
- run generated analyze script
- stop PostgreSQL

# Configuration

The upgrade requires some settings which need to be passed as
environment variables to the container running the upgrade script.
As such, you need to set them in the `docker-compose.yml` before
starting the container.

If you are using an environment file to keep your database settings
for your app container, you can also us that for the upgrade. To do
so, edit the `docker-compose.yml` and add it to the `env_file` list.

The following lists the configurable settings and their default
values:

Environment variable | Description                                 | Default value
-------------------- | ------------------------------------------- | ----------------------------
NEW_VERSION          | Target version of PostgreSQL                | <NONE>
OLD_VERSION          | Source version of PostgreSQL                | content of `/old/PG_VERSION`
POSTGRES_USER        | Name of the database superuser              | postgres
POSTGRES_PASSWORD    | Password of the database superuser          | <NONE>

Only `NEW_VERSION` is mandatory, as the defaults should be fine for
the other settings most of the time.

## Location of the databases

The source database needs to be available via `/old` in the container.
The default `docker-compose.yml` binds `./old` to that path.

During the migration, several files will be created (e.g., logs) by
the postgres user. These will be written to `/work` in the container.
By default, we bind `./work` to that path, similar as `./old`.

The migrated, new database will be written to `/work/new` in the
container.

# Migrating the data

I highly advise running the container non-detached so you can follow what happens:

```
docker-compose up --build
```

If everything is okay, the script will exit with code 0.
Otherwise, you should look at the logs in `./work` to find the cause of the error.

