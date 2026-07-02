# 我的 Mihomo 规则集

自动构建、定时更新的 Mihomo 规则集，支持 `.list` 和 `.mrs` 格式。

> 📚 详细使用说明请查看 [MRS_GUIDE.md](./MRS_GUIDE.md)

## 📁 规则文件

```
rules/
├── domain/          # 域名规则
│   ├── direct.list  # 直连域名
│   └── proxy.list   # 代理域名
└── ip/              # IP 规则
    └── direct.list  # 直连 IP
```

## 📝 规则格式（behavior: domain）

```text
# rules/domain/proxy.list

google.com          # DOMAIN-SUFFIX（匹配子域名）
.github.com         # . 前缀 = DOMAIN-SUFFIX
+.bilibili.com      # +. 前缀 = DOMAIN-SUFFIX
full:example.com    # DOMAIN（精确匹配）
keyword:google      # DOMAIN-KEYWORD（关键词）
regexp:.*\.cn$      # DOMAIN-REGEX（正则）
```

## 🚀 在 Mihomo 中使用

### 方式一：MRS 格式（推荐，性能最好）

```yaml
rule-providers:
  direct:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/rules/domain/direct.mrs"
    path: ./ruleset/direct.mrs
    interval: 86400

  proxy:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/rules/domain/proxy.mrs"
    path: ./ruleset/proxy.mrs
    interval: 86400

rules:
  - RULE-SET,direct,DIRECT
  - RULE-SET,proxy,🚀 节点选择
```

### 方式二：LIST 格式

```yaml
rule-providers:
  direct:
    type: http
    behavior: domain
    format: text
    url: "https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/rules/domain/direct.list"
    path: ./ruleset/direct.list
    interval: 86400

rules:
  - RULE-SET,direct,DIRECT
```

## 🔗 CDN 地址

```
# MRS 格式（推荐）
https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/rules/domain/direct.mrs

# LIST 格式
https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/rules/domain/direct.list
```

## ⚙️ 自动构建

推送到 `main` 分支且修改了 `rules/` 目录时，GitHub Actions 会自动：

1. ✅ 读取 `.list` 规则文件
2. ✅ 生成 `geosite.dat`
3. ✅ 转换为 `.mrs` 格式
4. ✅ 发布到 `release` 分支

## 📄 License

MIT
