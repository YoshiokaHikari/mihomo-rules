# 我的 Mihomo 规则集

自动构建、定时更新的 Mihomo 规则集。

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
payload:
  - DOMAIN-SUFFIX,google.com
  - DOMAIN-SUFFIX,github.com
  - DOMAIN,www.specific.com
  - DOMAIN-KEYWORD,google
  - IP-CIDR,10.0.0.0/8
```

## 🚀 在 Mihomo 中使用

```yaml
rule-providers:
  direct:
    type: http
    behavior: classical
    url: "https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/rules/domain/direct.yaml"
    path: ./ruleset/direct.yaml
    interval: 86400

  proxy:
    type: http
    behavior: classical
    url: "https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/rules/domain/proxy.yaml"
    path: ./ruleset/proxy.yaml
    interval: 86400

  direct-ip:
    type: http
    behavior: classical
    url: "https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/rules/ip/direct.yaml"
    path: ./ruleset/direct-ip.yaml
    interval: 86400

rules:
  - RULE-SET,direct,DIRECT
  - RULE-SET,proxy,🚀 节点选择
  - RULE-SET,direct-ip,DIRECT
```

## 🔗 CDN 地址

```
https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/rules/domain/规则名.yaml
https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/rules/ip/规则名.yaml
```

## 💡 关于 MRS 格式

MRS (Meta Rule Set) 是 Mihomo 的二进制规则格式，加载更快、内存更低。

但生成 MRS 需要使用 `domain-list-community` 文本格式，而不是 classical YAML 格式。

如果你想要 MRS 格式，可以把规则改成这种写法：

```text
# rules/domain/proxy.txt（文本格式）
google.com
github.com
full:www.specific.com
keyword:google
```

然后告诉我，我会更新 workflow 来生成 MRS 文件。

## ⚙️ 自动构建

推送到 `main` 分支且修改了 `rules/` 目录时，GitHub Actions 会自动发布到 `release` 分支。

## 📄 License

MIT
