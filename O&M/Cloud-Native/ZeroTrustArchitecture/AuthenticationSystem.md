完成本章学习后, 读者应能够达到以下目标:

> 理解现代身份认证(Authentication)与授权(Authorization)的区别, 明确二者在零信任架构中的职责边界.
> 理解 Human Identity、Machine Identity、Service Identity、Device Identity 等不同身份类型及其应用场景.
> 掌握现代身份认证体系的发展历程, 理解从本地账号、LDAP、SSO 到云原生身份体系的演进过程.
> 深入理解 OAuth 2.0 的设计目标、核心角色、授权流程及各种 Grant Type 的适用场景.
> 理解 OpenID Connect(OIDC) 如何在 OAuth 2.0 基础上实现身份认证, 掌握 ID Token 的作用及典型登录流程.
> 掌握 JWT 的结构(Header、Payload、Signature)、签名机制、生命周期及安全风险.
> 理解 Access Token 与 Refresh Token 的职责划分, 掌握 Token Rotation、短生命周期 Token 等最佳实践.
> 理解 PKCE 的设计原理及其在移动端、SPA、公网客户端中的重要性.
> 理解 Kubernetes、Service Mesh、SPIFFE、云 IAM 等云原生身份体系如何构建 Machine Identity 与 Service Identity.
> 掌握 LDAP、Kerberos、SAML、OAuth 2.0、OIDC 等主流身份协议的设计目标、优缺点及适用场景.
> 建立"身份即安全边界(Identity as the New Perimeter)"的零信任思想, 理解现代企业如何构建统一身份平台(Identity Platform).

在零信任架构中，**身份认证（Identity Authentication）** 是整个安全模型的核心。传统网络安全模型假设“进入可信网络后即可默认访问内部资源”，而零信任模型则从根本上否定这一假设：**网络位置不代表可信身份，每一次访问都必须基于身份进行验证**。

因此，零信任体系中的“身份”远不止于人类用户，它还涵盖：

- 服务（Service）
- 工作负载（Workload）
- 设备（Device）
- API 客户端（API Client）
- 自动化任务（Automation / CronJob）
- 甚至短暂的函数调用（Serverless Function）

在现代云原生环境中，一个请求可能经过以下链路：

```
User → Identity Provider (IdP) → API Gateway → Service Mesh → Microservice → Database
```

在这个链条的每一个环节，系统都必须明确两个问题：

- **你是谁？（Authentication，AuthN）**
- **你可以做什么？（Authorization，AuthZ）**

身份认证是一切访问控制的基础，如果身份不可信，后续所有授权、加密、审计都将形同虚设。本章将系统梳理从传统到云原生的身份认证技术演进，并深入探讨 OAuth 2.0、OIDC、JWT、PKCE 等关键技术细节，最终给出零信任时代推荐的身份架构。

---

## 8.1 身份认证的发展

身份系统经历了从简单到复杂、从本地到全球的演进。

### 第一阶段：本地账号认证

早期操作系统直接管理用户凭证：

```
username + password → Operating System
```

代表技术：Linux `/etc/passwd`、Windows Local Account。

特点：简单直接，但无法统一管理，密码泄露风险高，且不具备跨系统能力。

### 第二阶段：企业目录认证

企业需要集中管理员工账号，于是将身份外移到目录服务：

```
User → LDAP Directory → Applications
```

典型场景：企业员工用一套账号访问邮箱、VPN、内部系统。代表技术：LDAP、Active Directory。此阶段实现了集中管理，但协议较重，不适用于互联网和移动场景。

### 第三阶段：联邦身份认证

互联网时代，用户不希望为每个网站创建独立账号，催生了联邦身份：

```
Google Account → Third Party Application
```

通过标准协议（SAML、OAuth、OpenID Connect），一个身份提供者可以跨组织、跨系统被信任，实现单点登录和有限授权。

### 第四阶段：云原生身份体系

