---
title: 流程引擎activiti入门 -- 集成
date: 2019-11-28 15:26:40
tags: [java]
---

# 版本

 springboot 2.0.6

 activiti6.0

# 一.启动

## 1.配置

### pom.xml

 主要是引入 activiti-spring-boot-starter-basic 和 mysql

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>2.0.6.RELEASE</version>
      <relativePath/> <!-- lookup parent from repository -->
  </parent>
  <groupId>com.example</groupId>
  <artifactId>gae</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>war</packaging>
  <name>demo</name>
  <description>Demo project for Spring Boot</description>

  <properties>
      <java.version>1.8</java.version>
  </properties>

  <dependencies>
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-web</artifactId>
      </dependency>

      <dependency>
          <groupId>org.projectlombok</groupId>
          <artifactId>lombok</artifactId>
          <optional>true</optional>
      </dependency>
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-tomcat</artifactId>
          <!--			<scope>provided</scope>-->
      </dependency>
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-test</artifactId>
          <scope>test</scope>
          <exclusions>
              <exclusion>
                  <groupId>org.junit.vintage</groupId>
                  <artifactId>junit-vintage-engine</artifactId>
              </exclusion>
          </exclusions>
      </dependency>
      <!-- activiti -->
      <dependency>
          <groupId>org.activiti</groupId>
          <artifactId>activiti-spring-boot-starter-basic</artifactId>
          <version>6.0.0</version>
      </dependency>
      <dependency>
          <groupId>org.activiti</groupId>
          <artifactId>activiti-spring-boot-starter-actuator</artifactId>
          <version>6.0.0</version>
      </dependency>
      <!-- activiti end-->
      <!-- mybaties -->
      <dependency>
          <groupId>org.mybatis.spring.boot</groupId>
          <artifactId>mybatis-spring-boot-starter</artifactId>
          <version>1.3.0</version>
      </dependency>
      <dependency>
          <groupId>mysql</groupId>
          <artifactId>mysql-connector-java</artifactId>
          <scope>runtime</scope>
      </dependency>
      <!-- mybaties end -->
      <!--fastjson-->
      <dependency>
          <groupId>com.alibaba</groupId>
          <artifactId>fastjson</artifactId>
          <version>1.2.11</version>
      </dependency>
      <!-- fastjson end-->
  </dependencies>

  <build>
      <plugins>
          <plugin>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-maven-plugin</artifactId>
          </plugin>
      </plugins>
  </build>

</project>

```

### application.Properties

```
spring.datasource.driverClassName = com.mysql.jdbc.Driver
spring.datasource.url = jdbc:mysql://192.168.1.23:3306/activiti_test?useUnicode=true&characterEncoding=utf-8&useSSL=false
spring.datasource.username = root
spring.datasource.password = root123


# 配置如果数据库表不存在，自动创建
spring.activiti.database-schema-update=true
```

### 启动类

启动类要排除 SecurityAutoConfiguration.class，因为Activiti带的SecurityAutoConfiguratio会和spring的冲突。

```
import org.activiti.spring.boot.SecurityAutoConfiguration;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;


@SpringBootApplication(exclude = SecurityAutoConfiguration.class)
public class GaeportalApplication {

    public static void main(String[] args) {
        SpringApplication.run(GaeportalApplication.class, args);
    }

}
```

##  2.流程图

在resources路径下创建processes文件夹(activiti默认读取该路劲下的流程文件)

下载idea的插件，actBPM

新建一个流程，然后将该流程文件改为bpmn20.xml格式。如 流程文件 take-leave.bpmn 创建完成后，改为take-leave.bpmn20.xml, 放在processes目录下。

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:dc="http://www.omg.org/spec/DD/20100524/DC" xmlns:di="http://www.omg.org/spec/DD/20100524/DI" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" expressionLanguage="http://www.w3.org/1999/XPath" id="m1576030435332" name="" targetNamespace="http://www.activiti.org/testm1576030435332" typeLanguage="http://www.w3.org/2001/XMLSchema">
  <process id="process_business_opportunities_examine" isClosed="false" isExecutable="true" processType="None">
    <startEvent id="_2" name="StartEvent"/>
    <userTask activiti:assignee="${business.leader1}" activiti:exclusive="true" id="_4" name="商机认领">
      <documentation id="_4_D_1"><![CDATA[商机认领]]></documentation>
    </userTask>
    <userTask activiti:assignee="${business.leader2}" activiti:exclusive="true" id="_5" name="商机信息完善"/>
    <userTask activiti:assignee="${business.leader3}" activiti:exclusive="true" id="_6" name="项目经理审核"/>
    <userTask activiti:assignee="${business.leader4}" activiti:exclusive="true" id="_7" name="领导审核"/>
    <endEvent id="_8" name="EndEvent"/>
    <sequenceFlow id="_9" sourceRef="_2" targetRef="_4"/>
    <sequenceFlow id="_10" sourceRef="_4" targetRef="_5"/>
    <sequenceFlow id="_11" sourceRef="_5" targetRef="_6"/>
    <sequenceFlow id="_12" sourceRef="_6" targetRef="_7"/>
    <sequenceFlow id="_13" sourceRef="_7" targetRef="_8"/>
  </process>
  <bpmndi:BPMNDiagram documentation="background=#3C3F41;count=1;horizontalcount=1;orientation=0;width=842.4;height=1195.2;imageableWidth=832.4;imageableHeight=1185.2;imageableX=5.0;imageableY=5.0" id="Diagram-_1" name="New Diagram">
    <bpmndi:BPMNPlane bpmnElement="process_business_opportunities_examine">
      <bpmndi:BPMNShape bpmnElement="_2" id="Shape-_2">
        <dc:Bounds height="32.0" width="32.0" x="190.0" y="55.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="32.0" width="32.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="_4" id="Shape-_4">
        <dc:Bounds height="55.0" width="85.0" x="160.0" y="150.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="55.0" width="85.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="_5" id="Shape-_5">
        <dc:Bounds height="55.0" width="85.0" x="165.0" y="250.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="55.0" width="85.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="_6" id="Shape-_6">
        <dc:Bounds height="55.0" width="85.0" x="160.0" y="350.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="55.0" width="85.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="_7" id="Shape-_7">
        <dc:Bounds height="55.0" width="85.0" x="160.0" y="435.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="55.0" width="85.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="_8" id="Shape-_8">
        <dc:Bounds height="32.0" width="32.0" x="185.0" y="545.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="32.0" width="32.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="_13" id="BPMNEdge__13" sourceElement="_7" targetElement="_8">
        <di:waypoint x="201.0" y="490.0"/>
        <di:waypoint x="201.0" y="545.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="_12" id="BPMNEdge__12" sourceElement="_6" targetElement="_7">
        <di:waypoint x="202.5" y="405.0"/>
        <di:waypoint x="202.5" y="435.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="_9" id="BPMNEdge__9" sourceElement="_2" targetElement="_4">
        <di:waypoint x="206.0" y="87.0"/>
        <di:waypoint x="206.0" y="150.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="_11" id="BPMNEdge__11" sourceElement="_5" targetElement="_6">
        <di:waypoint x="205.0" y="305.0"/>
        <di:waypoint x="205.0" y="350.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="_10" id="BPMNEdge__10" sourceElement="_4" targetElement="_5">
        <di:waypoint x="205.0" y="205.0"/>
        <di:waypoint x="205.0" y="250.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>

```

# 注意

首次启动，会自动创建28张表，耐心等表创建完。
