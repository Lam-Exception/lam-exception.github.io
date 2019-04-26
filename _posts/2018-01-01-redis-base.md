---
layout: post
title: Redis API
description: 全局命令、五种数据结构与内部编码.
category: redis
---

## Redis中键(key)命令

|命令|说明|
|:-------------|:-----------------------------|
| keys *               | 查看所有键             |
| dbsize               | 查询键总数             |
| exists key           | 检查键是否存在          |
| del key [key ...]    | 删除一个或者多个键      |
| expire key seconds   | 设置键过期时间(秒)   	 |
| ttl key              | 检查键剩余过期时间		|
| type key             | 检查键对应值的数据类型   |

### _keys_ `VS` _dbsize_  
`dbsize`命令是直接获取Redis内置的键总数变量；  

`keys`命令是遍历所有的键；

所以当Redis中数据量庞大时，生产环境中严禁使用`keys`命令；

###  _exists key_ `VS` _type key_ `VS` _del key_  
`exists key`如果键存在返回`1`，不存在则返回`0`；

`type key`如果键存在返回值的数据类型，不存在则返回`none`；

`del key`返回已删除键的个数，如果删除的键不存在则返回`0`；

### _ttl key_  
返回正整数或`0`，表示键剩余的过期时间；

返回`-1`，表示键未曾设置过期时间；

返回`-2`，表示键不存在；

​    

## 字符串(string)
字符串类型是最Redis最基础的数据结构，值可以是简单字符串/XML/JSON/整数/浮点数/二进制等类型，但是一个字符串值最大不能超过512MB。  

字符串类型共有三种内部编码：

* int:8个字节的长整形；
* embstr:小于等于39字节的字符串；
* raw:大于39字节的字符串；

```
127.0.0.1:6379> set key_int 8888
OK
127.0.0.1:6379> object encoding key_int
"int"

127.0.0.1:6379> set key_embstr "Hello,welcome to DBW company!"
OK
127.0.0.1:6379> object encoding key_embstr
"embstr"

127.0.0.1:6379> set key_raw "the string greater than 39 byte,the string greater than 39 byte,the string greater than 39 byte!"
OK
127.0.0.1:6379> object encoding key_raw 
"raw"
```

​    

|命令|说明|
|:-------------|:---------------------------------|
|set key value					|设置值			|
|get key						|获取值			|
|del key [key ...]				|删除值			|
|mset key value [key value ...] |批量设置值		   |
|mget key [key ...]				|批量获取值	   	   |
|incr key						|计数自增	    	|
|decr key						|计数自减			|
|incrby key increment			|计数自增指定数字	 |
|decrby key decrement			|计数自减指定数字	 |
|incrbyfloat key increment		|计数自增浮点数	  |
|append key value				|字符串尾部追加      |
|strlen key						|获取字符串长度	  |
|setrange key offset value      |设置指定下标位字符	 |
|getrange key start end			|获取部分字符串	  |

### _set key value [ex seconds] [px milliseconds] [nx|xx]_
 `ex seconds` 设置秒级过期时间；

 `px milliseconds` 设置毫秒级过期时间；

 `nx` 键必须不存在，才可以设置成功，用于添加；

 `xx` 键必须存在，才可以设置成功，用于更新；

 `nx/xx` 可以与 `ex/px` 搭配使用；  

 `ex` 与 `px` 同时使用时，后出现的选项会覆盖前面出现选项的设置值；

### _get key_

如果获取的键不存在，则返回 `nil`；

### _mget & mset_ `VS` get & set

批量操作在实际应用中会大幅提升开发效率，以`get`为例，核心区别在于
```
n次get耗时 = n次网络请求耗时 + n次get命令时间
n次get耗时 = 1次网络请求耗时 + n次get命令时间
```
### incr `VS`  incrby `VS`  decr `VS` decrby `VS`  incrbyfloat 

 `incr` `decr` 命令执行时都必须要求当前键对应的字符串值可以被解析为整数，否则返回错误；  

 `incrby` `decrby` 命令执行时都必须要求当前键对应的字符串值 和 指定增量值都可以被解析为整数，否则返回错误；  

 `incrbyfloat` 命令执行时都必须要求当前键对应的字符串值 和 指定增量值都可以被解析为浮点数，否则返回错误；  

 `incrbyfloat` 命令执行后的结果最多保留到小数点后十七位；  

以上命令执行时如果当前键不存在，则默认该键对应的值为`0`；   

### append

字符串尾部扩展追加字符串值，可以理解为JAVA中 `"defaltString" + "appendString"`；

### setrange

设置当前键对应字符串值中指定下标位的值，下标从`0`开始；

### getrange