微服务和容器化使身份对象极大扩展：人类身份、机器身份、服务身份、设备身份共存。在 Kubernetes 中，Pod 通过 ServiceAccount 获取 JWT Token 来访问 API Server，这标志着工作负载身份的自动化管理成为可能。

云原生身份的核心特征：

- **短生命周期**：凭据动态生成，自动轮换。
- **平台无关**：身份不依赖 IP 或主机名。
- **细粒度**：每个工作负载拥有唯一的身份标识。
- **可证明**：基于密码学证书或令牌，可被验证。

---

## 8.2 Human Identity（人类身份）

人类身份指真实用户身份，例如员工、客户、管理员。典型组件包括 Identity Provider (IdP)、Directory Service、MFA 和 Token Service。

一个标准的人类身份认证流程：

1. 用户通过浏览器或应用登录 IdP。
2. IdP 对用户进行多因素认证。
3. IdP 签发 ID Token 和 Access Token。
4. 客户端携带 Token 访问应用。
5. 应用验证 Token 的签名、有效期、受众等信息。
6. 验证通过后，建立用户会话或授权资源访问。

此过程中，应用从不直接接触用户密码，密码只由 IdP 验证，这符合最小暴露原则。

---

## 8.3 Machine Identity（机器身份）

随着云原生的发展，机器身份的数量和重要性已经远超用户身份。在一个典型的微服务调用链中：

```
Frontend Service → Backend Service → Database
```

Backend 必须向 Database 证明“我是合法的 Backend 服务”。传统做法是 IP 白名单 + 静态密码，这存在密钥泄漏、无法追踪、无生命周期管理等严重缺陷。

现代机器身份方案要求：

- 短期凭据（Short-lived Credential）
- 自动轮换
- 基于平台的证明（Attestation）

例如：Kubernetes ServiceAccount JWT、SPIFFE 身份、云平台 IAM Role（如 AWS IAM Role for Service Accounts）。

机器身份是服务间零信任的基石，没有它，一切服务间流量都无法进行基于身份的授权。

---

## 8.4 Service Identity（服务身份）

Service Identity 是机器身份的一种特化。核心要求：**服务之间通信必须拥有明确、可验证的身份**。

例如，`payment-service` 调用 `order-service`：

```
GET /orders
Authorization: Bearer eyJ...
```

`order-service` 需要完成以下步骤：

1. 验证 JWT 签名。
2. 提取身份声明（如 `spiffe://company.com/payment-service`）。
3. 根据授权策略决定放行或拒绝。

这种模式将服务身份从网络层提升到应用层，使得策略可以基于逻辑服务名而非 IP 地址编写。

---

## 8.5 OAuth 2.0

OAuth 2.0 是当前互联网最广泛使用的授权框架。再次强调：**OAuth 2.0 不是认证协议，而是授权协议**。它解决的核心问题是：一个应用如何安全地获得访问另一个资源的权限，而无需知晓用户的凭证。

**核心角色**

| 角色                   | 说明                         |
| ---------------------- | ---------------------------- |
| Resource Owner         | 资源拥有者（通常为用户）     |
| Client                 | 请求资源的应用               |
| Authorization Server   | 授权服务器（负责签发令牌）   |
| Resource Server        | 资源服务器（存储受保护资源） |

**抽象流程**

```
Client → Authorization Request → Authorization Server
Client ← Access Token ← Authorization Server
Client → Bearer Token → Resource Server
Client ← Protected Resource ← Resource Server
```

OAuth 2.0 通过 Scope 控制授权范围，通过 Refresh Token 实现长会话，已成为 API 经济的基础协议。

---

## 8.6 OAuth Grant Type（授权模式）

OAuth 2.0 定义了多种授权模式（Grant Type）以适配不同场景。这里详细介绍三种关键模式。

### Authorization Code（授权码模式）

最安全、最常用的模式，适用于具有后端服务的 Web 应用。

完整流程：

