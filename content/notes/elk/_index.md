---
title: "ELK"
authors: ["gio"]
---

#### Useful APIs

```
# Cluster health
curl -XGET http://localhost:9200/_cluster/health?pretty=true

# Indices operation
curl -XGET http://localhost:9200/_cat/indices?v
curl -XGET http://localhost:9200/_cat/indices?h=h,s,i,id,p,r,dc,dd,ss,creation.date.string&s=creation.date:desc
# h=health
# s=status
# i=index
# id=uuid
# p=pri
# r=rep
# dc=docs.count
# dd=docs.deleted
# ss=store.size

curl -XDELETE http://localhost:9200/<<index_name>>,<<index_name>>

# Shards operation
curl -XGET http://localhost:9200/_cat/shards
curl -XGET http://localhost:9200/_cat/shards?h=index,shard,prirep,state,unassigned.reason
```

#### Fix: Unassigned Shards

```
# Fetch all shards
curl -XGET http://localhost:9200/_cat/shards?h=index,shard,prirep,state,unassigned.reason

# Disable allocation
curl -XPUT 'localhost:9200/_settings' \
    -d '{"index.routing.allocation.disable_allocation": false}'
```

Also see [here](https://stackoverflow.com/questions/19967472/elasticsearch-unassigned-shards-how-to-fix)

#### Disable read-only mode

If for some reasons your elasticsearch node runs out of space, it will automatically lock all indices by enabling read-only mode. After you provide more spaces to elasticsearch node, you have to manually disable read-only mode by doing this to each indices:

```
curl -X PUT -H "Content-Type: application/json" -d '{"index": {"blocks": {"read_only_allow_delete": "false"}}}' "localhost:9200/<INDEX NAME>/_settings"
```

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
