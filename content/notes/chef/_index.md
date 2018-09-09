---
title: "Chef"
authors: ["gio"]
---

#### Logging

To log something out when executing chef, we can write this in our recipe:

```ruby
log 'testing' do
  message 'any message'
  level :info
end
```

#### Uploading Cookbook

1. put cookbook that you want to upload in cookbooks directory
2. `knife cookbook upload <cookbook-name>`

#### Editing Attributes on Chef Server

```shell
knife node show -Fj p-node-01 > p-node-01.json
# Edit the p-node-01.json file
knife node from file p-node-01.json
```
