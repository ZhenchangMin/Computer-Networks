# Lec2: Internet Applications

## Internet Applications Overview

**Application（应用）**：运行在端系统（end systems）上的、互相通信的分布式进程(processes)

- 例如：Email, Web, P2P文件共享
- 只运行在端系统上，**不运行**在网络核心设备（服务器）上
- 进程之间**交换消息**（exchange messages）来实现应用功能


**Application-layer protocols（应用层协议）**：

- 是应用程序的一个部分
- 定义应用之间**交换的消息类型和动作**
- 使用下层协议（TCP, UDP等）提供的通信服务（传输层协议）来传输消息

---

### 典型互联网应用一览

| Application          | App-Layer Protocol         | Transport Protocol |
| -------------------- | -------------------------- | ------------------ |
| Email                | SMTP [RFC 2821]            | TCP                |
| Remote terminal      | Telnet [RFC 854]           | TCP                |
| Web                  | HTTP [RFC 2616]            | TCP                |
| File transfer        | FTP [RFC 959]              | TCP                |
| Streaming multimedia | Proprietary (e.g. RealNetworks) | RTP, TCP or **UDP** |
| Internet telephony   | Proprietary (e.g. Dialpad) | SIP on **UDP**     |

> 注意：实时性要求高的应用（流媒体、网络电话）倾向于使用UDP，因为它延迟低，不强制重传。

---

## 应用层关键术语（Jargons）

### Process（进程）
**Process**：运行在某台主机内的程序

- **同一主机**的两个进程通信：使用**操作系统**提供的**进程间通信（IPC）**机制
- **不同主机**的两个进程通信：使用**应用层协议**

### User Agent（用户代理）

- **User agent**：介于用户界面（"上层"）和网络（"下层"）之间的**接口**
- 实现用户界面 + 应用层协议，例如：
  - Web：浏览器（browser）、Web服务器
  - Email：邮件客户端、邮件服务器
  - 流媒体：媒体播放器、媒体服务器

### socket（套接字）
- **socket**：应用层与传输层之间的接口，提供**进程间通信**的抽象
- 进程通过 socket 发送/接收消息，socket 负责调用传输层协议（TCP/UDP）来传输消息

---

## App-Layer Protocol 定义了什么？

| 要素 | 含义 | 例子 |
|------|------|------|
| **Types**（消息类型） | 交换哪些种类的消息 | request / response |
| **Syntax**（语法） | 消息包含哪些字段，字段如何分隔 | HTTP header格式 |
| **Semantics**（语义） | 字段的含义是什么 | 状态码200=OK |
| **Rules**（规则） | 进程何时、如何发送/响应消息 | 先发GET，再收200 |

---

## Application Architectures（应用架构）
应用架构由app研发者设计，规定如何在各种端系统上部署应用进程，以及它们如何通信。

两种主要结构：**Client-Server（CS）** 和 **Peer-to-Peer（P2P）**

### Client-Server Paradigm（客户端-服务器模式）

```
        Client A ──request──►┐
        Client B ──request──►├──► Server（always-on）
        Client C ──request──►┘       │
                            ◄──reply─┘
```

**Client（客户端）**：
- 按需启动（start as required）
- 主动发起连接，"先开口"（speaks first）
- IP地址可以是动态的
- 例子：Web浏览器、邮件客户端

**Server（服务器）**：总是打开，等待客户端连接
- 以守护进程方式**常驻运行**（run as daemon, always-on）
- 响应客户端请求，提供服务
- 拥有**固定的永久IP地址**
- 例子：Web服务器发送网页，邮件服务器投递邮件

---

### Peer-to-Peer Paradigm（点对点模式）

```
  Peer A ◄──────────────────► Peer B
    ▲                              ▲
    │                              │
    └──────────────────────────────┘
              peer-peer
    ▲                              ▲
    └──────────────────────────────┘
  Peer C ◄──────────────────► Peer D
```

- **没有**常驻服务器
- 任意端系统**直接互相通信**
- 每个peer**既请求服务，又提供服务**
- **自扩展性（self scalability）**：新加入的peer带来新的服务能力，也带来新的工作负载
- Peer的IP地址是**间歇性连接且可变**的 → 高度可扩展，但难以管理
- 例子：Gnutella、文件共享应用BitTorrent

