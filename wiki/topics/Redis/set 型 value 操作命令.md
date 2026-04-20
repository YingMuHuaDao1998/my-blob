# set 型 value 操作命令

![image-20240103100321145](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240103100321145.png)



1. sadd

- 格式：sadd key member [members]

- 功能：讲一个或多个 member 元素加入到集合 key 中，已经存在与集合中的 member 会被忽略
- 说明：![image-20240103100748500](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240103100748500.png)



![image-20240103101203582](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240103101203582.png)



2. smembers

- 格式：members key
- 功能：返回集合 key 中所有的元素
- ![image-20240103101423361](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240103101423361.png)

![image-20240103101439481](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240103101439481.png)



3. scard

- 格式：scard key

- 功能：返回 set 集合的长度

- 说明：当 key 不存在时，返回 0

  ![image-20240103101723331](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240103101723331.png)



4. sismember

- 格式：sismember  key member

- 功能：判断元素是否在集合 key 内

- 说明：在 1，不在 0

  ![image-20240103102610946](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240103102610946.png)



5. smove

- 格式：smove oldkey newkey member

- 功能：将旧集合中的元素移动到新集合当中

- 说明：![image-20240103103533953](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240103103533953.png)

  ![image-20240103103327294](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240103103327294.png)



6. srem

- 格式：srem key member [members]

- 功能：将集合中的元素移除

  ![image-20240103103954830](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240103103954830.png)



7. sdiff

- 格式：sdiff key1 key2

- 功能：查看两个集合的差集

  ![image-20240103105534152](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240103105534152.png)

sdiff 命令查看差集为临时结果,x 和 y 集合元素并没有发生变化，如果想保存差集，应该使用 sdiffstore命令

![image-20240103105633643](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240103105633643.png)

sdiffstore计算集合差集并将结果保存为新的集合

