---
title: "ELK"
authors: ["gio"]
---

#### Useful APIs

```
curl -XGET http://localhost:9200/_cluster/health?pretty=true
curl -XGET http://localhost:9200/_cat/indices?v
curl -XGET http://localhost:9200/_cat/shards
curl -XGET http://localhost:9200/_cat/shards?h=index,shard,prirep,state,unassigned.reason
```

#### Fixing: Unassigned Shards

```
# Fetch all shards
curl -XGET http://localhost:9200/_cat/shards?h=index,shard,prirep,state,unassigned.reason

# Disable allocation
curl -XPUT 'localhost:9200/_settings' \
    -d '{"index.routing.allocation.disable_allocation": false}'
```

Also see [here](https://stackoverflow.com/questions/19967472/elasticsearch-unassigned-shards-how-to-fix)

#### Authorization

1. https://www.elastic.co/guide/en/x-pack/current/authorization.html
2. https://discuss.elastic.co/t/kibana-default-basic-auth/86045

#### Optimizing fluentd

There are some optimization configs that we can specify for fluentd. For example:

```
<buffer>
  flush_thread_count 8
  retry_timeout 1h
  retry_max_times 10
  overflow_action drop_oldest_chunk
</buffer>
```

See also:

- https://github.com/fluent/fluentd/issues/1218
- https://docs.fluentd.org/v1.0/articles/performance-tuning-single-process
- https://docs.fluentd.org/v1.0/articles/buffer-section