1. 用户点击登录，浏览器重定向到授权服务器。
2. 用户在授权服务器完成认证并同意授权。
3. 授权服务器通过浏览器重定向返回一次性 Authorization Code。
4. 客户端后端使用该 Code 加上自己的客户端密钥向授权服务器交换 Access Token（和可选的 Refresh Token）。
5. 客户端使用 Access Token 访问资源。

优势：Token 不经过浏览器，前端无法直接获取，安全性高。支持 Refresh Token 实现静默续期。

### Client Credentials（客户端凭证模式）

纯机器对机器（M2M）认证，没有用户参与。

流程：

```
Service A → OAuth Server → Access Token → Service B
```

Service A 使用自己的 `client_id` 和 `client_secret` 直接向授权服务器请求 Access Token。此 Token 代表 Service A 自身的身份。

适用场景：微服务间调用、定时任务、CI/CD 管道等。

### Password Grant（密码模式）

已**不推荐使用**。用户将用户名和密码直接提供给客户端，客户端再向授权服务器换取 Token。这增加了密码泄露面，违背了最小暴露原则，OAuth 2.1 草案已将其移除。

---

## 8.7 OIDC（OpenID Connect）

OAuth 2.0 只解决了授权问题：“你能访问什么？”而零信任还需要认证：“你是谁？” **OpenID Connect (OIDC) 正是建立在 OAuth 2.0 之上的身份层（Identity Layer）**。

OIDC = OAuth 2.0 + ID Token + UserInfo Endpoint

**ID Token**

ID Token 是一个 JWT，包含用户身份声明（Claims），例如：

```json
{
  "iss": "https://idp.example.com",
  "sub": "123456",
  "email": "user@example.com",
  "name": "Alice",
  "aud": "my-app",
  "exp": 1700000000
}
```

客户端验证 ID Token 的签名、issuer、audience 和有效期后，即可确定用户身份，无需额外查询。

**OAuth 2.0 vs OIDC**

| 维度         | OAuth 2.0               | OIDC                     |
| ------------ | ----------------------- | ------------------------ |
| 目标         | 授权                    | 认证                     |
| 核心 Token   | Access Token            | ID Token                 |
| 知道用户身份 | 否                      | 是                       |
| 典型场景     | API 授权                | 登录、单点登录           |

“Login with Google” 之类社交登录，后台实际使用的是 OIDC，而不仅仅是 OAuth 2.0。

---

## 8.8 JWT（JSON Web Token）

JWT 是一种紧凑、自包含的 Token 格式，广泛应用于 OAuth 2.0 和 OIDC 的令牌传输。其结构为：

```
Base64(Header).Base64(Payload).Base64(Signature)
```

**Header**

```json
{
  "alg": "RS256",
  "typ": "JWT"
}
```

**Payload**

包含声明（Claims），例如：

```json
{
  "sub": "user123",
  "role": "admin",
  "exp": 1700000000
}
```

**Signature**

签名部分由头部中声明的算法和服务器私钥生成。验证方使用对应的公钥验证签名，确保 Token 未被篡改。

**JWT 的优势**

- **无状态**：服务器无需维护 session，仅通过验证签名即可信任 Token。
- **可传递**：在微服务链中，JWT 可携带用户身份和权限信息，便于下游授权。

**JWT 的问题与缓解**

- **无法主动撤销**：一旦签发，在有效期内无法强制失效。缓解措施包括缩短有效期（如 5~15 分钟）、使用 Refresh Token、或维护短效 Token 黑名单。
- **体积较大**：携带过多声明会影响网络效率，需合理控制 Payload 大小。

---

## 8.9 Refresh Token

Access Token 应设置为短生命周期（几分钟到一小时），以降低泄露风险。Refresh Token 则具有较长生命周期（天/月级），用于在 Access Token 过期后静默获取新 Token。

流程：

```
Client → Access Token expired
Client → Refresh Token → Authorization Server
Authorization Server → New Access Token
```

安全要求：

