>[!NOTE] 概述
>AWS EFS是一种云文件存储服务，它为Amazon EC2（Elastic Compute Cloud）实例提供可扩展且弹性的文件存储。EFS设计用于在多个EC2实例之间共享文件存储，非常适合需要高并发访问的应用程序，比如内容管理系统、数据分析应用、开发和测试环境等。
>
>**一句话总结**:AWS官方的NFS服务

### 文件系统的创建和管理

**1. 创建EFS文件系统** 在AWS管理控制台中，你可以轻松创建一个EFS文件系统。创建时需要指定VPC（Virtual Private Cloud），因为EFS文件系统会在你的VPC中创建。

**2. 挂载目标（Mount Targets）** 挂载目标是EFS文件系统在特定子网中的端点。想象每个挂载目标就像图书馆的分馆，EC2实例可以连接到这些分馆，访问存储在EFS文件系统中的文件。

**3. 文件系统的性能模式** EFS提供了两种性能模式：通用性能模式和Max I/O性能模式。

- **通用性能模式**（General Purpose）：适用于延迟敏感的应用，如web服务器、内容管理系统等。
- **Max I/O性能模式**：适用于需要高并发和吞吐量的应用，比如大数据分析和媒体处理。

**4. 存储类** EFS有标准存储类和EFS Infrequent Access（IA）存储类。

- **标准存储类**：适合频繁访问的数据。
- **IA存储类**：适合不经常访问的数据，成本更低。

### 数据访问和权限

**1. NFS协议** EFS使用NFS（Network File System）协议，版本为NFSv4和NFSv4.1。NFS协议允许多个EC2实例并发访问同一个文件系统，类似于多个人可以同时借阅图书馆的书籍。

**2. 权限管理** EFS使用AWS IAM（Identity and Access Management）来控制访问权限。你可以设置权限，定义哪些用户和角色可以访问文件系统，类似于图书馆管理系统中，只有注册用户才能借书。

### 数据备份和恢复

EFS自动提供冗余和高可用性。数据存储在多个可用区（Availability Zones）中，确保即使一个区域发生故障，数据仍然安全可用。此外，你可以使用EFS快照功能来备份和恢复文件系统的数据。

### 扩展和性能

EFS具有自动扩展能力，可以根据需要增加或减少存储容量，而无需中断应用程序。性能上，EFS能够提供一致的低延迟和高吞吐量，非常适合需要快速访问大量数据的应用。

### 安全性

EFS支持加密，确保数据在传输过程中和静止状态下都是加密的。就像图书馆的安全系统，确保书籍和借阅信息的安全。

## 关键点总结

1. **可扩展性**：EFS能够自动扩展或缩减容量，满足应用需求。
2. **高可用性**：数据存储在多个可用区，提供高可用性和耐久性。
3. **性能模式**：提供通用性能和Max I/O性能模式，适应不同应用需求。
4. **权限管理**：通过IAM控制访问权限，确保数据安全。
5. **存储类**：标准存储和IA存储类，优化存储成本。