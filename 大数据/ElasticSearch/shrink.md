# Index shrink

index shrink 是将 index 的所有 shard 数据移动到某个 node 的过程, 在此过程中可以制动新的 shard 数据, 从而达到减少 shard 的目的。

shrink 非常适合于时序数据(开始时指定多个 shard 以提高写性能,但是当写入完成后数据就变成只读的了),后续通过 shrink 减少副本数量可以节省存储空间。步骤如下：

+ 先把所有主分片都转移到一台主机上;
+ 在这台主机上创建一个索引,分片数较小,其他设置和原索引一致;
+ 把转移到的原索引的所有分片,复制(或硬链接)到新索引的目录下;
+ 对新索引进行打开操作恢复分片数据;
+ (可选)重新把索引的分片均衡到其他节点上。

具体步骤如下:

1. 设置待 shrink 的index移动到node完成名称

    因为这个操作流程需要把所有分片都转移到一台主机上，所以作为 shrink 主机，它的磁盘要足够大，至少要能放得下一整个索引。最好是一整块磁盘，因为硬链接是不能跨磁盘的，靠复制太慢了。

    PUT /my_source_index/_settings
    {
        "settings": {
            "index.routing.allocation.require._name": "shrink_node_name",
            "index.blocks.write": true
        }
    }

    等待，直到 index 的各 shard 数据移动到指定的节点。

1. 将 index 设置为只读

    PUT /my_source_index/_settings
    {
        "settings": {
            "index.blocks.write": true
        }
    }

    在 shrink 操作前，需要阻塞原 index 的写操作。

1. 开始 _shrink 操作，可以指定新的 index 的配置，如 shard 和 replicas 数量：

    POST my_source_index/_shrink/my_target_index
    {
        "settings": {
            "index.number_of_replicas": 1,
            "index.number_of_shards": 1, 
            "index.codec": "best_compression" 
        },
        "aliases": {
            "my_search_indices": {}
        }
    }

    该命令立即返回，但是后台继续操作，理想情况下很快完成(因为只是建立硬链接操作)。Elasticsearch 会一直等到 shrink 操作完成的时候，才会真的开始做 replica 分片的分配和重均衡，此前分片都处于 initializing 状态

注意：

1. shrink 操作前需要移动 index 的所有数据到一个节点上，可能会对集群和目标节点产生负载，所以最好串行执行；
1. 待 shrink 的 index，如果有其它的 index.routing.allocation 策略，则不能与 index.routing.allocation.require._name 指定的 node 冲突。
   常见的问题是，index 过去指定了 index.routing.allocation.require.role 为 hot，而新指定的 shrink_node_name node 的 role 属性非 hot 如 warm，两者冲突，这样就不会进行数据的移动。
1. shrink 前，老的 index 必须阻塞写操作，否则会 shrink 失败；
1. 可以过滤 GET _nodes/hot_threads 响应的中的 shrink 关键字来查看 shrink 的进度；

参考：  
http://www.cnblogs.com/bonelee/p/8136708.html  
https://www.elasticco/blog/resizing-elasticsearch-shards-for-fun-and-profit