---

### 混合模式：CS + P2P

现实中很多应用是两者混合的：

| 应用 | P2P部分 | CS部分 |
|------|---------|--------|
| **Skype** | 用户之间的语音通话（直连） | 中央服务器负责查找对方地址 |
| **即时通信（IM）** | 两用户之间聊天（直连） | 中央服务器负责用户在线检测/定位 |

> **Skype流程**：上线 → 向中央服务器注册IP → 呼叫时查询中央服务器获得对方IP → 直接P2P通话

---

## Web and HTTP

Web的按需操作使得它成为了**最流行的互联网应用**，HTTP是Web的应用层协议。

---

### HTTP概述

**HTTP**（HyperText Transfer Protocol）是Web的应用层协议，定义了Web客户端（浏览器）和Web服务器之间的通信规则。

Web页面（文档）是由**对象**组成的。
一个对象是一个文件，如HTML文件、JPEG图片、Java applet等，可以通过一个**URL**（Uniform Resource Locator）唯一寻址。

每个URL由两个部分组成：
- **主机名**：URL中的主机部分，指明对象所在的服务器
- **路径名**：URL中的路径部分，指明对象在服务器上的位置
例子：www.someschool.edu/dir1/dir2/file.html
www.someschool.edu 是主机名，/dir1/dir2/file.html 是路径名

因为**Web浏览器**实现了Web的客户端，所以在Web环境中浏览器和客户这两个术语可以互换使用。

**Web服务器**实现了Web的服务器端，负责存储Web对象并响应浏览器的请求。

HTTP使用TCP作为传输层协议，默认端口号为80。
客户首先发起一个与服务器的TCP连接，然后浏览器和服务器进程就可以通过套接字进行通信。

因为HTTP不保存任何有关客户的信息，所以它被称为**无状态协议（stateless protocol）**。
假定某个客户在几秒内连续请求同一个对象，服务器不会因为刚刚请求过就不做反应，服务器会对每个请求都进行处理。

Web使用了client-server架构，Web服务器总是打开的。

---

### 非持续连接和持续连接
对于应用程序而言，客户和服务器可能在一个相当长的时间范围内通信，对于客户发出的一系列请求，服务器要对每个请求加以响应。如果每个请求/响应对通过一个**单独的**TCP连接来完成，那么这就是**非持续连接（non-persistent connection）**。
如果所有的请求/响应对通过一个**相同的**TCP连接来完成，那么这就是**持续连接（persistent connection）**。

下面我们研究对于特定的应用程序HTTP，使用持续连接和非持续连接的优缺点。尽管其默认采用持续连接，但HTTP也支持非持续连接。

#### 采用非持续连接的HTTP
通过非持续连接，传送一个Web页面的步骤：（假设该页面含有1个html文件和10个图像文件）
并且假设改html文件的url是`http://www.someschool.edu/someDepartment/home.index`
1) HTTP客户进程在端口号80发起一个TCP连接到服务器www.someschool.edu的端口号80。客户和服务器上分别有一个套接字与该连接相关联。
2) HTTP客户通过套接字向服务器发送一个HTTP请求报文（消息）。
3) HTTP服务器经过他的套接字接受该请求报文，从其存储器中检索出对象`someDepartment/home.index`，在HTTP响应报文中封装该对象，并通过套接字向客户发送响应报文。
4) HTTP服务器进程通知TCP断开该TCP连接，在TCP确认客户已经完整地收到响应报文之后，才会实际关闭连接。
5) HTTP客户接收响应报文，TCP连接关闭。客户从报文中提取出html文件并检查。
6) 对每个引用的jpeg图像文件，客户重复步骤1-5。共10次。

每个TCP连接在服务器发送一个对象后关闭，不为其他对象而持续下来，每个TCP连接只传输**一个**请求报文和响应报文。实际上，所有TCP连接可能不是完全**串行**的，现代浏览器可以同时打开多个TCP连接来**并行**地请求和接收对象，从而缩短响应时间。

