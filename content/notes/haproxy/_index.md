---
title: "HAProxy"
authors: ["gio"]
---

#### Log HTTP response body in HAProxy (for debugging)

Put this in haproxy configuration.

```
option http-buffer-request
declare capture request len 400
http-request capture req.body id 0
log-format {"%[capture.req.hdr(0)]"}
```
