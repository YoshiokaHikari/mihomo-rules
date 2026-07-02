# 我的 Mihomo 规则集

自动构建、定时更新的 Mihomo 规则集，支持 `.mrs` 格式。

> 📚 详细使用说明请查看 [MRS_GUIDE.md](./MRS_GUIDE.md)

## 📁 规则文件

| 文件 | 说明 | 用途 |
|---|---|---|
| `my-direct.txt` | 直连域名 | 国内网站直连 |
| `my-proxy.txt` | 代理域名 | 需要代理的网站 |
| `streaming.txt` | 流媒体域名 | Netflix/YouTube/Disney+ 等 |
| `ai-services.txt` | AI 服务域名 | OpenAI/Claude/Gemini 等 |
| `reject-ads.txt` | 广告域名 | 广告过滤 |

## 🚀 快速使用

### 1. 添加规则

在 `rules/domain/` 目录下创建 `.txt` 文件：

```text
# 域名规则格式
example.com          # DOMAIN-SUFFIX（匹配子域名）
full:example.com     # DOMAIN（精确匹配）
keyword:example      # DOMAIN-KEYWORD（关键词）
regexp:.*\.com$      # DOMAIN-REGEX（正则）
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

## 📦 产出物

每次构建会生成以下格式：

| 格式 | 路径 | 说明 |
|---|---|---|
| MRS | `mrs/domain/*.mrs` | 二进制格式，推荐使用 |
| Text | `classical/*.list` | 纯文本格式 |
| YAML | `classical/*.yaml` | Clash classical 格式 |
| DAT | `geosite.dat` | V2Ray geosite 格式 |

## 🔗 CDN 地址

```
# jsDelivr CDN（推荐）
https://cdn.jsdelivr.net/gh/YoshiokaHikari/mihomo-rules@release/mrs/domain/规则名.mrs

# GitHub Raw
https://raw.githubusercontent.com/YoshiokaHikari/mihomo-rules/release/mrs/domain/规则名.mrs
```

## ⚙️ 自动构建

推送到 `main` 分支且修改了 `rules/` 目录时，GitHub Actions 会自动：

1. ✅ 构建 `geosite.dat`
2. ✅ 转换为 `.mrs` 格式
3. ✅ 生成 Text/YAML 格式
4. ✅ 发布到 `release` 分支和 GitHub Release
5. ✅ 清除 jsDelivr CDN 缓存

## 📄 License

MIT
