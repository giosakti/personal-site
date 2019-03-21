---
title: "Architecture"
authors: ["gio"]
---

#### To CQRS or not?

- CQRS is useful when separation of read and write operation is necessary, for example when long write operations can have impacts to read operations. But it does have a cost of eventual consistency.
- Not all system can allow eventual consistency, but sometimes the benefits outweight the drawbacks.
- Talking about pattern without context is difficult because patterns are applied on a per-case basis