- Refresh Token 仅发送给授权服务器，绝不发送给资源服务器。
- 支持 Rotation：每次使用后签发新的 Refresh Token，旧 Token 失效。
- 若检测到 Refresh Token 重用，应立即吊销相关用户的所有 Token。

---

## 8.10 PKCE（Proof Key for Code Exchange）

OAuth 2.0 的 Authorization Code 模式在公共客户端（如 Mobile App、SPA）中存在授权码拦截风险。攻击者可能通过自定义 Scheme 劫持或恶意软件截获授权码，然后冒名换取 Token。

PKCE 通过引入密码学验证来对抗这种攻击：

1. 客户端生成一个高熵随机字符串 `code_verifier`。
2. 计算其 SHA256 哈希得到 `code_challenge`，并随授权请求发送。
3. 授权服务器关联 `code_challenge` 并返回授权码。
4. 客户端在 Token 请求中附上原始 `code_verifier`。
5. 服务器计算哈希并与之前存储的 `code_challenge` 比对，一致则签发 Token。

没有 `code_verifier` 的攻击者即使截获授权码也无法换取 Token。PKCE 已成为 OAuth 2.1 中公共客户端的强制要求，也是零信任构建中移动和 SPA 应用的必要实践。

---

## 8.11 Kubernetes 中的身份体系

Kubernetes 的身份分为用户和服务两类：

- **用户身份**：通常通过 OIDC 集成外部 IdP，API Server 验证 ID Token 并提取用户和组信息用于 RBAC。
- **服务身份**：Pod 通过关联的 ServiceAccount 自动获取 JWT Token（Projected ServiceAccount Token），该 Token 包含 Pod 身份声明，并定期轮换。API Server 通过 TokenReview 或直接验证签名来确认服务身份。

在云环境，Kubernetes 还能与云 IAM 集成：

- **AWS**：IAM Roles for Service Accounts (IRSA)，Pod 获取临时 AWS 凭证。
- **Azure**：Workload Identity，Pod 通过 Microsoft Entra ID 获取 Token。
- **GCP**：Workload Identity，允许 Kubernetes ServiceAccount 扮演 GCP IAM Service Account。

这使得 Pod 可以安全地访问云资源，而无需存储长期密钥，完美契合零信任原则。

---

## 8.12 身份认证协议比较

| 协议       | 定位     | 认证 | 授权 | 典型场景         |
| ---------- | -------- | ---- | ---- | ---------------- |
| LDAP       | 目录协议 | 是   | 有限 | 企业用户目录     |
| Kerberos   | 票据认证 | 是   | 否   | Windows 域环境   |
| SAML       | 身份联邦 | 是   | 有限 | 企业 SSO         |
| OAuth 2.0  | 授权协议 | 否   | 是   | API 授权、委托    |
| OIDC       | 身份认证 | 是   | 是   | 现代登录、SSO     |

---

## 8.13 LDAP

LDAP（轻型目录访问协议）是一种访问和维护分布式目录服务的协议。数据以树状结构存储，例如：`cn=alice, ou=engineering, dc=company`。

优势在于成熟度和企业集成度极高，大多数企业组织架构已基于 AD 或 OpenLDAP 建立。缺点也很明显：协议复杂，不适合直接用于 Web 或 API 认证，缺乏现代化的 Token 机制。在零信任架构中，LDAP 通常作为 IdP 的后端身份源，通过 LDAP Connector 与现代协议（如 OIDC）桥接。

---

## 8.14 Kerberos

Kerberos 是基于对称密钥和票据的认证协议，核心是票据授予票据（TGT）和服务票据。

优势：密码不在网络上传输，支持双向认证，且票据具有时效性。缺点：对时钟同步要求严格，跨域配置复杂，不适用互联网和移动环境。Windows Active Directory 域内仍然广泛使用 Kerberos，但现代应用通常通过 LDAP 或 SAML/OIDC 与其集成。

---

## 8.15 SAML