截取获得指定下标范围内子字符串，此操作为前后包含，包括`start`下标与`end`下标；

​    

## 哈希(hash)
哈希类型是指键值本身又是一个键值对结构，其中的映射关系称为field-value，注意这里的value是field对应的值，不是键对应的值。

哈希类型的内部编码有两种：
* ziplist:使用紧凑结构实现多个元素的连续存储，更加节省内存；
* hashtable:当有value大于64字节时 或 当field个数超过512个时，内部编码由ziplist转为hashtable，提高读写效率；

|命令|说明|
|:-------------|:---------------------------------|
|hset key field value						|设置值|
|hget key field								|获取值|
|hdel key field [field ...]					|删除值|
|hlen key									|计算当前键值的field个数|
|hgetall key								|获取所有的field-value|
|hmget key field [field ...]				|批量获取|
|hmset key field value [field value ...]	|批量设置|
|hexists key field							|判断field是否存在|
|hkeys key									|获取所有的field|
|hvals key									|获取所有的key|
|hsetnx key field value						|带nx参数设置值，作用域为field|
|hincrby key field increment				|计数自增指定数字，作用域为value|
|hincrbyfloat key field increment			|计数自增指定浮点数，作用域为value|
|hstrlen key field							|计算field对应value长度|

### hset `VS` hsetnx

`hsetnx` 相当于 `hset` 增加 `nx` 参数，作用域为 `field`，`nx` 参数作用参考字符串数据类型；

### hgetall `VS` hkyes & hvals

 `hgetall` 获取所有的 `field-value`；

 `hkeys` 事实上应该叫 `hfields` 获取所有的 `field`；

 `hvals` 获取所有field对应的 `values`;

当哈希元素较多时，请勿使用 `hgetall` 方法，大概率会阻塞Redis；

### hincrby & hincrbyfloat & hstrlen

以上三个方法使用与字符串数据类型相同，只是作用域为 `field` 对应的 `value`；  

​    


## 列表(list)
列表类型用来存储多个有序的字符串，一个列表最多存储 `2^32 - 1` 个元素，并且元素可以重复。

列表类型内部编码有两种：

* ziplist:使用紧凑结构实现多个元素的连续存储，更加节省内存；
* linkedlist:当某个元素大小超过64字节时 或 当元素个数超过512个时，内部编码由ziplist转为linkedlist，便于在列表两端进行操作；

| 命令                                 | 说明                                        |
| :----------------------------------- | :------------------------------------------ |
| rpush key value [value ...]          | 右侧设置值                                  |
| rpushx key value [value ...]         | 右侧设置值                                  |
| lpush key value [value ...]          | 左侧设置值                                  |
| lpushx key value [value ...]         | 左侧设置值                                  |
| linsert key before/after pivot value | 在指定位置之前或之后设置值                  |
| lrange key start end                 | 获取指定范围内的元素                        |
| lindex key index                     | 获取指定位置的元素                          |
| llen key                             | 获取列表长度                                |
| lpop key                             | 从左侧弹出删除一个元素                      |
| rpop key                             | 从右侧弹出删除一个元素                      |
| lrme key count value                 | 删除指定元素                                |
| ltrim key start end                  | 按照范围修剪列表                            |
| lset key index value                 | 修改指定位置元素                            |
| rpoplpush source destination         | 将source中尾部元素弹出添加到destination头部 |
| blpop key                            | lpop阻塞版本                                |
| brpop key                            | rpop阻塞版本                                |
| brpoplpush source destination        | rpoplpush阻塞版本                           |

### lpush & rpush `VS` lpushx & rpushx

在列表两端添加元素，如果一次添加多个元素的情况下会按顺序挨个添加，并非一次性添加多个；

 `lpush` 与 `rpush` 命令下，如果键值不存在，redis会默认提供一个空列表然后进行添加元素；

 `lpushx` 与 `rpushx` 命令下，如果键值不存在，则不执行任何操作；

### linsert

在 `pivot` 位置之前或之后插入一个新元素；

当 `pivot` 不存在列表中时，不执行任何操作；

当键值不存在，被视为空列表，自然不存在`pivot`位置，不执行任何操作；

如果键值不存在 或 为空列表，返回`0`；

如果没有找到`pivot`位置，返回`-1`；

### lindex & lrange

 `lindex` 获取指定下标位置的元素；

 `lrange` 获取指定下标范围内的元素集，此操作为前后包含，包括 `start` 下标与 `end` 下标；

注意这两个方法，下标从左侧开始为`0`到`N-1`，下标从右侧开始为`-1`到`-N`；

### lrem

根据参数 `count` 的值，移除列表中与参数 `value` 相等的元素；

