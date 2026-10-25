## 约定

仅在 `/Operations/*` 下记录运维实践。

## 验证环境

| 项目 | 版本 |
| --- | --- |
| kubernetes | v1.32.3 |
| system OS | Red Hat Enterprise Linux 9.5 |
| network plugin | cilium v1.17.2 |

## 目录

- [Linux](Linux/README.md)：Linux 基础操作、系统管理与排障
- [Shell](Shell/)：Shell 编程
- [CloudNative](CloudNative/README.md)：容器、Kubernetes 与零信任架构
- [Databases](Databases/)：MySQL 与 PostgreSQL 运维
- [DataEngineering](DataEngineering/)：Dataphin 与数仓建设
- [Monitoring](Monitoring/)：监控体系、Prometheus 与 Zabbix
- [Services](Services/README.md)：中间件与基础设施服务

## 关于为什么不记录安装手册

当前各个`linux`发行版的发行商都建立了完备的包repo库；
使用包管理器安装各种服务，命令等，都十分的方便；
所以我认为关于安装这方面没什么可讲的；
当然这并不意味着我不会记录由于各种问题，安装服务失败的解决方式；
以及像`k8s`这种需要多个服务组合而成的服务的搭建部署；
