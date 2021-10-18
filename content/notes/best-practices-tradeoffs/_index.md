---
title: "Best Practices / Trade-offs"
authors: ["gio"]
---

> This note contains things that are "easy to do in the beginning" but "hard to do later" that have substantial benefits.

#### Restricting Deletion (or to some extent, Change) of existing records

- It's generally a good idea to restrict physical deletion in database, which means we should have `deleted_at` column instead.
- At some point it may also be worth it to consider "append-only" model where we restrict mutation of existing data. This may help us if we want to do horizontal-scaling (with the drawbacks being harder to get current state of the data, esp. for complex query). Tread wisely.

#### Setting-up Multitenancy

- Setting-up multitenancy is relatively easier to do if we do it from the very beginning, which is by introducing an `app_id` column in all relational tables without exception.
- This effort will have at least 2 benefits:
  - If later down the line we want to make our app multi-tenant.
  - If we want to reuse same DB for different environment.
