# **Geospatial** **简介**

Geospatial，地理空间。

Redis 在 3.2 版本中引入了 Geospatial 这种新的数据类型。该类型本质上仍是一种集合，只不过集合元素比较特殊，是一种由三部分构成的数据结构，这种数据结构称为空间元素： 

-  经度：longitude。有效经度为[-180，180]。正的表示东经，负的表示西经。

- 纬度：latitude。有效纬度为[-85.05112878，85.05112878]。正的表示北纬，负的表示南纬。

- 位置名称：为该经纬度所标注的位置所命名的名称，也称为该 Geospatial 集合的空间元素名称。通过该类型可以设置、查询某地理位置的经纬度，查询某范围内的空间元素，计算两空间元素间的距离等。



（1）**geoadd**

- 格式：GEOADD key longitude latitude member  [longitude latitude member …]

-  功能：将一到多个空间元素添加到指定的空间集合中。

- 说明：当用户尝试输入一个超出范围的经度或者纬度时，该命令会返回一个错误。

<img src="/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240112100147670.png" alt="image-20240112100147670" style="zoom:80%;" />



(2)  **geopos**

- 格式：GEOPOS key member [member …]

- 功能：从指定的地理空间中返回指定元素的位置，即经纬度。

- 说明：因为 该命令接受可变数量元素作为输入，所以即使用户只给定了一个元素，命令也会返回数组。

![image-20240112100606219](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240112100606219.png)



(3) **geodist**

- 格式：GEODIST key member1 member2 [unit]

- 功能：返回两个给定位置之间的距离。其中 unit 必须是以下单位中的一种：

- m ：米，默认
- km ：千米
- mi ：英里
- ft：英尺
- 说明：如果两个位置之间的其中一个不存在， 那么命令返回空值。另外，在计算距离时会假设地球为完美的球形， 在极限情况下， 这一假设最大会造成 0.5% 的误差。

![image-20240112101436055](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240112101436055.png)



(4) **geohash**

- 格式：GEOHASH key member *member …+

- 功能：返回一个或多个位置元素的 Geohash 值。

- 说明：GeoHash 是一种地址编码方法。他能够把二维的空间经纬度数据编码成一个字符串。该值主要用于底层应用或者调试， 实际中的作用并不大。

![image-20240112101834651](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240112101834651.png)



(5) **georadius**

- 格式：GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [ASC|DESC] [COUNT count]

- 功能：以给定的经纬度为中心，返回指定地理空间中包含的所有位置元素中，与中心距离不超过给定半径的元素。返回时还可携带额外的信息：

WITHDIST ：在返回位置元素的同时，将位置元素与中心之间的距离也一并返回。距离的单位和用户给定的范围单位保持一致。

WITHCOORD ：将位置元素的经维度也一并返回。 WITHHASH：将位置元素的 Geohash 也一并返回，不过这个 hash 以整数形式表示

命令默认返回未排序的位置元素。 通过以下两个参数，用户可以指定被返回位置元素的排序方式：

ASC ：根据中心的位置，按照从近到远的方式返回位置元素。

DESC ：根据中心的位置，按照从远到近的方式返回位置元素。

- 说明：在默认情况下， 该命令会返回所有匹配的位置元素。虽然用户可以使用 COUNT <count> 选项去获取前 N 个匹配元素，但因为命令在内部可能会需要对所有被匹配的元素进行处理，所以在对一个非常大的区域进行搜索时，即使使用 COUNT 选项去获取少量元素，该命令的执行速度也可能会非常慢。

![image-20240112103843829](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240112103843829.png)

![image-20240112103806820](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240112103806820.png)



(6) **georadiusbymember**

- 格式：GEORADIUSBYMEMBER key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [ASC|DESC] [COUNT count]

- 功能：这个命令和 GEORADIUS 命令一样，都可以找出位于指定范围内的元素，但该命令的中心点是由位置元素形式给定的，而不是像 GEORADIUS 那样，使用输入的经纬度来指定中心点。

- 说明：返回结果中也是包含中心点位置元素的

![image-20240112104456010](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240112104456010.png)

查看北京周边 6000km 的城市，参数为位置元素，不是手动输入的经纬度



## 应用场景

Geospatial 的意义是地理位置，所以其主要应用地理位置相关的计算。例如，微信发现中的“附近”功能，添加朋友中“雷达加朋友”功能；QQ 动态中的“附近”功能；钉钉中的“签到”功能等。