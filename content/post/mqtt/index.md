---
title: "MQTT"
description: MQTT (originally an initialism of MQ Telemetry Transport) is a lightweight, publish-subscribe, machine to machine network protocol for message queue/message queuing service. 
date: 2023-02-23T22:25:50+08:00
categories: 
- Linux运维
tags:
- MQTT
- EMQX
---

[EMQX Docs](https://www.emqx.io/docs/en/v5.0/)

## 安装

### Docker安装


[EMQX Docker官方镜像](https://hub.docker.com/_/emqx)

- Docker安装EMQX非常简单 按照官方镜像文档说明创建容器即可
- 注意事项
  - 容器安装的数据持久化
  - 需要持久化文件 参考官方镜像说明文档即可

- Docker Volume持久化参考(`单机测试环境`)

```bash
# 创建所需数据卷
docker volume create emqx_data
docker volume create emqx_etc
docker volume create emqx_log

# 启动容器
docker run -d --name=emqx --restart=always \
--mount source=emqx_data,target=/opt/emqx/data \
--mount source=emqx_etc,target=/opt/emqx/etc \
--mount source=emqx_log,target=/opt/emqx/log \
-p 1883:1883 \
-p 8083:8083 \
-p 8084:8084 \
-p 8883:8883 \
-p 18083:18083 \
emqx/emqx:latest

```
