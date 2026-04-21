# noSql非关系数据库

本机 redis 配置文件目录：/opt/homebrew/etc/redis.conf

(一) 键值存储数据库

像 map一样的 key-value 对，典型代表为 redis

(二) 列存储数据库

关系型数据库是典型的行存储数据库，其存在问题是，按行存储的数据在物理层面占用的是连续存储空间，不适合海量数据存储，而按列存储则可实现分布式存储，适合海量存储，典型的是HBase.

(三) 文档型数据库

典型代表MongoDB

其是NoSQL与关系型数据库的结合

(四)图形(Graph)数据库

用于存放一个节点关系的数据库，例如描述不同人间的关系，典型代表Neo-4j



缓存数据分类：

实时同步数据：要求缓存中的数据和 db 种数据保持同步，只要 db 中数据更改，缓存中就清除数据，等下次写入db 种新的数据。

阶段性同步数据：其没必要于 db 中数据实时同步，只要相差不多就可以，给缓存数据添加生存时长属性。

1. 例子

   某商户网站首页展示网站使用人数，以及站内消费等信息，不必实时刷新缓存，实时数据并不影响网站的运营。

![image-20231228171052900](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20231228171052900.png)



（1）单线程模型

​		优点：可维护性强，性能高，不存在并发读写情况，不存在执行熟悉不确定性，不存在线程切换开销，不会产生死锁，不会产生为了维护数据安全而产生加锁/解锁的开销。

​       缺点：理想来说 redis 存储数据在内存中，内存读写数据为100ns,redis  每秒钟理想化读写数据为 1s/100ns = 1*10^9 / 100 = 10^7=10000000 个数据，而现实 redis 每秒的处理数据为 1w-8w,影响其速度的原因就是单线程模型的原因。

性能会收到影响，由于单线程只占据一个处理器，会造成对处理器的浪费。

​              

（2）多线程模型

![image-20231228174729483](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20231228174729483.png)

## 单线程多路复用技术

![image-20231228175716368](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20231228175716368.png)

对于多路复用器的多路选择算法有三种，select 模型，poll 模型，epoll 模型。

poll模型的选择算法：采用的是轮训算法，该模型对客户端就绪处理是有延迟的。

epoll模型的选择算法，采用的是回掉方式。根据就绪事件发生后的处理方式的不同又可分为 lt 模型与 et 模型。

![image-20231228232600416](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20231228232600416.png)

防止误操作命令，将命令禁用，rename-command flushall 、rename-command flushdb

在配置文件中禁用相应命令。

将手动启动改为守护线程启动

![image-20231229105458880](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20231229105458880.png)

在 redis .config配置文件中，daemonize属性决定，redis 启动是否为后台启动，当为 true 时，为守护线程。

```sql
redis-server /opt/homebrew/etc/redis.conf
```

![image-20231229105904685](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20231229105904685.png)

使用此条命令，redis 由后台挂载的方式启动了。

![image-20231229112642685](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20231229112642685.png)

include属性加载外部配置文件，覆盖原配置文件相同属性。建议卸载配置文件最后一行。

![image-20231229152759826](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20231229152759826.png)

tcp中的 backlog队列长度由内核参数somaxconn来决定，所以，在 redis 中该队列长度由配置文件设置和somaxconn来共同决定：取他们中的最小值。

```sql
cat /proc/sys/net/core/somaxconn
```

 查看linux中 somaxconn 值

![image-20231229154642960](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20231229154642960.png)

生产环境下（特别是高并发情况下),backlog的值最好大一些，否则可能影响系统性能。

### 设置 somaxconn 值

```
vim /etc/sysctl.config
```

在配置文件加一行 

```
net.core.somaxconn = 2048
```

![image-20231229155953321](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20231229155953321.png)

如果是生产环境下不能重启服务器，应该重新加载配置文件。

```
sysctl -p
```

![image-20231229160340379](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20231229160340379.png)

```
requirepass 111
```

![image-20231230170507887](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20231230170507887.png)

 设置值访问 redis 需要输入密码。

![image-20231230172120426](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20231230172120426.png)

## clients模块用于设置客户端相关的属性，其中仅包含一个属性maxclients

maxclients  10000 

maxclients用于设置Redis可并发处理的客户端数量，默认值为 10000，如果达到该最大连接数，则会拒绝再来的新连接，并返回一个异常信息，已达到最大连接数。

注意，该值不能超过linux 系统支持的可打开的文件描述符最大数量阈值。

查看该阈值的方法如下：

![image-20231230172439974](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20231230172439974.png)

```sql
ulimit -n 
```

当前设备阈值为 256 比配置文件中 maxclients 值小 所以 maxclients 值不生效

修改该值，可以通过修改etc/secutiry/limits.config文件的属性。

## THREAD模块

（1）io-threads

![image-20231230182355342](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20231230182355342.png)

该属性用于指定要启用多线程io 模型时，要使用的线程数量，查看当前系统。

查看当前系统中包含的 cpu 数量，

```
lscpu 查看当前系统中包含的 cpu 数量。
```

![image-20231230182721445](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20231230182721445.png)

 