HTTP仅定义了HTTP客户程序和服务器程序之间的通信协议，至于浏览器**如何解释**（向用户显示）从服务器接收的对象，HTTP并不关心。

**往返时间**（Round-Trip Time, RTT）是指一个短分组（small packet）从客户到服务器然后再返回客户所花费的时间。
RTT包括分组传播时延、分组在中间路由器和交换机上的排队时延以及分组处理时延。

如果用户点击超链接：
三次握手的过程。
发起TCP连接：一个RTT
请求文件：一个RTT
接收到整个文件：文件传输时间
因此总响应时间是2RTT+HTML文件传输时间

#### 采用持续连接的HTTP
非持续连接的缺点：
- 必须为每一个请求的对象建立和维护一个新的TCP连接，增加了开销
- 每一个对象需要经受两倍的RTT时延，即一个RTT用于创建TCP连接，另一个RTT用于发送HTTP请求和接收HTTP响应

在持续连接中，通常，如果一个连接经过一定时间间隔仍未被使用，就会被HTTP服务器关闭。

### HTTP报文格式（message format）
HTTP报文有两种：**请求报文**和**响应报文**。

#### HTTP请求报文
一个简单的典型的HTTP请求报文如下：
```
GET /somedir/page.html HTTP/1.1
Host: www.someschool.edu
Connection: close
User-agent: Mozilla/5.0 
Accept-language: en-us
```
该报文是由普通的ASCII文本书写的，有五行，每行以回车和换行符结尾。
实际上一个请求报文能有更多行或者至少为1行

第一行是**请求行**（request line），包含三个字段：方法（method）、URL和HTTP版本号。
- 方法字段：指定要执行的操作，如GET、POST、HEAD、PUT、DELETE等，绝大多数采用GET方法
- URL字段：指定要请求的对象的URL
- HTTP版本号字段：指定HTTP协议的版本，常见的有HTTP/1.0和HTTP/1.1

请求行后面的是**首部行**（header lines）
首部行Host: www.someschool.edu 指定了服务器的主机名，HTTP/1.1要求必须有Host首部行
首部行Connection: close 指定了服务器在发送完响应报文后关闭TCP连接
首部行User-agent: Mozilla/5.0 指定了客户使用的浏览器类型
首部行Accept-language: en-us 指定了客户接受的语言类型

看过一个例子之后，一个请求报文的通用格式是在首部行后面还有一个**附加的回车和换行符**，接着有一个实体体（entity body），实体体包含了要发送给服务器的数据，如表单数据等。
一个实体体的例子
```
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 31

<html><h1>Hello</h1></html>
```
使用GET的时候，实体体通常是空的；使用POST的时候，实体体包含了要发送的数据（提交的表单），比如用户向搜索引擎输入的关键词。

**表单**就是网页里让用户填写信息并提交的区域，例如搜索框、登录框等。表单数据可以通过HTTP请求报文发送给服务器，服务器根据这些数据进行处理并返回结果。
值得一提的是并不是用表单生成的请求报文都要用POST方法，GET方法也可以用来提交表单数据，这时表单数据会被编码成URL的一部分，附加在URL字段中，例如：
```
GET /cgi-bin/search?monkeys&bananas HTTP/1.1
```

HEAD方法类似GET方法，但服务器在响应时**不返回实体体**，只返回首部行和状态行。HEAD方法常用于检查某个对象是否存在以及获取对象的元数据（如大小、修改时间等），而不需要下载整个对象。

PUT方法用于将客户提供的数据存储在服务器上指定的位置，DELETE方法用于删除服务器上的指定对象

