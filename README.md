# 我的 Mihomo 规则集

自动构建、定时更新的 Mihomo 规则集，支持 `.mrs` 格式。

> 📚 详细使用说明请查看 [MRS_GUIDE.md](./MRS_GUIDE.md)

## 📁 规则文件

```
rules/
├── domain/          # 域名规则
│   ├── direct.yaml  # 直连域名
│   └── proxy.yaml   # 代理域名
└── ip/              # IP 规则
    └── direct.yaml  # 直连 IP
```

## 📝 规则格式

使用标准的 **Mihomo classical 格式**：

```yaml
# rules/domain/proxy.yaml
payload:
  - DOMAIN-SUFFIX,google.com
  - DOMAIN-SUFFIX,github.com
  - DOMAIN,www.specific.com
  - DOMAIN-KEYWORD,google
```

## 🚀 在 Mihomo 中使用

### 方式一：MRS 格式（推荐，性能更好）

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

### 方式二：YAML 格式

```yaml
rule-providers:
  direct:
    type: http
    behavior: classical
    url: "https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/rules/domain/direct.yaml"
    path: ./ruleset/direct.yaml
    interval: 86400

rules:
  - RULE-SET,direct,DIRECT
```

## 🔗 CDN 地址

```
# MRS 格式
https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/rules/domain/规则名.mrs

# YAML 格式
https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/rules/domain/规则名.yaml
```

## ⚙️ 自动构建

推送到 `main` 分支且修改了 `rules/` 目录时，GitHub Actions 会自动：

1. ✅ 解析 YAML 规则
2. ✅ 生成 MRS 文件（domain 规则）
3. ✅ 发布到 `release` 分支
4. ✅ 清除 jsDelivr CDN 缓存

## 📄 License

MIT
