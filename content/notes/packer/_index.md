---
title: "Packer"
authors: ["gio"]
---

## Building Image

1. Install packer, see http://packer.io/intro/getting-started/install.html
2. Write `packer.json` check builders & provisioners reference, example:
  - https://www.packer.io/docs/builders/lxd.html
  - http://packer.io/intro/getting-started/provision.html
  - https://www.packer.io/docs/provisioners/chef-solo.html
3. `packer validate packer.json`
4. `packer build packer.json`