#### HTTP响应报文
下面是一个例子，是对上面的请求报文的响应报文：
```
HTTP/1.1 200 OK
Connection: close
Date: Wed, 22 Jul 2020 19:15:56 GMT
Server: Apache/2.4.41 (Ubuntu)
Last-Modified: Wed, 22 Jul 2020 19:15:56 GMT
Content-Length: 1024
Content-Type: text/html
(data, data, data, ...)
```
该报文的第一行是**状态行**（status line），包含三个字段：HTTP版本号、状态码和状态短语。
- HTTP版本号字段：指定HTTP协议的版本，常见的有HTTP/1.0和HTTP/1.1
- 状态码字段：是一个三位数字，表示服务器对请求的处理结果，常见的状态码有：
  - 200 OK：请求成功，服务器返回了请求的对象
  - 301 Moved Permanently：请求的对象被永久移动到新的URL，响应报文中会包含新的URL
  - 400 Bad Request：请求报文有语法错误，服务器无法理解
  - 404 Not Found：请求的对象不存在
  - 505 HTTP Version Not Supported：服务器不支持请求报文中指定的HTTP版本

状态行后面是**首部行**（header lines），然后是**实体体**。
在这个例子里面有6个首部行：
- Connection: close 指定了服务器在发送完响应报文后关闭TCP连接
- Date: 指定了服务器**发送响应报文**的日期和时间，而不是对象创建或者修改的时间
- Server: 指定了服务器使用的软件类型和版本，类似于HTTP请求报文中的User-agent首部行
- Last-Modified: 指定了服务器上对象的创建或者最后修改时间
- Content-Length: 指定了被发送对象的字节数
- Content-Type: 指定了实体体中的对象的类型，例如text/html表示HTML文件，image/jpeg表示JPEG图片等。

### 用户与服务器的交互：cookie
cookie允许站点对用户进行跟踪，记录用户的状态信息（如登录状态、购物车内容等），并在用户再次访问时提供个性化的体验。
cookie技术有4个组件：
1) 在HTTP响应报文中的一个cookie首部行
2) 在HTTP请求报文中的一个cookie首部行
3) 用户端系统中保留的一个cookie文件，由用户的浏览器进行管理
4) 位于Web站点的一个后端数据库

cookie可以用于标识一个用户。用户初次访问一个站点时，假设是amazon.com，服务器会在HTTP响应报文中包含一个cookie首部行，例如：
```
Set-Cookie: 1678
```
在这个例子中，服务器为用户分配了一个cookie值1678，并通过Set-Cookie首部行发送给用户的浏览器。用户的浏览器会将这个cookie值保存在本地的cookie文件中。以后，当用户每次访问amazon.com时，浏览器会在每个HTTP请求报文中包含一个cookie首部行，例如：
```
Cookie: 1678
```
因此服务器就可以通过这个cookie值来识别用户，并提供个性化的服务，例如显示用户的购物车内容、推荐商品等。

### Web缓存
Web缓存也叫代理服务器（proxy server），是一个**中间服务器**，能够代表初始Web服务器来满足客户的HTTP请求。
Web缓存器有自己的磁盘存储空间，可以用来存储从Web服务器获取的对象的副本。当一个客户请求一个对象时，假设是`http://www.cnn.com/index.html`，Web缓存器首先检查它的磁盘存储空间中是否有该对象的副本，如果有，就直接返回给客户；如果没有或者副本已经过期，就向`http://www.cnn.com`发出HTTP请求来获取该对象，并将获取到的对象返回给客户，同时将该对象的副本保存在磁盘存储空间中，以便下次请求时可以直接使用。

Web缓存器既是服务器又是客户。Web缓存器可以大大减少对客户请求的响应时间，并且能够减少一个机构的接入链路到因特网的通信量，从而降低机构的网络成本。同时，Web缓存器整体上大大减少了因特网上的Web流量，改善了应用性能。

但是在缓存器里面的对象可能是陈旧的，因此我们可以通过条件GET。
如果HTTP请求报文使用了GET方法，并且包含了一个If-Modified-Since首部行，例如：
```
GET /index.html HTTP/1.1
If-Modified-Since: Wed, 22 Jul 2020 19:15:56 GMT
```
缓存器在缓存对象的同时，也缓存了响应报文中的最后修改日期。通过发送条件GET可以执行最新检查，只有在指定日期之后该对象被修改过，才发送该对象。
下面是对条件GET请求的响应报文的例子：
```
HTTP/1.1 304 Not Modified
Date: Wed, 22 Jul 2020 19:15:56 GMT
Server: Apache/2.4.41 (Ubuntu)

(enpty entity body)
```
注意这里用了状态码304 Not Modified，表示该对象自指定日期以来没有被修改过，因此缓存器可以继续使用它的副本，而不需要从服务器获取该对象。
这个响应报文里面没有包含所请求的对象，包含对象只会浪费带宽和增加响应时间。

