---
title: "Terraform"
authors: ["gio"]
---

#### "Error: Provider configuration not present"

This might happen if we tried to remove an existing resource/module that do not have `required_providers` block already in place.

To solve this, we need to:

1. Revert the removal of the existing resource/module
2. In `version.tf` ensure that we have proper mapping of required providers, for example if we use Google:

```
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 3.90.1"
    }
  }
  required_version = ">= 0.13"
}

```
3. Plan and then apply, even if there's no changes
4. Try removing the resource/module again and then plan & apply

Reference: https://support.hashicorp.com/hc/en-us/articles/1500000332721-Error-Provider-configuration-not-present