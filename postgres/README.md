# Setting up Postgres Docker Container for your NBA Analytics Database

1. Install Docker
2. Configure Dockerfile or Docker-Compose file (2 options, I prefer Docker-Compose)
3. Have fun!

- [How to run PostgreSQL in Docker on Mac (for local development)](https://www.saltycrane.com/blog/2019/01/how-run-postgresql-docker-mac-local-development/)

### Setting up Docker Alpine-Postgres Image

This documentation is taken directly from the [Alpine Postgres Docker hub](https://hub.docker.com/r/yobasystems/alpine-postgres).

#### Environment Variables:

The PostgreSQL image uses several environment variables which are easy to miss. While **none** of the variables are required, setting a password as a minimum will ensure some degree of security when using the image.

#### Main Postgres parameters:

- `POSTGRES_PASSWORD`: This environment variable is recommended for you to use the PostgreSQL image. This environment variable sets the superuser password for PostgreSQL. The default superuser is defined by the POSTGRES_USER environment variable. In the below example, it is being set to "RaNd0MpA55W0Rd".

- `POSTGRES_USER`: This optional environment variable is used in conjunction with POSTGRES_PASSWORD to set a user and its password. This variable will create the specified user with superuser power and a database with the same name. If it is not specified, then the default user of "postgres" will be used.

- `PGDATA`: This optional environment variable can be used to define another location - like a subdirectory - for the database files. The default is `/var/lib/postgresql/data`, but if the data volume you're using is a fs mountpoint (like with GCE persistent disks), Postgres initdb recommends a subdirectory (for example /var/lib/postgresql/data/pgdata ) be created to contain the data.

- `POSTGRES_DB`: This optional environment variable can be used to define a different name for the default database that is created when the image is first started. If it is not specified, then the value of `POSTGRES_USER` will be used.

- `POSTGRES_INITDB_ARGS`: This optional environment variable can be used to send arguments to postgres initdb. The value is a space separated string of arguments as postgres initdb would expect them. This is useful for adding functionality like data page checksums: -e POSTGRES_INITDB_ARGS="--data-checksums".

### Using Docker: Creating an instance using Docker (not my preferred way)

```bash
docker run --name some-postgres -e POSTGRES_PASSWORD=RaNd0MpA55W0Rd -d yobasystems/alpine-postgres
```

This will create a new db called `postgres`, with user `postgres` and set root password of `RaNd0MpA55W0Rd`.

### Using Docker-Compose: Creating an instance (my Preferred way)

```bash
# docker-compose.yml file
db:
    image: yobasystems/alpine-postgres
    environment:
        POSTGRES_DB: basketball
        POSTGRES_USER: cbarkley
        POSTGRES_PASSWORD=76ers # no space if space, use quotes like variables below
    expose:
        - "5432"
    volumes: #choose_your_location:postgres_default (don't change)
        - /data/postgres/pgdata:/var/lib/postgresql/data
    restart: always
```

### Docker Volumes

There's two main ways to define or create volumes in Docker (or Docker-compose). One is a `named volume` and the other is a `bind volume`. The `named` volume will be created in the directory nested deep within your OS in that home Docker directory. IF you would like a volume in a different local path, you need to use a `bind` volume instead where you can define it's path.

### Main Docker Commands I use

```bash
# this starts up the container using the filename.yml configuration
# -d starts the containers in the background and leaves them running.
docker-compose -f filename.yml up -d
docker-compose -f postgres-compose.yml up -d # more specific

# this stops the container
docker-compose -f filename.yml stop -d

# this stops AND destroys the container - LOSE all state info that is not external to the container, e.g. volumes should be
docker-compose -f filename.yaml down -d

# equiv of ssh'ing into container
docker exec -it container_id bash
```

### Docker-Compose Documentation

- https://docs.docker.com/compose/reference/overview/

```bash
Usage:
  docker-compose [-f <arg>...] [options] [COMMAND] [ARGS...]
  docker-compose -h|--help

Options:
  -f, --file FILE             Specify an alternate compose file
                              (default: docker-compose.yml)
  -p, --project-name NAME     Specify an alternate project name
                              (default: directory name)
  --verbose                   Show more output
  --log-level LEVEL           Set log level (DEBUG, INFO, WARNING, ERROR, CRITICAL)
  --no-ansi                   Do not print ANSI control characters
  -v, --version               Print version and exit
  -H, --host HOST             Daemon socket to connect to

  --tls                       Use TLS; implied by --tlsverify
  --tlscacert CA_PATH         Trust certs signed only by this CA
  --tlscert CLIENT_CERT_PATH  Path to TLS certificate file
  --tlskey TLS_KEY_PATH       Path to TLS key file
  --tlsverify                 Use TLS and verify the remote
  --skip-hostname-check       Don't check the daemon's hostname against the
                              name specified in the client certificate
  --project-directory PATH    Specify an alternate working directory
                              (default: the path of the Compose file)
  --compatibility             If set, Compose will attempt to convert deploy
                              keys in v3 files to their non-Swarm equivalent

Commands:
  build              Build or rebuild services
  bundle             Generate a Docker bundle from the Compose file
  config             Validate and view the Compose file
  create             Create services
  down               Stop and remove containers, networks, images, and volumes
  events             Receive real time events from containers
  exec               Execute a command in a running container
  help               Get help on a command
  images             List images
  kill               Kill containers
  logs               View output from containers
  pause              Pause services
  port               Print the public port for a port binding
  ps                 List containers
  pull               Pull service images
  push               Push service images
  restart            Restart services
  rm                 Remove stopped containers
  run                Run a one-off command
  scale              Set number of containers for a service
  start              Start services
  stop               Stop services
  top                Display the running processes
  unpause            Unpause services
  up                 Create and start containers
  version            Show the Docker-Compose version information
```

### Other Resources

- [Using volumes in Docker Compose](https://devopsheaven.com/docker/docker-compose/volumes/2018/01/16/volumes-in-docker-compose.html)
- [How to purge Docker images, containers, networks or volumes](https://devopsheaven.com/docker/volumes/purge/devops/2018/05/25/purge-docker-images-containers-networks-volumes.html)
- [Using psql (command line)](https://devopsheaven.com/tutorial/postgresql)

### Backup Postgres DB

- [pg_dump](https://www.postgresql.org/docs/current/backup-dump.html)

### SQLalchemy and Postgres

- [Beginner's Guide to Using Databases with Python: Postgres, SQLAlchemy, and Alembic](https://www.learndatasci.com/tutorials/using-databases-python-postgres-sqlalchemy-and-alembic/)

---

docker exec CONTAINER_NAME pg_dump -U USER --format=c DBNAME > mydump.sqlc

- https://stackoverflow.com/questions/39210274/how-do-i-backup-a-database-in-docker
- https://blog.koley.in/2019/backup-and-restore-postgresql-database-running-on-docker
- https://www.petefreitag.com/item/889.cfm

To run pg_dump you can use docker exec command:

To backup:

docker exec -u <your_postgres_user> <postgres_container_name> pg_dump -Fc <database_name_here> > db.dump

To drop db (Don't do it on production, for test purpose only!!!):

docker exec -u <your_postgres_user> <postgres_container_name> psql -c 'DROP DATABASE <your_db_name>'

To restore:

docker exec -i -u <your_postgres_user> <postgres_container_name> pg_restore -C -d postgres < db.dump

Also you can use docker-compose analog of exec. In that case you can use short services name (postgres) instead of full container name (composeproject_postgres).

https://www.petefreitag.com/item/880.cfm