## Domain Name Service (DNS)

### 为什么需要DNS？

人类喜欢用好记的名字（`www.baidu.com`），但网络通信需要**IP地址**（`119.75.217.109`）。

DNS的功能：**将域名映射为IP地址**（domain name → IP address）

---

### DNS系统设计

**为什么不用集中式DNS？**
集中式，就是所有域名和IP地址映射都存储在一台服务器上：

| 问题 | 说明 |
|------|------|
| 单点故障 | 一台服务器宕机 → 全网瘫痪 |
| 流量过大 | 全球所有查询都打到一台服务器 |
| 物理距离 | 远端用户查询延迟极大 |
| 维护困难 | 全球所有域名集中维护 |

**结论：集中式DNS无法扩展（doesn't scale!）**

**DNS的实际设计**：
- **分布式数据库**：由层次化的多个**名字服务器（name servers）**实现
- **应用层协议**：DNS本身是应用层协议，用于主机和名字服务器之间通信来"解析"域名
- **负载均衡**：一个域名可映射多个IP地址（轮询分发）

---

### DNS的设计目标（Goals）

| 目标 | 说明 |
|------|------|
| **唯一性** | 无命名冲突 |
| **可扩展性** | 支持海量名字和频繁更新 |
| **分布式自治管理** | 各机构自主管理自己的名字，无需跟踪全局变化 |
| **高可用性** | 查询服务随时可用 |
| **查询快速** | 解析延迟要低 |
| **完美一致性是非目标** | 允许短暂的不一致（缓存过期等） |

---

### DNS的实现方式：层次化（Hierarchy）

解决方案：**划分命名空间（partition the namespace）**，对每个分区**分别分配管理权和解析服务**

DNS的三种相互交织的层次结构：

```
  1. 层次化命名空间（Hierarchical namespace）
     ─ 以前是平坦的，现在是树状的

  2. 层次化管理（Hierarchically administered）
     ─ 以前集中管理，现在各机构自治

  3. 分布式服务器层次（Hierarchy of servers）
     ─ 以前集中存储，现在分布式存储
```

---

### 层次化命名空间（Hierarchical Namespace）

```
                              root
              ┌──────┬────┬──────┬────┬────┬────┬────┐
             edu    com  gov   mil  org  net  uk   fr  ...
            /   \
        umich  berkeley
        /    \
      eecs   law
      /
    cse

    完整名字（从叶到根）：cse.eecs.umich.edu
```

- **顶级域（Top Level Domains, TLD）**：edu, com, gov, org, uk, fr...
- **域（Domain）= 子树**：例如 `.edu`、`umich.edu`、`eecs.umich.edu`
- **名字 = 从叶节点到根的路径**：`cse.eecs.umich.edu`
- 树的深度任意（上限128层）
- **命名冲突自然避免**：每个域自己负责其子域名字的唯一性

---

### 层次化管理（Hierarchical Administration）

```
  ┌─────────────────────────────────────────┐
  │  ICANN/IANA 管理: root + 所有TLD         │
  │                  root                   │
  │         edu  com  gov  mil  org  net... │
  └─────────────────────────────────────────┘
          │edu
  ┌────────────────┐  ┌─────────────────┐
  │ UMich 管理:    │  │ Berkeley 管理:  │
  │  umich.edu     │  │  berkeley.edu   │
  │  *.umich.edu   │  │  *.berkeley.edu │
  └────────────────┘  └─────────────────┘
  ┌────────────────┐
  │ EECS 管理:     │
  │  eecs.umich.edu│
  │  *.eecs.umich  │
  └────────────────┘
```

- **Zone（区域）**：对应一个**管理权威（administrative authority）**，负责该区域内的所有名字
- 例如：UMich 控制 `*.umich.edu`，EECS 控制 `*.eecs.umich.edu`

---

### DNS服务器的层次结构

