# 简介

HyperLogLog 是 Redis 2.8.9 版本中引入的一种新的数据类型，其意义是 hyperlog log，超级日志记录。该数据类型可以简单理解为一个 set 集合，集合元素为字符串。但实际上HyperLogLog 是一种基数计数概率算法，通过该算法可以利用极小的内存完成独立总数的统计。其所有相关命令都是对这个“set 集合”的操作。HyperLogLog 算法是由法国人 Philippe Flajolet 博士研究出来的，Redis的作者 Antirez 为了纪念 Philippe Flajolet 博士对组合数学和基数计算算法分析的研究，在设计 HyperLogLog 命令的时候使用了 Philippe Flajolet姓名的英文首字母 PF 作为前缀。遗憾的是 Philippe Flajolet 博士于 2011年 3 月 22 日因病在巴黎辞世。

HyperLogLog 算法是一个纯数学算法，我们这里不做研究。HyperLogLog 是 Redis 2.8.9 版本中引入的一种新的数据类型，其意义是 hyperlog log，超级日志记录。该数据类型可以简单理解为一个 set 集合，集合元素为字符串。但实际上HyperLogLog 是一种基数计数概率算法，通过该算法可以利用极小的内存完成独立总数的统计。其所有相关命令都是对这个“set 集合”的操作。HyperLogLog 算法是由法国人 Philippe Flajolet 博士研究出来的，Redis的作者 Antirez 为了纪念 Philippe Flajolet 博士对组合数学和基数计算算法分析的研究，在设HyperLogLog 命令的时候使用了 Philippe Flajolet姓名的英文首字母 PF 作为前缀。遗憾的是 Philippe Flajolet 博士于 2011年 3 月 22 日因病在巴黎辞世。HyperLogLog 算法是一个纯数学算法，我们这里不做研究。



(1)pfadd

- 格式：PFADD key element [element …]

- 功能：将任意数量的元素添加到指定的 HyperLogLog 集合里面。如果内部存储被修改了返回 1，否则返回 0。

![image-20240112092927600](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240112092927600.png)



(2)pfcount

- 格式：PFCOUNT key *key …+

- 功能：该命令作用于单个 key 时，返回给定 key 的 HyperLogLog 集合的近似基数；该命

令作用于多个 key 时，返回所有给定 key 的 HyperLogLog 集合的并集的近似基数；如果

key 不存在，则返回 0。

![image-20240112093227349](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240112093227349.png)



(3)pfmerge

- 格式：PFMERGE destkey sourcekey [sourcekey …]

- 功能：将多个 HyperLogLog 集合合并为一个 HyperLogLog 集合，并存储到 destkey 中，

合并后的 HyperLogLog 的基数接近于所有 sourcekey 的 HyperLogLog 集合的并集。





