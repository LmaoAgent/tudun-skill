# 图盾·工坊 Claude Skill

> 电商商品图生成 — 给 Claude / Claude Code 装上, agent 就会自动调图盾 API 出主图 / 卖点图 / 促销图 / 详情图.

## 给最终用户

### 一句话装

把这个仓库 URL 发给 Claude:

```
https://github.com/LmaoAgent/tudun-skill
```

或者在 Claude Code 里 `/install-skill https://github.com/LmaoAgent/tudun-skill`.

### 你需要

1. 一个 [图盾·工坊](https://image.bjaydmy.com) 账号 (免费注册)
2. 一把 API Key (账户设置 → API Keys 自助生成)

### 装完怎么用

直接对 Claude 说人话, 例如:

- 「这是我的产品图 https://...jpg, 帮我做一张白底主图」
- 「用这张原图做 3 张卖点图, 突出轻薄、防水、长续航」
- 「给这个商品生成拼多多详情页, 4 张, 冷色调高级感」

Claude 会自动调图盾 API, 把生成的图链接贴回对话.

---

## 给开发者 / 二次集成

详细 API 接入文档:

- **REST API**: https://github.com/LmaoAgent/tudun-workshop/blob/main/docs/api-v1.md
- **MCP server URL** (Claude Desktop / Cherry Studio): `https://image.bjaydmy.com/api/v1/mcp`
- **OpenAPI spec** (扣子 / Dify / n8n import): `https://image.bjaydmy.com/api/v1/openapi.json`

主仓库: https://github.com/LmaoAgent/tudun-workshop

---

## License

MIT
