# 我的 Mihomo 规则集

自动构建、定时更新的 Mihomo 规则集。

> 📚 详细使用说明请查看 [MRS_GUIDE.md](./MRS_GUIDE.md)

## 📁 规则文件

| 文件 | 说明 | 用途 |
|---|---|---|
| `rules/domain/*.yaml` | 域名规则 | 直连/代理域名 |
| `rules/ip/*.yaml` | IP 规则 | 直连/代理 IP |

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

```yaml
# rules/ip/direct.yaml
payload:
  - IP-CIDR,10.0.0.0/8
  - IP-CIDR,192.168.0.0/16
```

## 🚀 快速使用

### 1. 添加规则

在 `rules/domain/` 或 `rules/ip/` 下创建 `.yaml` 文件。

### 2. 推送到 GitHub

```bash
git add -A
git commit -m "Update rules"
git push origin main
```

GitHub Actions 会自动发布到 `release` 分支。

### 3. 在 Mihomo 中使用

```yaml
rule-providers:
  proxy:
    type: http
    behavior: classical
    url: "https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/classical/proxy.yaml"
    path: ./ruleset/proxy.yaml
    interval: 86400

  direct:
    type: http
    behavior: classical
    url: "https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/classical/direct.yaml"
    path: ./ruleset/direct.yaml
    interval: 86400

rules:
  - RULE-SET,direct,DIRECT
  - RULE-SET,proxy,🚀 节点选择
```

## 🔗 CDN 地址

```
# jsDelivr CDN（推荐）
https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/classical/规则名.yaml

# GitHub Raw
https://raw.githubusercontent.com/YoshiokaHikari/mihomo-rules/release/classical/规则名.yaml
```

## ⚙️ 自动构建

推送到 `main` 分支且修改了 `rules/` 目录时，GitHub Actions 会自动：

1. ✅ 复制规则文件到 `release` 分支
2. ✅ 发布 GitHub Release
3. ✅ 清除 jsDelivr CDN 缓存

## 📄 License

MIT
