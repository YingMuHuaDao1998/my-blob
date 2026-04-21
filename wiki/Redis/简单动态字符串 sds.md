# sds 简介

无论是 Redis 的 Key 还是 Value，其基础数据类型都是字符串。例如，Hash 型 Value 的

field 与 value 的类型、List 型、Set 型、ZSet 型 Value 的元素的类型等都是字符串。虽然 Redis

是使用标准 C 语言开发的，但并没有直接使用 C 语言中传统的字符串表示，而是自定义了一

种字符串。这种字符串本身的结构比较简单，但功能却非常强大，称为简单动态字符串，

Simple Dynamic String，简称 SDS。

注意，Redis 中的所有字符串并不都是 SDS，也会出现 C 字符串。C 字符串只会出现在字

符串“字面常量”中，并且该字符串不可能发生变更。

redisLog(REDIS_WARNNING, “sdfsfsafsafds”);

# sds 结构

SDS 不同于 C 字符串。C 字符串本身是一个以双引号括起来，以空字符’\0’结尾的字符序

列。但 SDS 是一个结构体，定义在 Redis 安装目录下的 src/sds.h 中：

```
struct sdshdr {

// 字节数组，用于保存字符串

char buf[];

// buf[]中已使用字节数量，称为 SDS 的长度

int len;

// buf[]中尚未使用的字节数量

int free;

}
```

例如执行 SET country “China”命令时，键 country 与值”China”都是 SDS 类型的，只不过

一个是 SDS 的变量，一个是 SDS 的字面常量。”China”在内存中的结构如下：

![image-20240107224505523](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240107224505523.png)

![image-20240107224528544](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240107224528544.png)

# sds优势

C 字符串使用 Len+1 长度的字符数组来表示实际长度为 Len 的字符串，字符数组最后以

空字符’\0’结尾，表示字符串结束。这种结构简单，但不能满足 Redis 对字符串功能性、安全

性及高效性等的要求。

(1) 防止"字符串长度获取"性能瓶颈

对于 C 字符串，若要获取其长度，则必须要通过遍历整个字符串才可获取到的。对于超

长字符串的遍历，会成为系统的性能瓶颈。

但，由于 SDS 结构体中直接就存放着字符串的长度数据，所以对于获取字符串长度需要

消耗的系统性能，与字符串本身长度是无关的，不会成为 Redis 的性能瓶颈

