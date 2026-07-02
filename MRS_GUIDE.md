# Mihomo 规则集完全指南

> 📚 本文档详细介绍 Mihomo (Clash.Meta) 规则集的格式、用法和最佳实践。

## 目录

- [规则类型](#规则类型)
- [Rule Provider 格式](#rule-provider-格式)
- [Rule Provider 行为](#rule-provider-行为)
- [本仓库使用方法](#本仓库使用方法)
- [规则编写技巧](#规则编写技巧)
- [常见问题](#常见问题)

---

## 规则类型

Mihomo 支持以下规则类型：

### 域名规则

| 规则类型 | 说明 | 示例 |
|---|---|---|
| `DOMAIN` | 精确匹配域名 | `DOMAIN,www.google.com` |
| `DOMAIN-SUFFIX` | 匹配域名后缀 | `DOMAIN-SUFFIX,google.com` |
| `DOMAIN-KEYWORD` | 域名包含关键词 | `DOMAIN-KEYWORD,google` |
| `DOMAIN-REGEX` | 正则匹配域名 | `DOMAIN-REGEX,.*\.google\.com$` |

### IP 规则

| 规则类型 | 说明 | 示例 |
|---|---|---|
| `IP-CIDR` | 匹配 IPv4 CIDR | `IP-CIDR,10.0.0.0/8` |
| `IP-CIDR6` | 匹配 IPv6 CIDR | `IP-CIDR6,2001:db8::/32` |
| `GEOIP` | 匹配 GeoIP 数据库 | `GEOIP,CN` |
| `IP-ASN` | 匹配 ASN | `IP-ASN,AS4134` |

### 其他规则

| 规则类型 | 说明 | 示例 |
|---|---|---|
| `SRC-IP-CIDR` | 源 IP 匹配 | `SRC-IP-CIDR,192.168.1.0/24` |
| `SRC-PORT` | 源端口匹配 | `SRC-PORT,12345` |
| `DST-PORT` | 目标端口匹配 | `DST-PORT,443` |
| `PROCESS-NAME` | 进程名匹配 | `PROCESS-NAME,chrome` |
| `PROCESS-PATH` | 进程路径匹配 | `PROCESS-PATH,/usr/bin/curl` |
| `RULE-SET` | 引用规则集 | `RULE-SET,my-proxy` |
| `GEOSITE` | 匹配 GeoSite 数据库 | `GEOSITE,google` |
| `AND` | 逻辑与 | `AND,((DOMAIN-SUFFIX,google.com),(DST-PORT,443))` |
| `OR` | 逻辑或 | `OR,((DOMAIN-SUFFIX,google.com),(DOMAIN-SUFFIX,github.com))` |
| `NOT` | 逻辑非 | `NOT,((DOMAIN-SUFFIX,google.com))` |
| `IN-TYPE` | 入站类型 | `IN-TYPE,HTTP` |
| `NETWORK` | 网络类型 | `NETWORK,tcp` 或 `NETWORK,udp` |

---

## Rule Provider 格式

Rule Provider 支持三种格式：

### 1. YAML 格式 (`behavior: classical`)

```yaml
payload:
  - DOMAIN,www.google.com
  - DOMAIN-SUFFIX,google.com
  - DOMAIN-KEYWORD,google
  - IP-CIDR,10.0.0.0/8
  - GEOIP,CN
```

### 2. Text 格式 (`behavior: domain` 或 `behavior: ipcidr`)

**Domain 格式：**
```text
# 域名列表，每行一个
google.com
github.com
full:www.exact.com
keyword:example
regexp:.*\.test\.com$
```

**IP CIDR 格式：**
```text
# IP CIDR 列表，每行一个
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

### 3. MRS 格式 (`behavior: domain` 或 `behavior: ipcidr`)

MRS (Meta Rule Set) 是 Mihomo 的二进制规则格式：
- ✅ 加载速度更快
- ✅ 内存占用更低
- ✅ 支持增量更新

---

## Rule Provider 行为

### `domain` 行为

适用于域名规则，支持以下前缀：

| 前缀 | 转换为 | 说明 |
|---|---|---|
| (无) | `DOMAIN-SUFFIX` | 匹配域名及子域名 |
| `full:` | `DOMAIN` | 精确匹配 |
| `keyword:` | `DOMAIN-KEYWORD` | 关键词匹配 |
| `regexp:` | `DOMAIN-REGEX` | 正则匹配 |

**示例：**
```text
google.com          → DOMAIN-SUFFIX,google.com
full:www.google.com → DOMAIN,www.google.com
keyword:google      → DOMAIN-KEYWORD,google
regexp:.*\.com$     → DOMAIN-REGEX,.*\.com$
```

### `ipcidr` 行为

适用于 IP CIDR 规则：

```text
10.0.0.0/8          → IP-CIDR,10.0.0.0/8
2001:db8::/32       → IP-CIDR6,2001:db8::/32
```

### `classical` 行为

适用于完整规则语法（YAML 格式）：

```yaml
payload:
  - DOMAIN,www.google.com
  - DOMAIN-SUFFIX,google.com
  - DOMAIN-KEYWORD,google
  - IP-CIDR,10.0.0.0/8
  - GEOIP,CN
  - PROCESS-NAME,chrome
```

---

## 本仓库使用方法

### 仓库结构

```
mihomo-rules/
├── .github/workflows/
│   └── build.yml          # 自动构建脚本
├── rules/
│   ├── domain/            # 域名规则文件
│   │   ├── my-direct.txt  # 直连域名
│   │   └── my-proxy.txt   # 代理域名
│   └── ip/                # IP 规则文件
│       └── my-direct.txt  # 直连 IP
├── MRS_GUIDE.md           # 本文档
└── README.md              # 仓库说明
```

### 规则文件格式

在 `rules/domain/` 目录下创建 `.txt` 文件：

```text
# 注释行以 # 开头
# 空行会被忽略

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

### 自动构建

推送到 `main` 分支后，GitHub Actions 会自动：

1. 读取 `rules/domain/` 和 `rules/ip/` 下的规则文件
2. 构建 `geosite.dat`
3. 转换为 `.mrs` 格式
4. 发布到 `release` 分支和 GitHub Release

### 在 Mihomo 中引用

#### 方式一：MRS 格式（推荐）

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

#### 方式二：Text 格式

```yaml
rule-providers:
  my-direct:
    type: http
    behavior: domain
    format: text
    url: "https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/classical/my-direct.list"
    path: ./ruleset/my-direct.list
    interval: 86400

rules:
  - RULE-SET,my-direct,DIRECT
```

#### 方式三：YAML 格式

```yaml
rule-providers:
  my-direct:
    type: http
    behavior: classical
    url: "https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/classical/my-direct.yaml"
    path: ./ruleset/my-direct.yaml
    interval: 86400

rules:
  - RULE-SET,my-direct,DIRECT
```

---

## 规则编写技巧

### 1. 规则顺序很重要

Mihomo 按顺序匹配规则，**先匹配到的规则生效**：

```yaml
rules:
  # ❌ 错误：宽泛规则在前，后面的精确规则永远不会生效
  - DOMAIN-SUFFIX,com,🚀 节点选择
  - DOMAIN,www.specific.com,DIRECT

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
  # ... 更多 Google 域名

# ✅ 使用 RULE-SET
rule-providers:
  google:
    type: http
    behavior: domain
    url: "https://..."
    path: ./ruleset/google.mrs

rules:
  - RULE-SET,google,🚀 节点选择
```

### 3. 善用 GEOSITE 和 GEOIP

Mihomo 内置了 GeoSite 和 GeoIP 数据库：

```yaml
rules:
  # 匹配 Google 相关域名
  - GEOSITE,google,🚀 节点选择

  # 匹配中国大陆域名
  - GEOSITE,cn,DIRECT

  # 匹配中国大陆 IP
  - GEOIP,CN,DIRECT
```

### 4. 逻辑规则组合

```yaml
rules:
  # 匹配 Google 的 443 端口
  - AND,((DOMAIN-SUFFIX,google.com),(DST-PORT,443)),🚀 节点选择

  # 匹配 Google 或 GitHub
  - OR,((DOMAIN-SUFFIX,google.com),(DOMAIN-SUFFIX,github.com)),🚀 节点选择

  # 排除特定域名
  - AND,((DOMAIN-SUFFIX,com),(NOT,((DOMAIN-KEYWORD,local)))),🚀 节点选择
```

### 5. 使用 FINAL 规则

```yaml
rules:
  # ... 其他规则

  # 最终规则：未匹配的流量走代理
  - MATCH,🚀 节点选择
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

### Q: 如何调试规则匹配？

在 Mihomo 配置中启用日志：

```yaml
log-level: debug
```

或者使用 External Controller 查看：

```bash
# 查看当前规则
curl http://127.0.0.1:9090/rules

# 查看连接信息
curl http://127.0.0.1:9090/connections
```

### Q: 如何更新规则？

- **自动更新**：设置 `interval` 参数（单位：秒）
- **手动更新**：删除 `path` 指定的本地文件，重启 Mihomo

### Q: 规则文件放在哪里？

默认路径是 Mihomo 工作目录下的 `ruleset/` 文件夹：

```yaml
rule-providers:
  my-rule:
    type: http
    behavior: domain
    url: "https://..."
    path: ./ruleset/my-rule.mrs  # 相对于 Mihomo 工作目录
```

### Q: 如何使用本地规则文件？

```yaml
rule-providers:
  my-rule:
    type: file
    behavior: domain
    path: ./my-rules.yaml  # 本地文件路径
```

### Q: `behavior` 应该选哪个？

| 场景 | 推荐 behavior |
|---|---|
| 只有域名规则 | `domain` |
| 只有 IP/CIDR 规则 | `ipcidr` |
| 混合规则（域名+IP+进程等） | `classical` |

### Q: 如何查看规则集内容？

```bash
# 查看 MRS 文件信息（需要 mihomo CLI）
mihomo convert-ruleset mrs domain my-rule.mrs

# 查看 Text 文件
cat my-rule.list

# 查看 YAML 文件
cat my-rule.yaml
```

---

## 参考资料

- [Mihomo 官方文档](https://wiki.metacubex.one/)
- [MetaCubeX GitHub](https://github.com/MetaCubeX/mihomo)
- [Rule Provider 配置](https://wiki.metacubex.one/en/config/rule-providers/)
- [GeoSite 数据](https://github.com/MetaCubeX/meta-rules-dat)
- [GeoIP 数据](https://github.com/Loyalsoldier/geoip)

---

## 📄 License

MIT
