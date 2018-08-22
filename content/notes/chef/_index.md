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
