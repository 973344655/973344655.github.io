---
title: 单例模式
date: 2018-10-31 16:43:55
tags: [java]
---

#### 1.用途
用来维护一个全局的类，保存某些常用数据，减少资源消耗。
#### 2.原理
维护一个类，确保只有单一的对象被创建（创建一次），提供一个唯一的访问接口。私有的构造函数确保外部不能进行访问。

#### 3.两种常用实现
1.饿汉式
线程安全，非懒加载
```
public class Singleton {
    //这句饿汉式的关键，类加载时初始化
    private static  final Singleton instance = new Singleton();
    //维护的数据
    private Map<String,Map> data;
    //构造函数,初始化一次
    private Singleton(){
        this.data = new HashMap<String, Map>();
    };
    //获取实例
    public static final Singleton getInstance(){
        return instance;
    }

    //全局获取维护的数据
    public  Map<String, Map> getData(){
        return this.data;
    }
    //修改数据
    public void setData(Map<String, Map> data) {
        this.data = data;
    }
}
```

2.静态内部类
可用调用时再初始化加载(lazyLoad)。<br>
线程安全，懒加载

```
public class Singleton {
    //静态内部类
    private static class SingletonHolder{
        private static  final Singleton instance = new Singleton();
    }
    //维护的数据
    private Map<String,Map> data;
    //构造函数,初始化一次
    private Singleton(){
        this.data = new HashMap<String, Map>();
    };
    //获取实例
    public static final Singleton getInstance(){
        return SingletonHolder.instance;
    }

    //全局获取维护的数据
    public  Map<String, Map> getData(){
        return this.data;
    }
    //修改数据
    public void setData(Map<String, Map> data) {
        this.data = data;
    }
}

```

使用
```
package com.bonc.sms.controller;

import com.bonc.sms.entity.PlatformChannelSpEntity;
import com.bonc.sms.entity.PlatformEntity;
import com.bonc.sms.service.PlatformService;
import com.bonc.sms.util.Singleton;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Component
public class InitDid implements CommandLineRunner {
    private static final Logger logger = LoggerFactory.getLogger(InitDid.class);
    @Autowired
    PlatformService platformService;
    @Override
    public void run(String... strings) {
        Map data = new HashMap<String,Map>();
        System.out.println("**************************platform*************initdid");
        List<PlatformEntity> platform = platformService.getPlatformAll();
        System.out.println("***:"+ platform.size());
        for(PlatformEntity platformEntity: platform){
//            String url = platformService.getSpUrl(platformEntity.getSp_number()).getUrl();
            Map value = new HashMap<String, String>();
            value.put("platform_name", platformEntity.getPlatform_name());
            value.put("platform_address", platformEntity.getPlatform_address());
            value.put("platform_type", platformEntity.getPlatform_type());
            value.put("platform_key",platformEntity.getPlatform_key());
            List<PlatformChannelSpEntity> platformChannelSpEntities = platformService.getPlatformChannelSp(platformEntity.getPlatform_id());
            List<String> channel = new ArrayList<String>();
            Map<String,String> channel_sp = new HashMap<String, String>();
            Map<String,String> channel_url = new HashMap<String, String>();
            for(PlatformChannelSpEntity platformChannelSpEntity: platformChannelSpEntities){
                channel.add(platformChannelSpEntity.getChannel_id());
                channel_sp.put(platformChannelSpEntity.getChannel_id(),platformChannelSpEntity.getSp_number());
                String url = platformService.getSpUrl(platformChannelSpEntity.getChannel_id());
                channel_url.put(platformChannelSpEntity.getChannel_id(),url);
            }
            value.put("channel",channel);
            value.put("channel_sp",channel_sp);
            value.put("channel_url", channel_url);
            data.put(platformEntity.getPlatform_id(), value);
        }
        Singleton.getInstance().setData(data);
        logger.info("需要的平台缓存信息初始化完成");
    }
}

```
