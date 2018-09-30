---
title: "Chef"
authors: ["gio"]
---

#### Logging

Writing this on your recipe will result in log message appearing during the provider execution phase.

```ruby
log 'test' do
  level :info
end
```

While this will result in the message appearing during resource gathering stage of the chef run.

```ruby
Chef::Log.info("Connection to database '#{dbname}' on '#{host}' failed")
```

You can set the log level of your chef run by setting the -l flag.

```sh
chef-client -l info
```

#### Debugging Chef Solo

The stacktrace can be seen at:

`~/chef-solo/local-mode-cache/cache/chef-stacktrace.out`

#### Modify cookbook on the node directly and apply it (useful when testing)

Modify cookbook(s) on `/var/chef/cache/cookbooks`.

Then run chef using `--skip-cookbook-sync` parameter.

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

#### Useful knife commands

```
# Search nodes by tag
knife search -i "tags:<tag>"

# Search nodes by runlist
knife search -i "run_list:<run_list>"

# Search nodes by runlist (with wildcard)
knife search -i "run_list:recipe\\[foo\\:\\:*\\]"

# Show a node attribute in json
knife node show <node-name> -F json

# Show a node attribute in json and fetch the tags using jq
knife node show <node-name> -F json  | jq '.normal.tags'

# Get a specific node attribute (e.g. ipaddress)
knife node show <node-name> -a ipaddress -Fj | jq '.[].ipaddress'

# Add runlist to a node
knife node run_list add <node-name> "recipe[foo]"

# SSH to a node and execute a command
knife ssh "name:<node-name>" 'sudo chef-client -o "recipe[foo]"' -x ubuntu -a ipaddress -i <private-key>
```

#### Running multiple specific recipes on a node

```
chef-client -o "recipe[recipe1],recipe[recipe2]"
```

#### Do something based on current node runlist

```ruby
run_list = node.primary_runlist
if run_list.includes?('role[foo]')
  do_something
else
  do_something_else
end
```
