# zset 有序 set 型 value 操作命令

1. zadd

- ZADD key score member [[score member] [score member] ...]

- 功能：将一个或多个 member 元素及其 score 值加入到有序集 key 中的适当位置。

- 说明：score 值可以是整数值或双精度浮点数。如果 key 不存在，则创建一个空的有序

  集并执行 ZADD 操作。当 key 存在但不是有序集类型时，返回一个错误。如果命令执

  行成功，则返回被成功添加的新成员的数量，不包括那些被更新的、已经存在的成员。

  若写入的 member 值已经存在，但 score 值不同，则新的 score 值将覆盖老 score。

![image-20240103152216229](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240103152216229.png)



2. **zrange** **与** **zrevrange**

- 格式：ZRANGE key start stop [WITHSCORES] 或 ZREVRANGE key start stop [WITHSCORES]
-  功能：返回有序集 key 中，指定区间内的成员。zrange 命令会按 score 值递增排序，zrevrange命令会按score递减排序。具有相同 score 值的成员按字典序/逆字典序排列。可以通过使用 WITHSCORES 选项，来让成员和它的 score 值一并返回
-  说明：下标参数从 0 开始，即 0 表示有序集第一个成员，以 1 表示有序集第二个成员，以此类推。也可以使用负数下标，-1 表示最后一个成员，-2 表示倒数第二个成员，以此类推。超出范围的下标并不会引起错误。例如，当 start 的值比有序集的最大下标还要大，或是 start > stop 时，ZRANGE 命令只是简单地返回一个空列表。再比如 stop 参数的值比有序集的最大下标还要大，那么 Redis 将 stop 当作最大下标来处理。若 key 中指定范围内包含大量元素，则该命令可能会阻塞 Redis 服务。所以生产环境中如果要查询有序集合中的所有元素，一般不使用该命令，而使用 zscan 命令代替

![image-20240103152727262](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240103152727262.png)