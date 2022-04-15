---
title: "PostgreSQL"
authors: ["gio"]
---

#### Common Commands

```
CREATE ROLE pathfinder_mono;
ALTER ROLE pathfinder_mono WITH LOGIN;
ALTER ROLE pathfinder_mono WITH SUPERUSER;
ALTER ROLE pathfinder_mono WITH PASSWORD 'pathfinder_mono';

SHOW max_connections;
SELECT COUNT(*) FROM pg_stat_activity;
SELECT pid, usename, application_name, backend_start, state_change, state FROM pg_stat_activity where state='idle';
SELECT pg_terminate_backend(PID);
ALTER system SET max_connections = 1000;

SET SESSION idle_in_transaction_session_timeout = '5min';
ALTER DATABASE SET idle_in_transaction_session_timeout = '5min'
```

#### Creating new database with specified collation

If we're creating new database with specific collation, sometimes it will return error like this:

`ERROR:  new collation (C.UTF-8) is incompatible with the collation of the template    database (en_US.UTF8)`

If this happens, we should specify template0 as the template by specifying `-T template0`

See: [here](https://stackoverflow.com/questions/18870775/how-to-change-the-template-database-collection-coding-on-postgresql)

#### Useful queries for monitoring

See: [here](https://github.com/nilenso/postgresql-monitoring)

#### Autovacuum

Autovacuum should be turned off for high volume DB in postgres (by default it's on).

> Autovacuum works well when configured correctly. However its default configuration is appropriate for databases of a few hundred megabytes in size, and is not aggressive enough for larger databases. In production environments it starts to fall behind. 

References:

- https://www.citusdata.com/blog/2016/11/04/autovacuum-not-the-enemy/

#### References

- [Things I Wished More Developers Knew About Databases](https://rakyll.medium.com/things-i-wished-more-developers-knew-about-databases-2d0178464f78)
- [Useful Queries for PostgreSQL Index Maintenance](https://www.percona.com/blog/2020/03/31/useful-queries-for-postgresql-index-maintenance/)
- [PostgreSQL Monitoring for Application Developers: The DBA Fundamentals](https://blog.crunchydata.com/blog/postgresql-monitoring-for-application-developers-dba-stats)
