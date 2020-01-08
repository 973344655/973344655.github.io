---
title: mysql分布式锁
date: 2020-1-06 11:02:34
tags: [java]
---


分布式系统中，某个功能只在一台机器上执行(即，只执行一次)，用分布式锁实现。

分布式的CAP理论告诉我们“任何一个分布式系统都无法同时满足一致性（Consistency）、可用性（Availability）和分区容错性（Partition tolerance），最多只能同时满足两项。所以，需要对这三者做出取舍.

# 一.分布式锁

## 1.什么是分布式锁

分布式锁是控制分布式系统或不同系统之间共同访问共享资源的一种锁实现，如果不同的系统或同一个系统的不同主机之间共享了某个资源时，往往需要互斥来防止彼此干扰来保证一致性。

## 2.实现方式


- 基于数据库

- 基于缓存(redis，memcached等)

- 基于Zookeeper

# 二.基于Mysql实现分布式锁

https://www.hollischuang.com/archives/1716

## 1.基于数据表

### 1.1 实现逻辑

```
DROP TABLE IF EXISTS `lock_method_demo`;
CREATE TABLE `lock_method_demo` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `lock_method` varchar(255) NOT NULL COMMENT '需要锁住的方法',
  `desc` varchar(255) DEFAULT NULL COMMENT '备注',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `lock_method` (`lock_method`) USING BTREE COMMENT 'lock_method唯一'
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8;

```

建一张表，用来判断是否加锁，lock_method 字段为加锁的操作，建个索引，将该字段用unique唯一性进行约束。

当一台机器要进行操作时，往该表插入一条数据，然后其它机器要进行相同操作时，也会往该表插入一条数据(lock_method)相同，因为lock_method唯一(数据库保证只有一个操作会成功)，其它机器会插入失败。

代码里面根据插入操作是否成功来决定是否要执行相关功能。当操作成功后,删除该条记录(解锁)。


### 1.2 特点

简单方便

## 2.基于数据库排他锁

### 2.1.for update

在sql的后面加上 for update 可以进行数据加锁，防止高并发出错.InnoDB引擎在加锁的时候，只有通过索引进行检索的时候才会使用行级锁，否则会使用表级锁。

for update 仅适用于InnoDB，并且必须开启事务，在begin与commit之间才生效

详细解释: https://blog.csdn.net/u011957758/article/details/75212222

```
select * from table where xxx for update
```

### 2.2 实现

还是使用上面建的那张表

sql
```
//使用 主键(id)为条件,为行级锁
<select id="methodName" resultType="string" parameterType="int">
    select lock_method from lock_method_demo where id=#{id} for update
</select>

<update id="updateMethod" >
    update lock_method_demo set lock_method=#{method} where id=#{id}
</update>
```

service
```
// Isolation.READ_COMMITTED 保证一个事务修改的数据提交后才能被另外一个事务读取。
@Transactional(isolation = Isolation.READ_COMMITTED,rollbackFor = {Exception.class})
public String lockMethod2(int id){

    System.out.println("[*]: 线程 1 准备读数据");
    String method = mysqlLockTestMapper.methodName(id);
    System.out.println("[*]: 线程 1 读取完数据");
    try {
        Thread.sleep(5000);

    }catch (Exception e){
        e.printStackTrace();
    }

    System.out.println("[*]: 线程 1 更新数据");
    mysqlLockTestMapper.updateMethod("newName1", id);
    return method;
}
@Transactional(isolation = Isolation.READ_COMMITTED,rollbackFor = {Exception.class})
public String lockMethod3(int id){
    System.out.println("[*]: 线程 2 准备读数据");
    String method = mysqlLockTestMapper.methodName(id);
    System.out.println("[*]: 线程 2 读取完数据");
    try {
        Thread.sleep(3000);

    }catch (Exception e){
        e.printStackTrace();
    }

    System.out.println("[*]: 线程 2 更新数据");
    mysqlLockTestMapper.updateMethod("newName2", id);
    return method;
}
```

测试
```
@Component
public class InitDid implements CommandLineRunner {
    @Autowired
    MysqlLockTest1Service mysqlLockTest1Service;

    @Override
    public void run(String... strings) throws Exception {
         Thread thread1 = new Thread(){
             @Override
             public void run() {
                 Long start  = System.currentTimeMillis();
                 System.out.println("[*]: 线程1" + ": " +  mysqlLockTest1Service.lockMethod2(1));
                 Long end  = System.currentTimeMillis();
                 System.out.println("[*]: 线程1耗时: " + (end - start));
             }
         };

        Thread thread2 = new Thread(){
            @Override
            public void run() {
                Long start  = System.currentTimeMillis();
                System.out.println("[*]: 线程2" + ": " +  mysqlLockTest1Service.lockMethod3(1));
                Long end  = System.currentTimeMillis();
                System.out.println("[*]: 线程2耗时: " + (end - start));
            }
        };

        thread1.start();
        thread2.start();
    }
}

```

结果

不加 for update时，结果如下，实际上数据库更新后值为newName1
```
[*]: 线程 2 准备读数据
[*]: 线程 1 准备读数据
[*]: 线程 2 读取完数据
[*]: 线程 1 读取完数据
[*]: 线程 2 更新数据
[*]: 线程2: name1
[*]: 线程2耗时: 4836
[*]: 线程 1 更新数据
[*]: 线程1: name1
[*]: 线程1耗时: 6844
```

加for update
```
[*]: 线程 2 准备读数据
[*]: 线程 1 准备读数据
[*]: 线程 1 读取完数据
[*]: 线程 1 更新数据
[*]: 线程 2 读取完数据
[*]: 线程1: name1
[*]: 线程1耗时: 6866
[*]: 线程 2 更新数据
[*]: 线程2: newName1
[*]: 线程2耗时: 10975
```

因为使用了数据连接池，所以哪个线程先执行sql得看输出。

如结果，线程1sql先执行，数据被加锁(线程2的sql等待)，线程1sql执行完后，线程2的sql再执行。
