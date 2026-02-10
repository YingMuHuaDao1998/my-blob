# BitMap操作

（1）**BitMap** **简介**

BitMap 是 Redis 2.2.0 版本中引入的一种新的数据类型。该数据类型本质上就是一个仅

包含 0 和 1 的二进制字符串。而其所有相关命令都是对这个字符串二进制位的操作。用于描

述该字符串的属性有三个：key、offset、bitValue。 

- key：BitMap 是 Redis 的 key-value 中的一种 Value 的数据类型，所以该 Value 一定有其对

应的 key。 

- offset：每个 BitMap 数据都是一个字符串，字符串中的每个字符都有其对应的索引，该

索引从 0 开始计数。该索引就称为每个字符在该 BitMap 中的偏移量 offset。这个 offset

的值的范围是[0，2 32 -1]，即该 offset 的最大值为 4G-1，即 4294967295，42 亿多。 

-  bitValue：每个 BitMap 数据中都是一个仅包含 0 和 1 的二进制字符串，每个 offset 位上

的字符就称为该位的值 bitValue。bitValue 的值非 0 即 1。



（2）setbit

- 格式：SETBIT key offset value

- 功能：为给定 key 的BitMap 数据的 offset 位置设置值为 value。其返回值为修改前该 offset

位置的 bitValue

- 说明：对于原 BitMap 字符串中不存在的 offset 进行赋值，字符串会自动伸展以确保它

可以将 value 保存在指定的 offset 上。当字符串值进行伸展时，空白位置以 0 填充。当

然，设置的 value 只能是 0 或 1。不过需要注意的是，对使用较大 offset 的 SETBIT 操作

来说，内存分配过程可能造成 Redis 服务器被阻塞。

![image-20240111155549825](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240111155549825.png)



（3）gitbit

- 格式：GETBIT key offset

- 功能：对 key 所储存的 BitMap 字符串值，获取指定 offset 偏移量上的位值 bitValue。

- 说明：当 offset 比字符串值的长度大，或者 key 不存在时，返回 0 。

![image-20240111155657487](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240111155657487.png)



（4）bitcount

- 格式：BITCOUNT key [start] [end]

- 功能：统计给定字符串中被设置为 1 的 bit 位的数量。一般情况下，统计的范围是给定

的整个 BitMap 字符串。但也可以通过指定额外的 start 或 end 参数，实现仅对指定字节

范围内字符串进行统计，包括 start 和 end 在内。注意，这里的 start 与 end 的单位是

字节，不是 bit，并且从 0 开始计数。

- 说明：start 和 end 参数都可以使用负数值： -1 表示最后一个字节， -2 表示倒数第二个

字节，以此类推。另外，对于不存在的 key 被当成是空字符串来处理，因此对一个不存

在的 key 进行 BITCOUNT 操作，结果为 0 。

![image-20240111155836211](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240111155836211.png)



（5）bitpos

- 格式：BITPOS key bit [start] [end]

- 功能：返回 key 指定的 BitMap 中第一个值为指定值 bit(非 0 即 1) 的二进制位的位置。

pos，即 position，位置。在默认情况下， 命令将检测整个 BitMap，但用户也可以通过

可选的 start 参数和 end 参数指定要检测的范围。

- 说明：start 与 end 的意义与 bitcount 命令中的相同

![image-20240111160244241](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240111160244241.png)



（6）bitop

- 格式：BITOP operation destkey key *key …+

- 功能：对一个或多个 BitMap 字符串 key 进行二进制位操作，并将结果保存到 destkey 上。

- operation 可以是 AND 、 OR 、 NOT 、 XOR 这四种操作中的任意一种：

-  BITOP AND destkey key [key ...] ：对一个或多个 BitMap 执行按位与操作，并将结果

保存到 destkey 。 

- BITOP OR destkey key [key ...] ：对一个或多个 BitMap 执行按位或操作，并将结果保

存到 destkey 。 

- BITOP XOR destkey key [key ...] ：对一个或多个 BitMap 执行按位异或操作，并将结

果保存到 destkey 。 

-  BITOP NOT destkey key ：对给定 BitMap 执行按位非操作，并将结果保存到 destkey 。 

说明：

- 除了 NOT 操作之外，其他操作都可以接受一个或多个 BitMap 作为输入。

- 除了 NOT 操作外，其他对一个 BitMap 的操作其实就是一个复制。

- 如果参与运算的多个 BitMap 长度不同，较短的 BitMap 会以 0 作为补充位与较长www.bjpowernode.com **75** / **248** Copyright© 动力节点

BitMap 运算，且运算结果长度与较长 BitMap 的相同。

![image-20240111170126829](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240111170126829.png)

![image-20240111170228756](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240111170228756.png)

- 应用场景

  由于 offset 的取值范围很大，所以其一般应用于大数据量的二值性统计。例如平台活跃

  用户统计（二值：访问或未访问）、支持率统计（二值：支持或不支持）、员工考勤统计（二

  值：上班或未上班）、图像二值化（二值：黑或白）等。

  不过，对于数据量较小的二值性统计并不适合 BitMap，可能使用 Set 更为合适。当然，

  具体多少数据量适合使用 Set，超过多少数据量适合使用 BitMap，这需要根据具体场景进行

  具体分析。

  例如，一个平台要统计日活跃用户数量。

  如果使用 Set 来统计，只需上线一个用户，就将其用户 ID 写入 Set 集合即可，最后只需

  统计出 Set 集合中的元素个数即可完成统计。即 Set 集合占用内存的大小与上线用户数量成

  正比。假设用户 ID 为 m 位 bit 位，当前活跃用户数量为 n，则该 Set 集合的大小最少应该是

  m*n 字节。

  如果使用 BitMap 来统计，则需要先定义出一个 BitMap，其占有的 bit 位至少为注册用

  户数量。只需上线一个用户，就立即使其中一个 bit 位置 1，最后只需统计出 BitMap 中 1 的

  个数即可完成统计。即 BitMap 占用内存的大小与注册用户数量成正比，与上线用户数量无

  关。假设平台具有注册用户数量为 N，则 BitMap 的长度至少为 N 个 bit 位，即 N/8 字节。

  何时使用 BitMap 更合适？令 m*n 字节 = N/8 字节，即 n = N/8/m = N/(8*m) 时，使用

  Set 集合与使用 BitMap 所占内存大小相同。以淘宝为例，其用户 ID 长度为 11 位(m)，其注

  册用户数量为 8 亿(N)，当活跃用户数量为 8 亿/(8*11) = 0.09 亿 = 9*106 = 900 万，使用 Set

  与 BitMap 占用的内存是相等的。但淘宝的日均活跃用户数量为 8 千万，所以淘宝使用 BitMap

  更合适。

