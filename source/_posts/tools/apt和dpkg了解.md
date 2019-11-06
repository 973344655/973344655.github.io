---
title: apt和dpkg
date: 2019-10-23 17:18:10
tags: [tools]
---

# 一.为什么写这篇总结

经常使用apt和dpkg安装软件，但是从没有详细的看过它们有哪些用法。经常忘记，所以记录一下。

# 二.apt

## 1.apt是什么
apt是debian和ubuntu中的包管理工具，适用于deb包，主要为了 便捷的 在软件仓库中搜索,安装,更新和卸载软件.

## 2.apt源
apt 使用的软件源 在 /etc/apt/sources.list 中配置

### 2.1 apt

```
apt updated 根据配置的源，更新包信息

apt upgrade 升级已安装的包

apt dist-upgrade 升级系统

apt install package  安装包

apt install package=version 安装（或升级降级到）指定版本

apt install package --reinstall 安装已经安装，但可能存在问题的包

apt install -f 修复依赖关系，缺少某些包会自动安装

apt remove package 删除包

apt autoremove package 删除包，及其依赖包

apt remove package --purge 删除包，及其配置文件

apt autoclean 清理无用的包
```

### 2.2 apt-cache

```
apt-cache search reg(正则) 搜索包
apt-cache show package 显示包的相关信息
```

# 三.dpkg

dpkg用于debian或ubuntu安装.适用于deb包

```
dpkg -i package 安装软件包

dpkg -l 显示已安装软件，太多，可以grep

dpkg -s package 显示已安装的包的信息

dpkg -L package 显示已安装包的文件信息(有哪些文件，在哪些目录)

dpkg -r package 删除包

dpkg --purge package 纯净删除，包括配置文件

dpkg -c package 查看一个安装包中包含的文件，及安装后的文件路径
```

# 四.apt和dpkg关系

因为 apt install 自动安装的也是.deb包。所以，当安装某个包，发现缺失某些依赖包，在apt中找不到，又不想换源，或者换源也找不到，可以在网上将依赖包包(.deb)下载下来，用dpkg 来安装。