count > 0 : 从表头开始向表尾搜索，移除与 `value` 相等的元素，数量为 `count`；

count < 0 : 从表尾开始向表头搜索，移除与 `value` 相等的元素，数量为 `count` 的绝对值；

count = 0 : 移除表中所有与 `value` 相等的值；

### ltrim

按照索引范围对列表进行裁剪，只保留索引范围内元素，其他元素全部移除；

此操作为前后包含，包括 `start` 下标与 `end` 下标，下标从左侧开始为`0`到`N-1`，下标从右侧开始为`-1`到`-N`；

### rpoplpush 

在一个原子操作时间内将 `source` 列表中的末尾元素弹出添加到 `destination` 列表的头部；

如果 `source` 和 `destination` 为同一个列表，则将列表末位元素移动到头部；

​       

## 集合(set)

集合类型也是用来保存多个字符串元素，但是与列表不同的是集合中不允许出现重复元素，并且集合中的所有元素都是无序的，无法使用下标定位。

集合类型的内部编码有两种：

* intset:当元素个数较少且都为整数时采用，减少内存使用；
* hashtable:当某个元素不为整数时 或 当元素个数超过512个时，内部编码由intset转为hashtable，提高读写效率；

| 命令                                  | 说明                   |
| :------------------------------------ | :--------------------- |
| sadd key member [member ...]          | 添加元素               |
| srem key member [member ...]          | 删除元素               |
| scard key                             | 计算元素个数           |
| sismember key member                  | 判断元素是否在集合中   |
| srandmember key [count]               | 随机获取指定个数的元素 |
| spop key                              | 随机弹出一个元素       |
| smembers key                          | 获取集合中所有元素     |
| smove source destination member       | 两个集合间移动一个元素 |
| sinter key [key ...]                  | 获取多个集合的交集元素 |
| sinterstore destination key [key ...] | 保存多个集合的交集元素 |
| sunion key [key ...]                  | 获取多个集合的并集元素 |
| sunionstore destination key [key ...] | 保存多个集合的并集元素 |
| sdiff key [key ...]                   | 获取多个集合的差集元素 |
| sdiffstore destination key [key ...]  | 保存多个集合的差集元素 |

### sadd

将一个或多个元素加入到集合中，忽略集合中已经存在的元素；

如果集合不存在，则创建一个只包含所有当前被添加元素的集合；

### spop `VS` srem

 `spop` 将返回随机移出的元素；

 `srem` 返回实际移除元素的数量，不包括集合内不存在被忽略的元素；

### srandmember

随机返回指定个数的元素；

当 `count` 为`1`时，与 `spop` 不同的是并不会对集合本身产生改变；

当 `count` 为正数，且小于集合长度时，返回结果中没有重复元素；

当 `count` 为负数，返回 `count` 绝对值个元素，可能会有重复；

### sinter `VS` sinterstore

 `sinter` 获取多个集合的交集元素直接返回；

 `sinterstore` 获取多个集合的交集元素并保存在 `destination` 集合中；

 `destination` 集合可以是自身或已经存在的集合，会直接覆盖；

### sunion `VS` sunionstore

 `sunion` 获取多个集合的并集元素直接返回；

 `sunionstore` 获取多个集合的并集元素并保存在 `destination` 集合中；

 `destination` 集合可以是自身或已经存在的集合，会直接覆盖；

### sdiff `VS` sdiffstore

 `sdiff` 获取多个集合的差集元素直接返回；

 `sdiffstore` 获取多个集合的差集元素并保存在 `destination` 集合中；

 `destination` 集合可以是自身或已经存在的集合，会直接覆盖；

​       

## 有序集合(zset)

在集合的基础上为每个元素设置一个分数 `score` 来进行排序，同样的有序集合中的元素不能重复，但是分数可以重复；

