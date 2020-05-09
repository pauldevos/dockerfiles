### Docker Docs

server: basketballdb
hostname: localhost
username: postgres
password: postgres

- **SSH into Docker container**

```bash
docker exec -it 3cdb7385c127 bash -l
# to exit
ctrl + d
```

https://www.saltycrane.com/blog/2019/01/how-run-postgresql-docker-mac-local-development/

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
