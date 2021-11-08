---
layout: post
title: "直播系统架构改造"
date: 2021-11-08 20:01:24+08:00
categories: work tech
---

## 直播系统改造

### 接入点
分离出来nginx，直播系统独立一个ngnix，在垂直方向上nginx前面可以部署一个lvs+keepalived。lvs因为是4层协议转发，所以并发能力更强，稳定性更好，keepalived用来保证lvs的高可用。但是仅仅是做反向代理。可以使用dns轮询进行多个ip轮询。

### DB
DB的操作都是由主讲老师进行写入，其他的课中需要落盘的数据可以通过kafka发送并进行落盘，对于非必要对账数据考虑分库进行。为了增强读的能力，采用读写分离。使用mysql的从库增强写的能力。
对于可能出现的缓存穿透，我们可以采用缓存空结果的方式进行规避。

### 缓存
1.基础数据预热进redis
2.避免大key和热key。
3.使用redis cluster增加redis实例。通过读取redis的从节点来增加redis的读能力。
4.发现热key的方式：业务发现，客户端统计，proxy统计。
5.解决热key的方法：a.使用本地缓存，需要注意：I.本地缓存和redis的一致性 II.本地缓存过大是否会影响程序整体性能。b.将热key进行分散。
```
// N 为 Redis 实例个数，M 为 N 的 2倍
const M = N * 2
//生成随机数
random = GenRandom(0, M)
//构造备份新 Key
bakHotKey = hotKey + "_" + random
data = redis.GET(bakHotKey)
if data == NULL {
    data = redis.GET(hotKey)
    if data == NULL {
        data = GetFromDB()
        // 可以利用原子锁来写入数据保证数据一致性
        redis.SET(hotKey, data, expireTime)
        redis.SET(bakHotKey, data, expireTime + GenRandom(0, 5))
    } else {
        redis.SET(bakHotKey, data, expireTime + GenRandom(0, 5))
    }
}
```
注意在过期时间上都有一个random值，为了防止缓存雪崩
6.获取redis中的数据每次尽量不要超过1kb。同时尽量使用pipeline的方式来操作redis，以减少io
7.大key问题，删除的过程要使用类scan命令进行删除。以免造成阻塞。需要考虑将大key进行拆分，string分成多个key/value hash和list等也需要拆分成多个。如果确实要存储长文本，可以考虑mongodb。
