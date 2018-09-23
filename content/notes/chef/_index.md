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

#### Debugging Chef Solo

The stacktrace can be seen at:

`~/chef-solo/local-mode-cache/cache/chef-stacktrace.out`


#### Uploading Cookbook

1. Don't forget to bump the version if cookbook is updated
2. Put cookbook that you want to upload in cookbooks directory
3. `knife cookbook upload <cookbook-name>`

#### Editing Attributes on Chef Server

```shell
knife node show -Fj p-node-01 > p-node-01.json
# Edit the p-node-01.json file
knife node from file p-node-01.json
```

#### Knife search

```
# Search nodes by tag
knife search -i "tags:<tag>"

# Search nodes by runlist
knife search -i "run_list:<run_list>"

# Show a node attribute in json
knife node show <node-name> -F json

# Show a node attribute in json and fetch the tags using jq
knife node show <node-name> -F json  | jq '.normal.tags'
```
