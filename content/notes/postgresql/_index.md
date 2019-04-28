---
title: "PostgreSQL"
authors: ["gio"]
---

#### Common Commands

postgres=# CREATE ROLE pathfinder_mono;
postgres=# ALTER ROLE pathfinder_mono WITH LOGIN;
postgres=# ALTER ROLE pathfinder_mono WITH SUPERUSER;
postgres=# ALTER ROLE pathfinder_mono WITH PASSWORD 'pathfinder_mono';

#### Creating new database with specified collation

If we're creating new database with specific collation, sometimes it will return error like this:

`ERROR:  new collation (C.UTF-8) is incompatible with the collation of the template    database (en_US.UTF8)`

If this happens, we should specify template0 as the template by specifying `-T template0`

See: [here](https://stackoverflow.com/questions/18870775/how-to-change-the-template-database-collection-coding-on-postgresql)

#### Useful queries for monitoring

See: [here](https://github.com/nilenso/postgresql-monitoring)
