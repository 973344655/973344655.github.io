---
title: vim
date: 2019-01-16 14:10:33
tags: [tools]
---
### 1.查找替换

```
全局替换
:%s/old/new/g
当前行替换
:s/old/new/g
替换当前行第一个
:s/old/new/
```
### 2.保存只读文件

```
:w !sudo tee %
```

### 3.块操作

列
```
ctrl + v 进入可视块模式
选择要操作的列
shift + i 输入要操作内容
esc 按两次

```
