# 官话介绍

云计算是一种按使用量付费的模式。这种模式提供可用的、便捷的、按需的网络访问， 进入可配置的计算资源共享池（资源包括网络，服务器，存储，应用软件，服务），这些资源能够被快速提供，只需投入很少的管理工作，或与服务供应商进行很少的交互。
也就是把每个人手中的独立资源集中起来，放在一个地方进行统一管理，然后动态分配给每个人使用。而云计算，就是把计算资源集中起来，这个计算资源，包括CPU、内存、硬盘等硬件，还有软件。



# 三个aaS

![](./../../../%E5%9B%BE%E7%89%87%E5%BA%93/%E4%B8%89%E4%B8%AAaas.jpg)

# 1. laaS

**IaaS:Infrastructure-as-a-Service（基础设施即服务)**

> IaaS服务商提供服务器，交换机等硬件设备

# 2. Paas

> **PaaS: Platform-as-a-Service（平台即服务）**

PaaS服务提供商提供各种开发和分发应用的解决方案；

例如：

1. 操作系统
2. 数据库系统等

# 3. SaaS

> **SaaS: Software-as-a-Service（软件即服务）**

用户所使用的服务如，邮件，购票



# openstack架构

![v2-8f583fec34a56f08168a51d34dc5cf3c_r](./../../../%E5%9B%BE%E7%89%87%E5%BA%93/os%E6%9E%B6%E6%9E%84.jpg)

上图中彩色方块为核心组建

| 组件名称   | 服务类型                    | 备注             | 详解链接     |
| ---------- | --------------------------- | ---------------- | ------------ |
| Horizon    | dashboard,web前端服务       |                  | [](#Horizon) |
| Nova       | compute,计算服务            |                  | [](#Nova)    |
| Neutron    | Networking,网络服务         |                  |              |
| Swift      | object Storage,对象存储服务 | 存储服务         |              |
| Cinder     | Block Storage,块存储服务    | Storage Services |              |
| Keystone   | Iidentity service,认证服务  | 共享服务         | [](Keystone) |
| Glance     | Image Service,镜像服务      | Shared Services  |              |
| Ceilometer | Telemetry,监控服务          |                  |              |
| Heat       | Orchestration,集群服务      | 高级服务         |              |





## Nova

> Nova是整个Openstack里面最核心的组件。当初Rackspace和NASA贡献代码时，NASA贡献的那部分就是Nova最早的代码（Rackspace贡献的代码是Swift）。OpenStack云实例生命期所需的各种动作都将由Nova进行处理和支撑，它负责管理整个云的计算资源、网络、授权及测度。

## Keystone

> Keystone为所有的OpenStack组件提供认证和访问策略服务，主要对（但不限于）Swift、Glance、Nova等进行认证与授权。

## Horizon

> Horizon是一个用以管理、控制OpenStack服务的Web控制面板。用户可以通过这个界面对OpenStack状态进行查看和管理