```
                    ┌──────────────────┐
                    │  Root Name Server│  ← 当本地服务器无法解析时联系
                    └────────┬─────────┘
                             │
            ┌────────────────┼────────────────┐
            ▼                ▼                ▼
     ┌─────────────┐  ┌────────────┐  ┌─────────────┐
     │  TLD Server │  │ TLD Server │  │  TLD Server  │
     │  (.com)     │  │  (.edu)    │  │  (.cn/.uk)   │
     └──────┬──────┘  └─────┬──────┘  └──────────────┘
            │               │
     ┌──────▼──────┐  ┌─────▼──────────┐
     │Authoritative│  │  Authoritative  │
     │DNS (google) │  │DNS (umich.edu)  │
     └─────────────┘  └────────────────┘
```

| 服务器类型 | 职责 |
|------------|------|
| **Root name servers** | 当本地服务器无法解析时被联系；知道所有TLD服务器地址 |
| **Top-level domain (TLD) servers** | 负责 com, org, net, edu 等及各国家域（cn, uk, fr） |
| **Authoritative DNS servers** | 组织自己的DNS服务器，提供该组织内**主机名到IP的权威映射** |
| **Local Name Servers** | 由ISP/公司/学校维护；主机发出查询时**首先**发到这里 |

// 学到这里了

**关于DNS数据库存储**：
- 每台服务器只存储整个DNS数据库的一个**小子集**
- 权威DNS服务器存储其负责域内所有名字的 **resource records**
- 每台服务器需要知道其他部分由谁负责：
  - 所有服务器都**知道根服务器地址**
  - 根服务器知道**所有TLD服务器地址**

---

### Root Name Servers（根名字服务器）

- 功能：**返回 TLD 服务器的 IP 地址映射**
- 全球共有 **13 个根名字服务器**（以字母 a–m 命名），每个都有多个副本站点（anycast）
![1776241136564](image/lec2InternetApplications/1776241136564.png)
根名称服务器是 DNS 查询流程的第一步：

当你查询一个域名时，如果本地 DNS 缓存中没有，就会查询根名称服务器
根名称服务器告诉你该域名的 TLD 服务器在哪里，然后继续查询权威名称服务器，获得最终的 IP 地址

---

### TLD & Authoritative Servers

**TLD 服务器（Top-Level Domain Servers）**：
- 负责 `com`, `org`, `net`, `edu`, `aero`, `jobs`, `museums` 以及所有国家域（`uk`, `fr`, `ca`, `jp`…）
- 例：Network Solutions（一个公司） 维护 `.com` TLD 服务器；Educause（一个教育组织） 维护 `.edu` TLD 服务器

**权威 DNS 服务器（Authoritative DNS Servers）**：
- 组织**自己的** DNS 服务器，提供该组织内主机名到 IP 的**权威映射**
- 可由组织自己维护，也可外包给服务商

---

### Local DNS Name Server（本地名字服务器）
ISP 是 Internet Service Provider，互联网服务提供商或网络运营商
ISP 就是“给你提供上网服务”的公司/机构

- **不严格属于层次结构**，但是是查询的入口
- 每个 ISP（住宅ISP、公司、大学）都有一个，也叫 **default name server（默认名字服务器）**
- 主机发起 DNS 查询时，**首先发往本地 DNS 服务器**：
  - 本地服务器维护**近期查询结果的缓存**（name-to-address translation pairs），但可能过期
  - 作为**代理（proxy）**，将查询转发到层次体系中

---

### DNS 名字解析过程（Name Resolution）

以 Bob在`cis.poly.edu`的主机上查询 Alice`gaia.cs.umass.edu`的 IP 地址为例：