SAML（安全断言标记语言）是一种基于 XML 的身份联合标准。典型流程：用户访问 Service Provider，被重定向到 Identity Provider，认证后 IdP 向 SP 发送 SAML Assertion，SP 验证后建立安全上下文。

SAML 在企业单点登录领域占据主导地位，大量商业 SaaS（Salesforce、Office 365 等）原生支持。缺点：XML 冗长，签名和加密实现复杂，移动端支持差。新系统倾向于 OIDC，但零信任架构仍需要兼容 SAML 以对接遗留应用。

---

## 8.16 OIDC 与现代身份

OIDC 代表了现代身份协议的主流方向，具有以下核心优势：

- JSON/JWT 简洁易处理。
- 内置发现机制（`.well-known/openid-configuration`），降低集成成本。
- 支持动态客户端注册、RP-Initiated Logout 等扩展。
- 完美兼容 OAuth 2.0，可同时实现认证和授权。

云原生生态中，Kubernetes、Istio、Grafana、GitLab 等均原生支持 OIDC，使其成为零信任用户身份认证的首选。

---

## 8.17 综合能力矩阵

| 能力维度   | Kerberos | LDAP  | SAML  | OAuth 2.0 | OIDC  |
| ---------- | -------- | ----- | ----- | --------- | ----- |
| 诞生年代   | 1980s    | 1990s | 2000s | 2010s     | 2010s |
| 协议格式   | 二进制   | LDAP  | XML   | JSON      | JWT   |
| 核心定位   | 票据认证 | 目录  | 联邦   | 授权       | 认证   |
| 互联网友好 | 低       | 低    | 中     | 高        | 最高   |
| 移动端支持 | 差       | 差    | 一般   | 好        | 好     |
| API 友好性 | 差       | 差    | 一般   | 优秀      | 优秀   |
| 云原生适配 | 差       | 一般  | 一般   | 优秀      | 优秀   |

---

## 8.18 零信任时代推荐架构

现代企业零信任身份平面应当有机整合多种身份类型：

```
人类身份 (OIDC)
        ↓
    Identity Provider
        ↓
   Access Token + ID Token
        ↓
    API Gateway (验证 Token，执行入口策略)
        ↓
    Service Mesh (mTLS + SPIFFE 身份)
        ↓
    服务间调用（基于 Workload Identity 的授权）
        ↓
    Database / 外部资源 (IAM Role / 短期凭据)
```

最终形成多层身份验证与连续验证的统一体：

```
Human Identity + Machine Identity + Service Identity + Device Identity + Continuous Verification
= Zero Trust Identity Plane
```

在此架构中，没有哪一段通信是默认可信的。无论请求来自外部用户、内部服务，还是自动化脚本，都必须出示有效凭据，并且该凭据仅允许完成其职责所需的最小权限。这恰恰就是零信任身份的核心理念：**一切主体皆不可信，每一次访问都必须重新证明自己**。

---

## 本章总结

身份认证体系已经走过了从“用户名+密码”到“短生命周期工作负载身份”的漫长演进：

```
密码 → LDAP → SAML SSO → OAuth 2.0 / OIDC → Workload Identity (SPIFFE)
```

现代云原生环境的最佳实践可以归纳为：

- 人类身份使用 **OIDC** 进行认证，同时获得 ID Token 和 Access Token。
- API 授权使用 **OAuth 2.0**，通过 Access Token 携带 scope。
- Token 格式采用 **JWT**，无状态且易于传递。
- 长期会话采用 **Refresh Token**，降低泄露风险。
- 公共客户端强制启用 **PKCE**。
- 服务间通信使用 **Service Identity（SPIFFE） + mTLS**，实现双向身份认证。
- 自动化工作负载使用 **Machine Identity**，通过平台证明获取短效凭据。

身份不再仅仅是“用户是谁”，而是：**每一个访问主体——无论是人、服务、设备还是定时任务——都必须明确证明自己的身份，并且仅获得完成任务所需的最小权限。** 这是构建零信任安全大厦最底层的基石。