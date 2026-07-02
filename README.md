# 我的 Mihomo 规则集

自动构建、定时更新的 Mihomo 规则集，支持 `.mrs` 格式。

## 📁 目录结构

```
.
├── .github/workflows/
│   └── build.yml          # GitHub Actions 自动构建
├── rules/
│   ├── domain/            # 域名规则
│   │   ├── my-direct.txt  # 直连域名
│   │   └── my-proxy.txt   # 代理域名
│   └── ip/                # IP 规则
│       └── my-direct.txt  # 直连 IP
├── README.md
└── .gitignore
```

## 📝 规则格式

### 域名规则 (`rules/domain/*.txt`)

```text
# 注释行以 # 开头，空行会被忽略
example.com          # DOMAIN-SUFFIX (匹配 example.com 及所有子域名)
full:example.com     # DOMAIN (精确匹配)
keyword:example      # DOMAIN-KEYWORD (关键词匹配)
regexp:.*\.example\.com$  # DOMAIN-REGEX (正则匹配)
```

### IP 规则 (`rules/ip/*.txt`)

```text
# 注释行以 # 开头，空行会被忽略
114.114.114.114/32   # 单个 IP
10.0.0.0/8           # CIDR 网段
```

## 🚀 使用方法

### 1. Fork 或创建仓库

```bash
# 方式一：Fork 本仓库
# 方式二：创建新仓库，将文件复制进去
```

### 2. 添加规则文件

在 `rules/domain/` 和 `rules/ip/` 目录下创建 `.txt` 文件。

### 3. 推送到 main 分支

```bash
git add -A
git commit -m "Update rules"
git push origin main
```

GitHub Actions 会自动构建并发布到 `release` 分支。

### 4. 在 Mihomo 中使用

#### 方式一：MRS 格式（推荐，性能最佳）

```yaml
rule-providers:
  my-direct:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/你的用户名/你的仓库名@release/mrs/domain/my-direct.mrs"
    path: ./ruleset/my-direct.mrs
    interval: 86400

  my-proxy:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/你的用户名/你的仓库名@release/mrs/domain/my-proxy.mrs"
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
    url: "https://cdn.jsdelivr.net/gh/你的用户名/你的仓库名@release/classical/my-direct.list"
    path: ./ruleset/my-direct.list
    interval: 86400

rules:
  - RULE-SET,my-direct,DIRECT
```

#### 方式三：YAML 格式（classical）

```yaml
rule-providers:
  my-direct:
    type: http
    behavior: classical
    url: "https://cdn.jsdelivr.net/gh/你的用户名/你的仓库名@release/classical/my-direct.yaml"
    path: ./ruleset/my-direct.yaml
    interval: 86400

rules:
  - RULE-SET,my-direct,DIRECT
```

## ⚙️ 自动构建流程

每次推送到 `main` 分支且修改了 `rules/` 目录时，GitHub Actions 会自动：

1. ✅ 检出 `Loyalsoldier/domain-list-custom` 构建工具
2. ✅ 检出 `v2fly/domain-list-community` 基础数据
3. ✅ 将你的规则文件复制到构建目录
4. ✅ 构建 `geosite.dat`
5. ✅ 使用 `MetaCubeX/meta-rules-converter` 转换为 `.mrs` 格式
6. ✅ 同时生成 classical YAML 和纯文本格式
7. ✅ 发布到 `release` 分支和 GitHub Release
8. ✅ 清除 jsDelivr CDN 缓存

## 📦 产出物

| 文件 | 说明 | 用途 |
|---|---|---|
| `mrs/domain/*.mrs` | Mihomo 二进制规则格式 | `behavior: domain` + MRS URL |
| `classical/*.yaml` | Clash classical YAML 格式 | `behavior: classical` |
| `classical/*.list` | 纯文本格式 | `behavior: domain` + `format: text` |
| `ipcidr/*.list` | IP CIDR 列表 | `behavior: ipcidr` + `format: text` |
| `geosite.dat` | V2Ray geosite 格式 | 通用兼容 |

## 🔗 CDN 地址格式

```
# GitHub Raw
https://raw.githubusercontent.com/用户名/仓库名/release/mrs/domain/规则名.mrs

# jsDelivr CDN（推荐，国内可访问）
https://cdn.jsdelivr.net/gh/用户名/仓库名@release/mrs/domain/规则名.mrs
```

## 📋 添加新规则

1. 在 `rules/domain/` 或 `rules/ip/` 下创建新的 `.txt` 文件
2. 按格式添加规则
3. 推送到 `main` 分支
4. 等待 GitHub Actions 自动构建
5. 在 Mihomo 配置中引用新的 rule-provider

## 📄 License

MIT