```
请求主机                 本地DNS服务器              根DNS服务器
cis.poly.edu           dns.poly.edu
    │                      │
    │──(1)查询：            │
    │  先访问本地DNS服务器─► │
    │                      │──(2) 查询─本地 DNS 去问根 DNS 服务器─────►│
    │                      │◄─(3) 返回TLD服务器地址──│
    │                      │──(4) 查询 TLD(.edu)──►TLD DNS
    │                      │◄─(5) 返回权威服务器────TLD DNS
    │                      │──(6) 查询权威服务器──►dns.cs.umass.edu
    │                      │◄─(7) 返回最终IP────────│
    │◄─(8) 返回IP──────────│
```
- Iterated query（迭代查询）：被问到的服务器只告诉你“下一站去哪问”，不替你问到底
- Host-Server: **recursive** query：主机对本地 DNS 是“递归”期望（你帮我查到底）
- Server-Server: **iterative** query：DNS **服务器之间**通常是“迭代”方式（一跳一跳指路）

1.请求主机 cis.poly.edu 先问本地 DNS 服务器 dns.poly.edu
2.本地 DNS 去问根 DNS 服务器
3.根 DNS 不给最终 IP，只返回“去问哪个 .edu 的 TLD 服务器”
4.本地 DNS 再去问 .edu 的 TLD DNS 服务器
5.TLD DNS 返回“负责 umass.edu 的权威 DNS 服务器地址”（dns.cs.umass.edu）
6.本地 DNS 去问权威 DNS 服务器
7.权威 DNS 返回 gaia.cs.umass.edu 的最终 IP
8.本地 DNS 把最终 IP 回给请求主机

---

### DNS Records（资源记录）

DNS 数据库中的条目称为 **Resource Record（RR）**，资源记录。格式为：

```
RR format: (name, value, type, ttl)
```
name域名，type记录类型，value记录的值，ttl生存时间（Time To Live）

常见的type类型：
| type | name | value |
|------|------|-------|
| **A** | 主机名 | IPv4 地址 |
| **NS** | 域名 | 该域权威 DNS 服务器的主机名 |
| **CNAME** | 别名（alias） | 规范主机名（canonical name） |
| **MX** | 域名 | 该域邮件服务器的主机名 |

**例子**：
- `(networkutopia.com, dns1.networkutopia.com, NS, 32768)` — NS记录，指向权威服务器，说明`networkutopia.com`的DNS服务器是`dns1.networkutopia.com`
- `(dns1.networkutopia.com, 212.212.212.1, A, 5600)` — A记录，给出IP地址，说明`dns1.networkutopia.com`的IP地址是`212.212.212.1`

---

### DNS Protocol（DNS协议）

- **消息类型**：Query 和 Reply，**两者格式相同**
  - Header：identifier、flags 等
  - Body：resource records
- **传输层**：使用 **UDP，端口 53**（通过哪条路来传输消息）
  - 规范也支持 TCP，但不总是实现

---

### DNS 的高可用性（Reliability）

| 机制 | 说明 |
|------|------|
| **主/从复制（Replicated servers）** | 至少一个副本可用则服务不中断；副本间可负载均衡 |
| **UDP 查询** | 轻量，可靠性在应用层自行实现 |
| **超时重试（Alternate servers on timeout）** | 超时后尝试其他服务器；重试同一服务器时采用**指数退避（exponential backoff）** |
| **相同标识符（Same identifier）** | 不关心哪台副本响应，只要返回相同标识符即可 |

---

### DNS Caching（缓存）

- 每次完整查询可能需要**最多 1 秒**的延迟
- **缓存大幅降低开销**：
  - 顶级服务器地址几乎不变
  - 热门域名（如 `www.cnn.com`）被频繁访问，本地服务器通常已缓存
- **工作原理**：
  - DNS 服务器缓存收到的查询响应
  - 响应中含有 **TTL（Time To Live）**字段
  - TTL 到期后，服务器**删除缓存条目**（可能短暂不一致）

---

### Attacking DNS

**DDoS 攻击**：
- 2002年10月，攻击者用僵尸网络向 13 个根服务器发送大量 ICMP 报文
  - 攻击未奏效：大部分根服务器执行了**分组过滤**（ICMP报文被阻止）
  - 很多域名在本地缓存，可绕过根服务器直接解析
- 更有效的攻击：向**TLD服务器**发送大量 DNS 请求（近年来较常见）

**重定向攻击（Redirect attacks）**：
- **Man-in-the-middle**：拦截查询，返回伪造响应
- **DNS poisoning（DNS污染）**：向 DNS 服务器发送伪造响应，使其缓存错误记录
  - 解决办法之一：修改本地 hosts 文件

