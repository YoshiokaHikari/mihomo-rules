# Mihomo 规则完全指南

> 📚 本文档详细介绍 Mihomo (Clash.Meta) 所有规则类型的写法和用法。
>
> 🔗 官方文档：https://wiki.metacubex.one/config/rules/

---

## 目录

- [规则总览](#规则总览)
- [域名规则](#域名规则)
- [IP 规则](#ip-规则)
- [网络规则](#网络规则)
- [进程规则](#进程规则)
- [逻辑规则](#逻辑规则)
- [规则集引用](#规则集引用)
- [GeoSite / GeoIP](#geosite--geoip)
- [特殊规则](#特殊规则)
- [Rule Provider 配置](#rule-provider-配置)
- [规则编写最佳实践](#规则编写最佳实践)
- [本仓库使用方法](#本仓库使用方法)
- [常见问题](#常见问题)

---

## 规则总览

| 规则类型 | 说明 | 语法 |
|---|---|---|
| `DOMAIN` | 精确匹配域名 | `DOMAIN,www.google.com,Policy` |
| `DOMAIN-SUFFIX` | 匹配域名后缀 | `DOMAIN-SUFFIX,google.com,Policy` |
| `DOMAIN-KEYWORD` | 域名包含关键词 | `DOMAIN-KEYWORD,google,Policy` |
| `DOMAIN-REGEX` | 正则匹配域名 | `DOMAIN-REGEX,.*\.google\.com$,Policy` |
| `IP-CIDR` | 匹配 IPv4 CIDR | `IP-CIDR,10.0.0.0/8,Policy` |
| `IP-CIDR6` | 匹配 IPv6 CIDR | `IP-CIDR6,2001:db8::/32,Policy` |
| `IP-ASN` | 匹配 ASN | `IP-ASN,AS4134,Policy` |
| `SRC-IP-CIDR` | 源 IP 匹配 | `SRC-IP-CIDR,192.168.1.0/24,Policy` |
| `SRC-PORT` | 源端口匹配 | `SRC-PORT,12345,Policy` |
| `DST-PORT` | 目标端口匹配 | `DST-PORT,443,Policy` |
| `PROCESS-NAME` | 进程名匹配 | `PROCESS-NAME,chrome,Policy` |
| `PROCESS-PATH` | 进程路径匹配 | `PROCESS-PATH,/usr/bin/curl,Policy` |
| `PROCESS-NAME-REGEX` | 正则匹配进程名 | `PROCESS-NAME-REGEX,^chrome$,Policy` |
| `PROCESS-PATH-REGEX` | 正则匹配进程路径 | `PROCESS-PATH-REGEX,/usr/bin/.*,Policy` |
| `NETWORK` | 网络类型匹配 | `NETWORK,tcp,Policy` |
| `IN-TYPE` | 入站类型匹配 | `IN-TYPE,HTTP,Policy` |
| `IN-NAME` | 入站名称匹配 | `IN-NAME,my-inbound,Policy` |
| `IN-USER` | 入站用户匹配 | `IN-USER,username,Policy` |
| `RULE-SET` | 引用规则集 | `RULE-SET,my-rules,Policy` |
| `GEOSITE` | 匹配 GeoSite 数据库 | `GEOSITE,google,Policy` |
| `GEOIP` | 匹配 GeoIP 数据库 | `GEOIP,CN,no-resolve` |
| `AND` | 逻辑与 | `AND,((DOMAIN-SUFFIX,google.com),(DST-PORT,443)),Policy` |
| `OR` | 逻辑或 | `OR,((DOMAIN-SUFFIX,google.com),(DOMAIN-SUFFIX,github.com)),Policy` |
| `NOT` | 逻辑非 | `NOT,((DOMAIN-SUFFIX,google.com)),Policy` |
| `SUB-RULE` | 子规则 | `SUB-RULE,(DST-PORT,443),my-sub-rule` |
| `MATCH` | 最终规则（兜底） | `MATCH,Policy` |

---

## 域名规则

### DOMAIN

精确匹配完整域名，**不匹配子域名**。

```yaml
rules:
  # 匹配 www.google.com，但不匹配 mail.google.com
  - DOMAIN,www.google.com,🚀 节点选择
  
  # 匹配 google.com，但不匹配 www.google.com
  - DOMAIN,google.com,🚀 节点选择
```

**适用场景**：
- 需要精确匹配某个特定域名
- 不想影响其他子域名

---

### DOMAIN-SUFFIX

匹配域名后缀，**包含所有子域名**。

```yaml
rules:
  # 匹配 google.com 及所有子域名（www.google.com, mail.google.com 等）
  - DOMAIN-SUFFIX,google.com,🚀 节点选择
  
  # 匹配 github.com 及所有子域名
  - DOMAIN-SUFFIX,github.com,🚀 节点选择
```

**适用场景**：
- 最常用的域名规则
- 匹配某个域名及其所有子域名

**注意**：
- `DOMAIN-SUFFIX,com` 会匹配所有 `.com` 结尾的域名！
- `DOMAIN-SUFFIX,google.com` 也会匹配 `notgoogle.com`（因为以 `google.com` 结尾）
- 如需精确匹配，使用 `DOMAIN`

---

### DOMAIN-KEYWORD

域名包含指定关键词即匹配。

```yaml
rules:
  # 匹配所有包含 "google" 的域名
  - DOMAIN-KEYWORD,google,🚀 节点选择
  
  # 匹配所有包含 "github" 的域名
  - DOMAIN-KEYWORD,github,🚀 节点选择
```

**适用场景**：
- 快速匹配多个相关域名
- 不需要精确控制时使用

**注意**：
- `DOMAIN-KEYWORD,google` 会匹配 `google.com`、`googleapis.com`、`notgoogle.com` 等
- 关键词匹配是大小写不敏感的

---

### DOMAIN-REGEX

使用正则表达式匹配域名。

```yaml
rules:
  # 匹配所有 .google.com 结尾的域名
  - DOMAIN-REGEX,.*\.google\.com$,🚀 节点选择
  
  # 匹配所有 .cn 结尾的域名
  - DOMAIN-REGEX,.*\.cn$,DIRECT
  
  # 匹配包含数字的域名
  - DOMAIN-REGEX,.*[0-9]+.*,DIRECT
```

**适用场景**：
- 需要复杂的匹配逻辑
- 其他规则类型无法满足需求时

**正则语法**：
- `.` 匹配任意字符
- `*` 匹配前一个字符 0 次或多次
- `+` 匹配前一个字符 1 次或多次
- `?` 匹配前一个字符 0 次或 1 次
- `^` 匹配开头
- `$` 匹配结尾
- `[]` 字符集合
- `()` 分组
- `|` 或

---

## IP 规则

### IP-CIDR

匹配 IPv4 地址或 CIDR 网段。

```yaml
rules:
  # 匹配单个 IP
  - IP-CIDR,114.114.114.114/32,DIRECT
  
  # 匹配 CIDR 网段
  - IP-CIDR,10.0.0.0/8,DIRECT
  - IP-CIDR,172.16.0.0/12,DIRECT
  - IP-CIDR,192.168.0.0/16,DIRECT
  
  # 匹配中国大陆 IP
  - IP-CIDR,1.0.1.0/24,DIRECT
  - IP-CIDR,1.0.2.0/23,DIRECT
```

**可选参数**：
- `no-resolve`：不进行 DNS 解析，只匹配原始 IP

```yaml
rules:
  # 不解析域名，只匹配 IP
  - IP-CIDR,10.0.0.0/8,DIRECT,no-resolve
```

---

### IP-CIDR6

匹配 IPv6 地址或 CIDR 网段。

```yaml
rules:
  # 匹配 IPv6 本地链路地址
  - IP-CIDR6,fe80::/10,DIRECT
  
  # 匹配 IPv6 私有地址
  - IP-CIDR6,fc00::/7,DIRECT
  
  # 匹配 IPv6 全局单播地址
  - IP-CIDR6,2000::/3,🚀 节点选择
```

---

### IP-ASN

匹配指定 ASN（自治系统号）的 IP 地址。

```yaml
rules:
  # 匹配中国电信 (AS4134)
  - IP-ASN,AS4134,DIRECT
  
  # 匹配中国联通 (AS4837)
  - IP-ASN,AS4837,DIRECT
  
  # 匹配中国移动 (AS9808)
  - IP-ASN,AS9808,DIRECT
  
  # 匹配 Google (AS15169)
  - IP-ASN,AS15169,🚀 节点选择
  
  # 匹配 Cloudflare (AS13335)
  - IP-ASN,AS13335,🚀 节点选择
```

**适用场景**：
- 按运营商分流
- 按 CDN 服务商分流

---

### SRC-IP-CIDR

匹配源 IP 地址（客户端 IP）。

```yaml
rules:
  # 匹配内网客户端
  - SRC-IP-CIDR,192.168.1.0/24,DIRECT
  
  # 匹配特定客户端
  - SRC-IP-CIDR,192.168.1.100/32,🚀 节点选择
```

**适用场景**：
- 按客户端 IP 分流
- 多用户场景下为不同用户设置不同策略

---

## 网络规则

### DST-PORT

匹配目标端口。

```yaml
rules:
  # 匹配 HTTP 端口
  - DST-PORT,80,DIRECT
  
  # 匹配 HTTPS 端口
  - DST-PORT,443,🚀 节点选择
  
  # 匹配 SSH 端口
  - DST-PORT,22,DIRECT
  
  # 匹配 DNS 端口
  - DST-PORT,53,DIRECT
  
  # 匹配端口范围（Mihomo 支持）
  - DST-PORT,8000-9000,DIRECT
```

---

### SRC-PORT

匹配源端口（客户端端口）。

```yaml
rules:
  # 匹配特定源端口
  - SRC-PORT,12345,DIRECT
```

**适用场景**：
- 较少使用
- 特殊网络配置时可能用到

---

### NETWORK

匹配网络类型。

```yaml
rules:
  # 匹配 TCP 流量
  - NETWORK,tcp,DIRECT
  
  # 匹配 UDP 流量
  - NETWORK,udp,🚀 节点选择
```

**适用场景**：
- TCP/UDP 分流
- 某些代理协议只支持 TCP 或 UDP

---

### IN-TYPE

匹配入站连接类型。

```yaml
rules:
  # 匹配 HTTP 入站
  - IN-TYPE,HTTP,DIRECT
  
  # 匹配 SOCKS5 入站
  - IN-TYPE,SOCKS5,🚀 节点选择
  
  # 匹配 HTTP 入站
  - IN-TYPE,HTTP,🚀 节点选择
  
  # 匹配 Mixed 入站
  - IN-TYPE,Mixed,🚀 节点选择
  
  # 匹配 TUN 入站
  - IN-TYPE,TUN,🚀 节点选择
  
  # 匹配 TProxy 入站
  - IN-TYPE,TProxy,🚀 节点选择
```

---

### IN-NAME

匹配入站连接名称。

```yaml
# 配置中的入站
listeners:
  - name: my-http
    type: http
    port: 7890
  
  - name: my-socks
    type: socks
    port: 7891

rules:
  # 匹配名为 my-http 的入站
  - IN-NAME,my-http,DIRECT
  
  # 匹配名为 my-socks 的入站
  - IN-NAME,my-socks,🚀 节点选择
```

---

### IN-USER

匹配入站用户认证。

```yaml
# 配置中的用户认证
listeners:
  - name: my-http
    type: http
    port: 7890
    users:
      - username: user1
        password: pass1
      - username: user2
        password: pass2

rules:
  # 匹配用户 user1
  - IN-USER,user1,🚀 节点选择
  
  # 匹配用户 user2
  - IN-USER,user2,DIRECT
```

---

## 进程规则

### PROCESS-NAME

匹配进程名称。

```yaml
rules:
  # 匹配 Chrome 浏览器
  - PROCESS-NAME,chrome,🚀 节点选择
  
  # 匹配 Firefox 浏览器
  - PROCESS-NAME,firefox,🚀 节点选择
  
  # 匹配 curl 命令
  - PROCESS-NAME,curl,DIRECT
  
  # 匹配 Telegram
  - PROCESS-NAME,Telegram,🚀 节点选择
  
  # Windows 进程需要包含 .exe
  - PROCESS-NAME,chrome.exe,🚀 节点选择
```

**适用场景**：
- 按应用程序分流
- 需要内核支持（TUN/TProxy 模式）

**注意**：
- 需要 root/Administrator 权限
- 需要 TUN 或 TProxy 入站
- Windows 进程名通常包含 `.exe`

---

### PROCESS-PATH

匹配进程完整路径。

```yaml
rules:
  # 匹配 Chrome 完整路径
  - PROCESS-PATH,/Applications/Google Chrome.app/Contents/MacOS/Google Chrome,🚀 节点选择
  
  # 匹配 Linux 下的 curl
  - PROCESS-PATH,/usr/bin/curl,DIRECT
  
  # 匹配 Windows 下的 Chrome
  - PROCESS-PATH,C:\Program Files\Google\Chrome\Application\chrome.exe,🚀 节点选择
```

**适用场景**：
- 需要区分同名进程的不同安装位置
- 比 PROCESS-NAME 更精确

---

### PROCESS-NAME-REGEX

使用正则表达式匹配进程名。

```yaml
rules:
  # 匹配所有 Chrome 相关进程
  - PROCESS-NAME-REGEX,^chrome.*$,🚀 节点选择
  
  # 匹配所有包含 "browser" 的进程
  - PROCESS-NAME-REGEX,.*browser.*,🚀 节点选择
```

---

### PROCESS-PATH-REGEX

使用正则表达式匹配进程路径。

```yaml
rules:
  # 匹配 /usr/bin/ 下的所有进程
  - PROCESS-PATH-REGEX,/usr/bin/.*,DIRECT
  
  # 匹配 Applications 目录下的进程
  - PROCESS-PATH-REGEX,/Applications/.*,🚀 节点选择
```

---

## 逻辑规则

### AND

逻辑与，所有条件都满足时匹配。

```yaml
rules:
  # 匹配 google.com 的 443 端口
  - AND,((DOMAIN-SUFFIX,google.com),(DST-PORT,443)),🚀 节点选择
  
  # 匹配内网客户端访问 HTTP
  - AND,((SRC-IP-CIDR,192.168.1.0/24),(DST-PORT,80)),DIRECT
  
  # 匹配 Chrome 访问 GitHub
  - AND,((PROCESS-NAME,chrome),(DOMAIN-SUFFIX,github.com)),🚀 节点选择
```

---

### OR

逻辑或，任一条件满足时匹配。

```yaml
rules:
  # 匹配 Google 或 GitHub
  - OR,((DOMAIN-SUFFIX,google.com),(DOMAIN-SUFFIX,github.com)),🚀 节点选择
  
  # 匹配 HTTP 或 HTTPS 端口
  - OR,((DST-PORT,80),(DST-PORT,443)),DIRECT
  
  # 匹配多个进程
  - OR,((PROCESS-NAME,chrome),(PROCESS-NAME,firefox)),🚀 节点选择
```

---

### NOT

逻辑非，条件不满足时匹配。

```yaml
rules:
  # 匹配非 .cn 结尾的域名
  - NOT,((DOMAIN-SUFFIX,.cn)),🚀 节点选择
  
  # 匹配非 80 端口
  - NOT,((DST-PORT,80)),🚀 节点选择
  
  # 匹配非 Chrome 进程
  - NOT,((PROCESS-NAME,chrome)),DIRECT
```

---

### 组合使用

```yaml
rules:
  # 匹配 Google 的非 80 端口
  - AND,((DOMAIN-SUFFIX,google.com),(NOT,((DST-PORT,80)))),🚀 节点选择
  
  # 匹配 (Google 或 GitHub) 的 443 端口
  - AND,((OR,((DOMAIN-SUFFIX,google.com),(DOMAIN-SUFFIX,github.com))),(DST-PORT,443)),🚀 节点选择
```

---

## 规则集引用

### RULE-SET

引用预定义的规则集。

```yaml
rule-providers:
  my-direct:
    type: http
    behavior: domain
    url: "https://example.com/direct.mrs"
    path: ./ruleset/direct.mrs
    interval: 86400

rules:
  # 引用规则集
  - RULE-SET,my-direct,DIRECT
```

---

### SUB-RULE

定义子规则，支持嵌套。

```yaml
sub-rules:
  my-sub-rule:
    - DOMAIN-SUFFIX,google.com,🚀 节点选择
    - DOMAIN-SUFFIX,github.com,🚀 节点选择
    - MATCH,DIRECT

rules:
  # 使用子规则
  - SUB-RULE,(DST-PORT,443),my-sub-rule
```

---

## GeoSite / GeoIP

### GEOSITE

匹配 GeoSite 数据库中的域名分类。

```yaml
rules:
  # 匹配 Google 相关域名
  - GEOSITE,google,🚀 节点选择
  
  # 匹配 GitHub
  - GEOSITE,github,🚀 节点选择
  
  # 匹配 Telegram
  - GEOSITE,telegram,🚀 节点选择
  
  # 匹配中国大陆域名
  - GEOSITE,cn,DIRECT
  
  # 匹配私有域名（局域网等）
  - GEOSITE,private,DIRECT
  
  # 匹配 Google（使用 @cn 标记，表示 Google 中国）
  - GEOSITE,google@cn,DIRECT
```

**常用分类**：
- `google` - Google 相关
- `github` - GitHub
- `telegram` - Telegram
- `twitter` - Twitter
- `facebook` - Facebook
- `youtube` - YouTube
- `netflix` - Netflix
- `cn` - 中国大陆
- `private` - 私有网络
- `category-ads-all` - 广告

---

### GEOIP

匹配 GeoIP 数据库中的 IP 分类。

```yaml
rules:
  # 匹配中国大陆 IP
  - GEOIP,CN,DIRECT
  
  # 匹配私有 IP（局域网）
  - GEOIP,LAN,DIRECT
  
  # 匹配 Google IP
  - GEOIP,Google,🚀 节点选择
  
  # 匹配 Cloudflare IP
  - GEOIP,Cloudflare,🚀 节点选择
```

**可选参数**：
- `no-resolve`：不进行 DNS 解析

```yaml
rules:
  # 不解析域名，只匹配 IP
  - GEOIP,CN,DIRECT,no-resolve
```

**常用分类**：
- `CN` - 中国大陆
- `LAN` - 局域网
- `Google` - Google
- `Cloudflare` - Cloudflare
- `Amazon` - Amazon AWS
- `Microsoft` - Microsoft Azure

---

## 特殊规则

### MATCH

最终规则，匹配所有未被其他规则匹配的流量。

```yaml
rules:
  # 其他规则...
  
  # 最终规则：未匹配的流量走代理
  - MATCH,🚀 节点选择
  
  # 或者：未匹配的流量直连
  - MATCH,DIRECT
```

**注意**：
- `MATCH` 必须放在所有规则的**最后**
- 每个配置只能有一个 `MATCH` 规则

---

## Rule Provider 配置

### 基本配置

```yaml
rule-providers:
  # 规则集名称
  my-rules:
    # 类型：http（远程）或 file（本地）
    type: http
    
    # 行为：domain、ipcidr 或 classical
    behavior: domain
    
    # 远程 URL（type: http 时必填）
    url: "https://example.com/rules.mrs"
    
    # 本地路径（type: file 时必填，type: http 时为缓存路径）
    path: ./ruleset/my-rules.mrs
    
    # 更新间隔（秒），type: http 时有效
    interval: 86400
```

---

### 行为类型

#### `domain` 行为

适用于域名规则文件，支持以下格式：

```text
# 每行一个规则
example.com          # 转换为 DOMAIN-SUFFIX,example.com
full:example.com     # 转换为 DOMAIN,example.com
keyword:example      # 转换为 DOMAIN-KEYWORD,example
regexp:.*\.com$      # 转换为 DOMAIN-REGEX,.*\.com$
```

#### `ipcidr` 行为

适用于 IP CIDR 规则文件：

```text
# 每行一个 CIDR
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
2001:db8::/32
```

#### `classical` 行为

适用于完整规则语法的 YAML 文件：

```yaml
payload:
  - DOMAIN,www.google.com
  - DOMAIN-SUFFIX,google.com
  - DOMAIN-KEYWORD,google
  - IP-CIDR,10.0.0.0/8
  - GEOIP,CN
  - PROCESS-NAME,chrome
  - DST-PORT,443
```

---

### 格式类型

#### MRS 格式（推荐）

二进制格式，性能最佳：

```yaml
rule-providers:
  my-rules:
    type: http
    behavior: domain
    url: "https://example.com/rules.mrs"
    path: ./ruleset/my-rules.mrs
    interval: 86400
```

#### Text 格式

纯文本格式，可读性好：

```yaml
rule-providers:
  my-rules:
    type: http
    behavior: domain
    format: text
    url: "https://example.com/rules.txt"
    path: ./ruleset/my-rules.txt
    interval: 86400
```

#### YAML 格式

YAML 格式，支持 classical 行为：

```yaml
rule-providers:
  my-rules:
    type: http
    behavior: classical
    url: "https://example.com/rules.yaml"
    path: ./ruleset/my-rules.yaml
    interval: 86400
```

---

### 本地文件

使用本地文件作为规则源：

```yaml
rule-providers:
  my-rules:
    type: file
    behavior: domain
    path: ./my-rules.yaml
```

---

## 规则编写最佳实践

### 1. 规则顺序很重要

Mihomo 按顺序匹配规则，**先匹配到的规则生效**：

```yaml
rules:
  # ❌ 错误：宽泛规则在前
  - DOMAIN-SUFFIX,com,🚀 节点选择
  - DOMAIN,www.specific.com,DIRECT  # 永远不会生效！

  # ✅ 正确：精确规则在前
  - DOMAIN,www.specific.com,DIRECT
  - DOMAIN-SUFFIX,com,🚀 节点选择
```

### 2. 使用 RULE-SET 减少重复

```yaml
# ❌ 冗长的配置
rules:
  - DOMAIN-SUFFIX,google.com,🚀 节点选择
  - DOMAIN-SUFFIX,googleapis.com,🚀 节点选择
  - DOMAIN-SUFFIX,gstatic.com,🚀 节点选择
  # ... 更多

# ✅ 使用 RULE-SET
rule-providers:
  google:
    type: http
    behavior: domain
    url: "https://..."

rules:
  - RULE-SET,google,🚀 节点选择
```

### 3. 善用 GEOSITE 和 GEOIP

```yaml
rules:
  # 使用内置数据库，无需自己维护
  - GEOSITE,google,🚀 节点选择
  - GEOSITE,cn,DIRECT
  - GEOIP,CN,DIRECT,no-resolve
```

### 4. no-resolve 的使用

```yaml
rules:
  # 匹配 IP 规则时，建议加上 no-resolve
  # 避免不必要的 DNS 解析
  - GEOIP,CN,DIRECT,no-resolve
  - IP-CIDR,10.0.0.0/8,DIRECT,no-resolve
```

### 5. 逻辑规则组合

```yaml
rules:
  # 复杂条件匹配
  - AND,((DOMAIN-SUFFIX,google.com),(DST-PORT,443)),🚀 节点选择
  - OR,((DOMAIN-SUFFIX,google.com),(DOMAIN-SUFFIX,github.com)),🚀 节点选择
  - NOT,((DOMAIN-KEYWORD,local)),🚀 节点选择
```

---

## 本仓库使用方法

### 仓库结构

```
mihomo-rules/
├── .github/workflows/
│   └── build.yml          # 自动构建脚本
├── rules/
│   ├── domain/            # 域名规则
│   │   ├── my-direct.txt  # 直连域名
│   │   ├── my-proxy.txt   # 代理域名
│   │   └── ...
│   └── ip/                # IP 规则
│       ├── my-direct.txt  # 直连 IP
│       └── ...
├── MRS_GUIDE.md           # 本文档
└── README.md              # 仓库说明
```

### 添加域名规则

在 `rules/domain/` 下创建 `.txt` 文件：

```text
# my-rules.txt
# 注释行以 # 开头，空行会被忽略

# 直连域名
baidu.com
taobao.com
bilibili.com

# 精确匹配
full:www.specific.com

# 关键词匹配
keyword:internal

# 正则匹配
regexp:.*\.local$
```

### 添加 IP 规则

在 `rules/ip/` 下创建 `.txt` 文件：

```text
# my-ip.txt
# 注释行以 # 开头，空行会被忽略

# CIDR 网段
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16

# 单个 IP
114.114.114.114/32
```

### 自动构建

推送到 `main` 分支后，GitHub Actions 会自动：

1. ✅ 读取 `rules/domain/` 和 `rules/ip/` 下的规则文件
2. ✅ 构建 `geosite.dat`
3. ✅ 转换为 `.mrs` 格式
4. ✅ 生成 Text/YAML 格式
5. ✅ 发布到 `release` 分支和 GitHub Release

### 在 Mihomo 中使用

```yaml
rule-providers:
  my-direct:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/mrs/domain/my-direct.mrs"
    path: ./ruleset/my-direct.mrs
    interval: 86400

  my-proxy:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/mrs/domain/my-proxy.mrs"
    path: ./ruleset/my-proxy.mrs
    interval: 86400

rules:
  - RULE-SET,my-direct,DIRECT
  - RULE-SET,my-proxy,🚀 节点选择
```

---

## 常见问题

### Q: MRS 和 Text 格式有什么区别？

| 特性 | MRS | Text |
|---|---|---|
| 加载速度 | ⚡ 快 | 🐢 慢 |
| 内存占用 | 💾 低 | 💾 高 |
| 可读性 | ❌ 二进制 | ✅ 文本 |
| 文件大小 | 📦 小 | 📦 大 |

**推荐使用 MRS 格式**，特别是规则数量较多时。

### Q: `behavior` 应该选哪个？

| 场景 | 推荐 behavior |
|---|---|
| 只有域名规则 | `domain` |
| 只有 IP/CIDR 规则 | `ipcidr` |
| 混合规则（域名+IP+进程等） | `classical` |

### Q: DOMAIN 和 DOMAIN-SUFFIX 有什么区别？

- `DOMAIN,www.example.com` 只匹配 `www.example.com`
- `DOMAIN-SUFFIX,example.com` 匹配 `example.com` 及所有子域名（`www.example.com`、`mail.example.com` 等）

### Q: 为什么我的规则不生效？

1. **检查规则顺序**：精确规则是否在宽泛规则之前
2. **检查语法**：规则类型和参数是否正确
3. **检查 MATCH**：是否有 MATCH 规则在最后
4. **查看日志**：启用 `log-level: debug` 查看匹配情况

### Q: 如何调试规则？

```yaml
# 启用调试日志
log-level: debug
```

使用 External Controller 查看：

```bash
# 查看规则列表
curl http://127.0.0.1:9090/rules

# 查看连接信息
curl http://127.0.0.1:9090/connections
```

### Q: 如何更新规则集？

- **自动更新**：设置 `interval` 参数（单位：秒）
- **手动更新**：删除 `path` 指定的本地文件，重启 Mihomo

---

## 参考资料

- [Mihomo 官方文档](https://wiki.metacubex.one/)
- [MetaCubeX GitHub](https://github.com/MetaCubeX/mihomo)
- [规则配置文档](https://wiki.metacubex.one/config/rules/)
- [Rule Provider 配置](https://wiki.metacubex.one/config/rule-providers/)
- [GeoSite 数据](https://github.com/MetaCubeX/meta-rules-dat)
- [GeoIP 数据](https://github.com/Loyalsoldier/geoip)

---

## 📄 License

MIT