| 命令                                                         | 说明                                 |
| ------------------------------------------------------------ | :----------------------------------- |
| zadd key score member [score member ...]                     | 添加成员                             |
| zscore key member                                            | 获取指定成员分数                     |
| zincrby key increment member                                 | 为指定成员增加指定分数               |
| zcard key                                                    | 计算成员个数                         |
| zcount min max                                               | 计算指定分数范围内成员个数           |
| zrange key start end [withscores]                            | 分数升序获取指定排名范围的成员       |
| zrangebyscore key min max [withscores] [limit offset count]  | 分数升序获取指定分数范围的成员       |
| zrevrange key start end [withscores]                         | 分数降序获取指定排名范围的成员       |
| zrevrangebyscore key min max [withscores] [limit offset count] | 分数降序获取指定分数范围的成员       |
| zrank key member                                             | 分数升序获取成员的排名               |
| zrevrank key member                                          | 分数降序获取成员的排名               |
| zrem key member [member ...]                                 | 删除成员                             |
| zremrangebyrank key start end                                | 分数升序删除指定排名区间范围内的成员 |
| zremrangebyscore key min max                                 | 删除指定分数区间范围内的成员         |
| zinterstore destination numkeys key [key ...] [weights weight [weight ...]] [aggregate sum\|min\|max] | 交集                                 |
| zunionstore destination numkeys key [key ...] [weights weight [weight ...]] [aggregate sum | 并集                                 |

 ### zadd

将一个或多个元素及其分数加入到有序集中，对集合中已经存在的元素更新其分数；

如果集合不存在，则创建一个只包含所有当前被添加元素的集合；

 `score` 值可以是整数值或双精度浮点数；

### zincrby 

为有序集 `key` 的成员 `member` 的 `score` 值加上增量 `increment`；

当 `key` 不存在，或 `member` 不是 `key` 的成员时， `zincrby key increment member` 等同于 `zadd key increment member`；

 `increment` 可以负数；

### zcount

返回有序集 `key` 中， `score` 值在 `min` 和 `max` 之间，包括 `score` 值等于 `min` 或 `max` 的成员的数量；

### zrange `VS` zrevrange

返回有序集 `key` 中，指定区间内的成员，包括 `start` 和 `end` 下标；

其中成员的位置按 `score` 值递增(从小到大)来排序；

具有相同 `score` 值的成员按字典序来排列；

下标参数 `start` 和 `stop` 从头部开始为`0`，从尾部开始为`-1`；

可以通过使用 `withscores` 选项，来让成员和它的 `score` 值一并返回；

相比之下 `zrevrange` 成员按照 `score` 值递减(从大到小)来排序；

### zrangebyscore `VS` zrevrangebyscore 

返回有序集 `key` 中，所有 `score` 值介于 `min` 和 `max` 之间，默认包括等于 `min` 或 `max` 的成员，但是Redis提供可以在 `min` 和 `max` 之前增加 `(` 符号来实现开区间，`min` 和 `max` 可以是 `-inf` 和 `+inf`；

其中成员的位置按 `score` 值递增(从小到大)来排序；

具有相同 `score` 值的成员按字典序来排列；

可以通过使用 `withscores` 选项，来让成员和它的 `score` 值一并返回；

相比之下 `zrevrangebyscore` 成员按照 `score` 值递减(从大到小)来排序；

可选的 `limit` 参数指定返回结果的数量及区间，当 `offset` 值很大时，注意定位 `offset` 可能比较耗时；

### zrank `VS` zrevrank

返回有序集 `key` 中成员 `member` 的排名；

`zrank` 有序集成员按 `score` 值递增 顺序排列，`score` 值最小的成员排名为 `0` ；

`zrevrank` 有序集成员按 `score` 值递减排序，`score` 值最大的成员排名为 `0` ；

### zrem

移除有序集 `key` 中的一个或多个成员，不存在的成员将被忽略；

### zremrangebyrank

移除有序集 `key` 中，指定排名区间内的所有成员，包括 `start` 和 `end` 下标；

下标参数 `start` 和 `stop` 从头部开始为`0`，从尾部开始为`-1`；

移除有序集 `key` 中，所有 `score` 值介于 `min` 和 `max` 之间的所有成员，默认包括等于 `min` 或 `max` 的成员；

Redis提供可以在 `min` 和 `max` 之前增加 `(` 符号来实现开区间，`min` 和 `max` 可以是 `-inf` 和 `+inf`；

### zinterstore 

计算给定的一个或多个有序集的交集，并将该交集(结果集)储存到 `destination` ；

其中给定 `key` 的数量必须以 `numkeys` 参数指定；

可以通过 `weights` 参数为各有续集中的每个成员分数设置计算权重，权重默认为 `1`；

通过 `aggregate` 参数设置，计算成员交集后，分值可以按照 `sum`、`min`、`max` 做汇总；

默认情况下，结果集中某个成员的 `score` 值是所有给定集下该成员 `score` 值之和；

### zunionstore 

计算给定的一个或多个有序集的交集，并将该并集(结果集)储存到 `destination` ；

其中给定 `key` 的数量必须以 `numkeys` 参数指定；

可以通过 `weights` 参数为各有续集中的每个成员分数设置计算权重，权重默认为 `1`；

通过 `aggregate` 参数设置，计算成员并集后，分值可以按照 `sum`、`min`、`max` 做汇总；

默认情况下，结果集中某个成员的 `score` 值是所有给定集下该成员 `score` 值之和；