**利用 DNS 进行 DDoS 放大攻击（Exploit DNS for DDoS）**：
- 发送查询时伪造**源地址为攻击目标 IP**
- 利用 DNS 响应比查询大（放大效果），淹没目标

---

## Electronic Mail（电子邮件）

互联网上使用最广泛的应用之一。

### 邮件系统组件
3个主要组成部分

**User Agent（用户代理）**：
- 用于撰写、编辑、阅读邮件
- 例：Eudora、Outlook、Foxmail、Netscape Messenger
- 负责发出邮件和接收邮件，收发消息都存储在服务器上

**Mail Server（邮件服务器）**(Host-Server)：
- **Mailbox（邮箱）**：存储用户收到的邮件
- **Message queue（消息队列）**：存储待发出的邮件
- 服务器之间使用 **SMTP** 协议传输邮件

**简单邮件传输协议**（SMTP, Simple Mail Transfer Protocol）是邮件服务器之间传输邮件的应用层协议，使用TCP。

---

### SMTP
SMTP用于从发送方的邮件服务器发送报文到接收方的邮件服务器，SMTP是一个**文本协议**，使用TCP端口25。
SMTP实际上比HTTP问世的早得多，但是他限制所有报文的体部分必须是7-bit ASCII文本，因此SMTP不支持多媒体邮件。后来，MIME（Multipurpose Internet Mail Extensions）协议被引入来扩展SMTP，使其能够支持多媒体邮件。

SMTP一般不使用中间邮件服务器发送邮件。


### 邮件投递的三个阶段（3 Stages of Mail Delivery）

| 阶段 | 描述 | 协议 |
|------|------|------|
| **1st Stage** | 本地用户代理 → 本地 SMTP 服务器（UA作为SMTP client，本地服务器作为SMTP server） | SMTP |
| **2nd Stage** | 本地服务器中转 → 远程 SMTP 服务器（本地服务器变为SMTP client） | SMTP |
| **3rd Stage** | 远程用户代理 → 远程服务器上的邮箱（读取邮件） | POP3 / IMAP4 |

> 邮件格式由 **RFC 822** 或 **MIME** 定义；Stage 1/2 传输用 SMTP，Stage 3 读取用 POP3/IMAP4（或 HTTP）

---

### 邮件投递场景（A Mail Delivery Scenario）

以 Alice 发邮件给 Bob（`bob@someschool.edu`）为例：

1. Alice 用 UA 撰写邮件，填写收件人
2. Alice 的 UA 通过 SMTP 将邮件发到**本地邮件服务器**，放入**消息队列**
3. Alice 的邮件服务器（SMTP client 侧）与 Bob 的邮件服务器**建立 TCP 连接**
4. 通过 TCP 连接将邮件发送给 Bob 的邮件服务器
5. Bob 的邮件服务器将邮件放入 **Bob 的邮箱**
6. Bob 调用 UA，通过 **POP3** 等协议读取邮件

---

### SMTP Transaction（SMTP 事务）

**三个传输阶段**：握手（Handshaking）→ 传输数据 → 关闭连接

**命令/响应交互方式**：
- Commands：ASCII 文本
- Response：状态码 + 短语

```
S: 220 hamburger.edu
C: HELO crepes.fr
S: 250 Hello crepes.fr, pleased to meet you
C: MAIL FROM: <alice@crepes.fr>
S: 250 alice@crepes.fr … Sender ok
C: RCPT TO: <bob@hamburger.edu>
S: 250 bob@hamburger.edu … Recipient ok
C: RCPT TO: <Johm@hamburger.edu>
S: 550 No such user here
C: DATA
S: 354 Enter mail, end with "." on a line by itself
C: Do you like ketchup?
C: How about pickles?
C: .
S: 250 Message accepted for delivery
C: QUIT
S: 221 hamburger.edu closing connection
```

> 可以用 `telnet servername 25` 手动模拟 SMTP 交互，依次输入 HELO、MAIL FROM、RCPT TO、DATA、QUIT 命令。

