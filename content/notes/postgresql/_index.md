---
title: "PostgreSQL"
authors: ["gio"]
---

#### Common Commands

```
# Create new user
CREATE ROLE pathfinder_mono;
ALTER ROLE pathfinder_mono WITH LOGIN;
ALTER ROLE pathfinder_mono WITH SUPERUSER;
ALTER ROLE pathfinder_mono WITH PASSWORD 'pathfinder_mono';

# Show/modify max connections
SHOW max_connections;
SELECT COUNT(*) FROM pg_stat_activity;
SELECT pid, usename, application_name, backend_start, state_change, state FROM pg_stat_activity where state='idle';
SELECT pg_terminate_backend(PID);
ALTER system SET max_connections = 1000;

# Find slow queries
SELECT
  pid,
  user,
  pg_stat_activity.query_start,
  now() - pg_stat_activity.query_start AS query_time,
  query,
  state,
  wait_event_type,
  wait_event
FROM pg_stat_activity
WHERE (now() - pg_stat_activity.query_start) > interval '5 minutes';

# Alter idle in transaction session timeout
SET SESSION idle_in_transaction_session_timeout = '5min';
ALTER DATABASE SET idle_in_transaction_session_timeout = '5min'
```

#### Useful queries for monitoring

- [PostgreSQL Monitoring for Application Developers: The DBA Fundamentals](https://blog.crunchydata.com/blog/postgresql-monitoring-for-application-developers-dba-stats)
- [nilenso/postgresql-monitoring](https://github.com/nilenso/postgresql-monitoring)

#### Tuning

- [Tuning Your PostgreSQL Server](https://wiki.postgresql.org/wiki/Tuning_Your_PostgreSQL_Server)
- [How to Tune PostgreSQL Memory](https://www.enterprisedb.com/postgres-tutorials/how-tune-postgresql-memory)

#### Indexing

- [Indexing in Postgres](https://medium.com/geekculture/indexing-in-postgres-db-4cf502ce1b4e)
- [GIN Index](https://pganalyze.com/blog/gin-index)
- [How to Create pg_trgm compound indexes](https://dba.stackexchange.com/questions/196053/how-to-create-pg-trgm-compound-indexes-with-date-columns)

#### Partitioning

- [How to divide single partition into two different partitions in PostgreSQL](https://stackoverflow.com/questions/63529097/how-to-divide-single-partition-into-two-different-partitions-in-postgresql-and-t)

#### Autovacuum

Autovacuum should be turned off for high volume DB in postgres (by default it's on).

> Autovacuum works well when configured correctly. However its default configuration is appropriate for databases of a few hundred megabytes in size, and is not aggressive enough for larger databases. In production environments it starts to fall behind. 

References:  
- https://www.citusdata.com/blog/2016/11/04/autovacuum-not-the-enemy/

#### References

- [Things I Wished More Developers Knew About Databases](https://rakyll.medium.com/things-i-wished-more-developers-knew-about-databases-2d0178464f78)
- [Useful Queries for PostgreSQL Index Maintenance](https://www.percona.com/blog/2020/03/31/useful-queries-for-postgresql-index-maintenance/)
