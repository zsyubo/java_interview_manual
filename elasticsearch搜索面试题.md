> ps:我个人es使用经验不足，大家可以看这篇文章：
> https://juejin.im/post/5d53f6be6fb9a06ac76daa1f   ElasticSearch（提高篇）
> https://mp.weixin.qq.com/s/oX_Urq2ge2pSf0w0I5maTw


# `elasticsearch了解多少，说说你们公司es的集群架构，索引数据大小，分片有多少，以及一些调优手段`
todo

# `elasticsearch的倒排索引是什么`
倒排索引，相反于一篇文章包含了哪些词，它从词出发，记载了这个词在哪些文档中出现过，由两部分组成——词典和倒排表。

# `elasticsearch 索引数据多了怎么办，如何调优，部署`
todo

# `elasticsearch是如何实现master选举的`
1. 第一步：确认候选主节点数达标，elasticsearch.yml设置的值discovery.zen.minimum_master_nodes；
2. 第二步：比较：先判定是否具备master资格，具备候选主节点资格的优先返回；若两节点都为候选主节点，则id小的值会主节点。注意这里的id为string类型。

题外话：获取节点id的方法。
```
GET /_cat/nodes?v&h=ip,port,heapPercent,heapMax,id,name
ip        port heapPercent heapMax id   name
127.0.0.1 9300          39   1.9gb Hk9w Hk9wFwU
```

# `详细描述一下Elasticsearch索引文档的过程`
![avatar](https://s2.ax1x.com/2019/10/15/KpHqWq.png)
单文档写入过程大概：
1. 第一步：客户写集群某节点写入数据，发送请求。（如果没有指定路由/协调节点，请求的节点扮演路由节点的角色。）
2. 第二步：节点1接受到请求后，使用文档_id来确定文档属于分片0。请求会被转到另外的节点，假定节点3。因此分片0的主分片分配到节点3上。
3. 第三步：节点3在主分片上执行写操作，如果成功，则将请求并行转发到节点1和节点2的副本分片上，等待结果返回。所有的副本分片都报告成功，节点3将向协调节点（节点1）报告成功，节点1向请求客户端报告写入成功。

如果面试官再问：第二步中的文档获取分片的过程？
回答：借助路由算法获取，路由算法就是根据路由和文档id计算目标的分片id的过程。
```
1shard = hash(_routing) % (num_of_primary_shards)
```

# `详细描述一下Elasticsearch搜索的过程？`
搜索拆解为“query then fetch” 两个阶段。
query阶段的目的：定位到位置，但不取。
步骤拆解如下：
- 1、假设一个索引数据有5主+1副本 共10分片，一次请求会命中（主或者副本分片中）的一个。
- 2、每个分片在本地进行查询，结果返回到本地有序的优先队列中。
- 3、第2）步骤的结果发送到协调节点，协调节点产生一个全局的排序列表。

fetch阶段的目的：取数据。
路由节点获取所有文档，返回给客户端。

# `Elasticsearch在部署时，对Linux的设置有哪些优化方法`

# `lucence内部结构是什么？`

# `深分页问题`