---
title: "PostgreSQL"
authors: ["gio"]
---

#### Creating new database with specified collation

If we're creating new database with specific collation, sometimes it will return error like this:

`ERROR:  new collation (C.UTF-8) is incompatible with the collation of the template    database (en_US.UTF8)`

If this happens, we should specify template0 as the template by specifying `-T template0`

See: [here](https://stackoverflow.com/questions/18870775/how-to-change-the-template-database-collection-coding-on-postgresql)
