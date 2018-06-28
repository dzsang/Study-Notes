# 先记录二叉树ORAM的一点东西
1. 数据块是始终和叶节点绑在一起的，无论在路径中的哪个位置，都是要靠叶子节点的编号去搜索然后在整个路径中查找的
2. 数据存放在路径中的某个桶，随机选择2个节点向下驱逐

# onion ORAM
1. 这里解释了bandwidth blowup是bandwidth measured in the number of blocks (i.e., blowup
compared to a normal RAM).  
他们利用服务端的计算来减少client-server的bandwidth
2. 参考的一个思想：client不要“手动地”读写数据来移动server端的数据，而是利用同态加密属性去指示server怎么做。
3. 不管针对半诚实场景，也要针对恶意场景
4. the client "guides" the server to perform ORAM accesses and evictions homomor-phically by sending the server some "helper values".
5. evict过程试讲整个桶完全驱逐，S3ORAM借鉴了它的驱逐策略，依据逆辞典序选择驱逐路径
1. onion ORAM补完
> 还是有个问题，在这种新的驱逐过程中，非叶节点是否会存储数据，因为  
After an eviction,
the buckets on the eviction path (excluding the leaves) are guaranteed to be empty. Further, at the
start of each eviction, each sibling bucket for that eviction is guaranteed to be empty.