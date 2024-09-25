在中国无法访问 `google` ,但是又不想使用\*\*百度, 这时候就需要SSR代理来绕过防火墙;
你很快会发现,我们无法 ping 通google,今天我就来回答各位的疑惑;
# 首先了解SSR
SSR是基于SS(Shadow socks)开发的 加密代理 软件

原理其实很简单 就是使用socks5代理

socks代理只是简单的传递数据包，而不必关心是何种协议，所以socks代理比其他应用层代理要快的多。

socks5代理是把你的网络数据请求通过一条通道(连接你和代理服务器之间的通道)，由服务器转发到目的地，

这个过程中你是没有通过一条专用通道的，只是数据包的发出，然后被代理服务器收到，整个过程并没有额外的处理

通俗的说：现在你有一个代理服务器在香港，比如你现在想要访问[google](https://www.amjun.com/tag/google/ "查看google的所有文章")，你的电脑发出请求，流量通过socks5连接发到你在香港的服务器上，然后再由你在香港的服务器去访问google，再把访问结果传回你的电脑，这样就实现了翻 墙。

SOCKS是一种网络传输协议，主要用于客户端与外网服务器之间通讯的中间传递。（定义）

socks协议设计之初的目的：就是为了让有权限的用户可以穿过过防火墙的限制 (穿透防火墙)

那这又和能访问google 却 ping 不同google 有什么关系呢?

别急先看下图:

![[annex/TCP.gif]]

我们之所以能够访问google是使用web通过http协议**应用层(第七层)**

ssr的socks代理是介于**传输层(第四层)**和**会话层(第五层)**

而我们在ping的时候，则是基于**网络层(第三层)**

众所周知的常识: 上层代理对下层无用,那么你在ping google时是没有代理在的当然ping不通了;

知道了原因问题就好解决了,我们需要一个可以将所有通信都进行代理的东东就行了;

谈到这个就不得不提一下VPN了，一讲到VPN我们脑海里浮现的是不是就是翻墙

其实不然，FQ只是VPN的一个小功能罢了~

VPN的主要**功能**:在公用网络上**建立专用网络，进行加密通讯。**

VPN的主要**目的**：是给**企业内网直接传输加密数据**，最重要的就是安全性

电脑使用VPN后，所有的网络通信都走代理。适合所有场景。VPN控制的是你电脑的整个网络

言归正传，若是我们想要ping 通Google的话，其实是可以通过VPN代理实现。

### vpn VS ssr

- 特点
    
    SSR是更注重**流量混淆隐秘**,
    
    VPN则是更注重**加密安全性**。
    
- 配置
    
    VPN对服务器有一定的性能要求，容易被攻击，容易被封服务器。成本高。
    
    SSR对服务器的要求很低 64M内存都能搭建，隐蔽性较好。成本很低。

我们可以使用 AWS (一定要是国际版) 的ec2服务,部署一个openVPN服务,在社区镜像中有打包好的,并且AWS 新用户有整整一年的免费ec2可以用;

![[annex/wukong.jpeg]]