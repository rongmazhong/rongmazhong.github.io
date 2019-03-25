---
layout:       post
title:        "Redis分布式锁"
subtitle:     "接口防并发加锁实现"
date:         2019-03-25
author:       "Rong"
header-mask:  0.3
catalog:      true
tags:
    - Java
    - Redis
    - 分布式锁
---

今天在业务开发的时候常常需要加锁实现并发控制，分布式锁一般有三种实现方式：1. 数据库乐观锁；2. 基于Redis的分布式锁；3. 基于ZooKeeper的分布式锁。之前写过博文做了数据库锁的记录，今天实现下Redis。

直接上码：

```java
/**
     * redis加锁
     *
     * @param code 报表code
     * @return boolean
     */
    private boolean tryLock(long code) {
        RedisClient redisClient = this.redisManager.getRedisClient("");
        Jedis jedis = redisClient.getResource();
        boolean flag = false;
        if (null != jedis) {
            try {
                String ret = jedis.set("REPORT_LOCK_" + code, String.valueOf(code), "NX", "PX", 300000);
                if (StringUtils.isNotBlank(ret) && "OK".equals(ret.toUpperCase())) {
                    LOGGER.debug("redis加锁成功，code：{}", code);
                    flag = true;
                }
            } finally {
                jedis.close();
            }
        }
        return flag;
    }

    /**
     * redis解锁
     *
     * @param code
     */
    private void unlock(long code) {
        this.redisManager.getRedisClient("").del("REPORT_LOCK_" + code);
        LOGGER.debug("redis解锁完成，code：{}", code);
    }
```

加锁关键的一行：`jedis.set(String key, String value, String nxxx, String expx, int time)`，这个set()方法一共有五个形参：

- 第一个为key，我们使用key来当锁，因为key是唯一的；
- 第二个为value；
- 第三个为nxxx，这个参数我们填的是NX，意思是SET IF NOT EXIST，即当key不存在时，我们进行set操作；若key- 已经存在，则不做任何操作；
- 第四个为expx，这个参数我们传的是PX，意思是我们要给这个key加一个过期的设置，具体时间由第五个参数决定；
- 第五个为time，与第四个参数相呼应，代表key的过期时间。  

执行后的结果有：1. 当前没有锁（key不存在），那么就进行加锁操作，并对锁设置个有效期，同时value表示加锁的客户端。2. 已有锁存在，不做任何操作。

[zookeeper的实现参考链接](http://www.hollischuang.com/archives/1716)

撒花*★,°*:.☆(￣▽￣)/$:*.°★* 。