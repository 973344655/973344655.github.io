---
title: 流程引擎activiti入门前概念准备
date: 2019-11-28 15:26:40
tags: [java]
---

# 一.核心模块

慢慢理解这块，需要什么功能，就在对应的块里面去找


- RepositoryService 流程仓库Service,可以管理流程仓库例如部署删除读取流程资源

- RuntimeService 运行时Service可以处理所有运行状态的流程实例流程控制(开始,暂停,挂起等)

- TaskService 任务Service用于管理、查询任务，例如签收、办理、指派等

- IdentitiService 身份Service可以管理查询用户、组之间的关系

- FormService 表单Service用于读取和流程、任务相关的表单数据

- HistoryService 历史Service用于查询所有的历史数据

- ManagementService 引擎管理Service，和具体业务无关，主要查询引擎配置，数据库作业

- DynamicBpmService 动态bpm服务，帮助我们在不重新部署的情况下更改流程中的任何内容

- BpmnModel bpmn模型对象，可以用于绘制流程图

# 二.一个流程

使用流程引擎的一般步骤通常是:

画流程图，定义其中的变量和条件

将流程图转为bpmn20.xml文件

runtimeService.startXXXX() 开始一个流程

TaskService.complete(leave.getTaskId(), vars); 完善一个流程的基本信息

taskService.complete(leave.getTaskId(),vars); 根据条件，决定流程走向

流程完成

# 三.查看流程

查看流程信息

查看流程信息(流程图形式)
processEngineConfiguration.getProcessDiagramGenerator().generateDiagram()

# 四.整体感受

![activiti](http://67.216.218.49:8000/file/blogs/java/base/activiti.png)

activiti.cfg.xml 是流程的配置文件，在springboot中，已经不需要它了，直接用spring的配置。
