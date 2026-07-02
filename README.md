# 我的 Mihomo 规则集

自动构建、定时更新的 Mihomo 规则集，支持 `.mrs` 格式。

> 📚 详细使用说明请查看 [MRS_GUIDE.md](./MRS_GUIDE.md)

## 📁 规则文件

| 文件 | 说明 | 用途 |
|---|---|---|
| `direct.yaml` | 直连域名 | 国内网站直连 |
| `proxy.yaml` | 代理域名 | 需要代理的网站 |

## 📝 规则格式

使用标准的 **Mihomo classical 格式**：

```yaml
payload:
  # DOMAIN-SUFFIX - 匹配域名及子域名
  - DOMAIN-SUFFIX,google.com
  - DOMAIN-SUFFIX,github.com

  # DOMAIN - 精确匹配
  - DOMAIN,www.google.com

  # DOMAIN-KEYWORD - 关键词匹配
  - DOMAIN-KEYWORD,google

  # DOMAIN-REGEX - 正则匹配
  - DOMAIN-REGEX,.*\.cn$

  # IP-CIDR - IP 网段
  - IP-CIDR,10.0.0.0/8
```

## 🚀 快速使用

### 1. 添加规则

在 `rules/domain/` 目录下创建 `.yaml` 文件：

```yaml
# rules/domain/my-rules.yaml
payload:
  - DOMAIN-SUFFIX,example.com
  - DOMAIN-SUFFIX,another.com
```

### 2. 推送到 GitHub

```bash
git add -A
git commit -m "Update rules"
git push origin main
```

GitHub Actions 会自动构建并发布到 `release` 分支。

### 3. 在 Mihomo 中使用

```yaml
rule-providers:
  my-rules:
    type: http
    behavior: classical
    url: "https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/classical/my-rules.yaml"
    path: ./ruleset/my-rules.yaml
    interval: 86400

rules:
  - RULE-SET,my-rules,DIRECT
```

## 📦 产出物

每次构建会生成以下格式：

| 格式 | 路径 | 说明 |
|---|---|---|
| MRS | `mrs/domain/*.mrs` | 二进制格式，推荐使用 |
| YAML | `classical/*.yaml` | classical 格式 |
| DAT | `geosite.dat` | V2Ray geosite 格式 |

## 🔗 CDN 地址

```
# jsDelivr CDN（推荐）
https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/classical/规则名.yaml

# GitHub Raw
https://raw.githubusercontent.com/YoshiokaHikari/mihomo-rules/release/classical/规则名.yaml
```

## ⚙️ 自动构建

推送到 `main` 分支且修改了 `rules/` 目录时，GitHub Actions 会自动：

1. ✅ 解析 YAML 规则文件
2. ✅ 构建 `geosite.dat`
3. ✅ 转换为 `.mrs` 格式
4. ✅ 发布到 `release` 分支和 GitHub Release
5. ✅ 清除 jsDelivr CDN 缓存

## 📄 License

MIT
