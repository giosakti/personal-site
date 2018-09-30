---
title: "GCP"
authors: ["gio"]
---

## SSL Certificate

```
gcloud compute ssl-certificates create <name> \
    --certificate certificate.crt \
    --private-key private_key.pem
```

## Load Balancer

- https://github.com/GoogleCloudPlatform/terraform-google-lb-http
- https://cloud.google.com/community/tutorials/modular-load-balancing-with-terraform
