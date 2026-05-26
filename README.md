# 图盾·工坊 Claude Skill

> 电商商品图生成 — agent 拿到链接, 自动调图盾 API 出主图 / 卖点图 / 促销图 / 详情图.

## 你用的是哪个 agent 平台？

跳到对应段落:

- [OpenClaw](#openclaw) — 开源自托管 AI agent (推荐)
- [Hermes Agent](#hermes-agent) — 开源自托管 AI agent
- [Claude Desktop](#claude-desktop) — Anthropic 官方桌面客户端
- [Cherry Studio](#cherry-studio) — 国内开源多模型 MCP 客户端
- [Continue](#continue-vs-code--jetbrains) — VS Code / JetBrains 插件
- [扣子 / Dify / n8n](#扣子--dify--n8n--低代码平台) — 低代码 agent 平台
- [自家后端 (Node / Python / curl)](#自家后端集成)

不管哪个平台, 第一步都一样:

### 准备工作

1. 浏览器开 https://image.bjaydmy.com 注册并登录账号
2. 账户设置 → API Keys → 「+ 创建新 Key」, 起个名字
3. **立即复制完整密钥** (格式 `tdgz_live_xxxxxxxxxxxxxxxx`) — 只显示这一次, 关闭就再也看不到

---

## OpenClaw

[OpenClaw](https://www.openclaw.ai) 是开源自托管 AI agent 平台, 自带 MCP 客户端能力, 接飞书/钉钉/Telegram 等 20+ 消息平台.

在 OpenClaw 的 MCP server 配置里加一项 (具体 config 文件路径见 [OpenClaw 文档](https://docs.openclaw.ai/zh-CN/cli/acp)):

```json
{
  "mcpServers": {
    "tudun-image-batch": {
      "transport": "streamable-http",
      "url": "https://image.bjaydmy.com/api/v1/mcp",
      "headers": {
        "Authorization": "Bearer tdgz_live_你的密钥"
      }
    }
  }
}
```

重启 OpenClaw, 在飞书/钉钉群里 @ bot 试:

> 「这是我的产品图 https://example.com/x.jpg, 做张白底主图」

---

## Hermes Agent

[Hermes Agent](https://hermesagent.org.cn) 是开源自托管 AI agent 框架, 原生支持 MCP, 集成飞书/钉钉/企业微信/Telegram/WhatsApp 多平台.

在 Hermes 的 MCP 配置文件里加 (具体路径见 Hermes 官方文档):

```json
{
  "mcpServers": {
    "tudun-image-batch": {
      "transport": "streamable-http",
      "url": "https://image.bjaydmy.com/api/v1/mcp",
      "headers": {
        "Authorization": "Bearer tdgz_live_你的密钥"
      }
    }
  }
}
```

`hermes mcp serve` 重启即生效.

---

## Claude Desktop

编辑配置文件:

- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "tudun-image-batch": {
      "transport": "streamable-http",
      "url": "https://image.bjaydmy.com/api/v1/mcp",
      "headers": {
        "Authorization": "Bearer tdgz_live_你的密钥"
      }
    }
  }
}
```

重启 Claude Desktop, 对话框右下角看到工具图标即装好.

---

## Cherry Studio

设置 → MCP 服务器 → 添加, 填:

- **类型**: streamable HTTP (或 SSE)
- **URL**: `https://image.bjaydmy.com/api/v1/mcp`
- **请求头**: `Authorization: Bearer tdgz_live_你的密钥`

保存后在对话里选用即可.

---

## Continue (VS Code / JetBrains)

[Continue](https://continue.dev) 的 `~/.continue/config.json` 里加:

```json
{
  "mcpServers": [
    {
      "name": "tudun-image-batch",
      "transport": "streamable-http",
      "url": "https://image.bjaydmy.com/api/v1/mcp",
      "headers": {
        "Authorization": "Bearer tdgz_live_你的密钥"
      }
    }
  ]
}
```

重启 Continue 即生效.

---

## 扣子 / Dify / n8n / 低代码平台

不用 MCP, 用 OpenAPI 直接 import:

1. 平台后台 → 工具 / 插件 → 「导入 OpenAPI」
2. URL 填: `https://image.bjaydmy.com/api/v1/openapi.json`
3. 鉴权选 Bearer, 填你的密钥
4. 完成, 在工作流里拖节点即可

---

## 自家后端集成

直接调 REST API:

**Node.js**:

```js
const KEY = process.env.TUDUN_API_KEY;
const res = await fetch('https://image.bjaydmy.com/api/v1/images/generations', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${KEY}`,
    'Content-Type': 'application/json',
    'Idempotency-Key': crypto.randomUUID(),
  },
  body: JSON.stringify({
    image_url: 'https://your-cdn.com/source.jpg',
    workflow: 'selling-points',
    prompts: ['卖点 A', '卖点 B', '卖点 C'],
  }),
});
console.log((await res.json()).data.map(d => d.url));
```

**Python**:

```python
import os, requests, uuid
res = requests.post(
    "https://image.bjaydmy.com/api/v1/images/generations",
    headers={
        "Authorization": f"Bearer {os.environ['TUDUN_API_KEY']}",
        "Idempotency-Key": str(uuid.uuid4()),
    },
    json={"image_url": "https://your-cdn.com/x.jpg", "workflow": "main-image"},
    timeout=180,
)
print([d["url"] for d in res.json()["data"]])
```

完整 API 文档:
- 中文: https://github.com/LmaoAgent/tudun-workshop/blob/main/docs/api-v1.md
- English: https://github.com/LmaoAgent/tudun-workshop/blob/main/docs/api-v1.en.md
- OpenAPI: https://image.bjaydmy.com/api/v1/openapi.json

---

## 装上以后怎么用

装好以后, 直接对 agent 说人话, 例如:

- 「这是我的产品图 `https://...jpg`, 做张白底主图」
- 「用这张原图做 3 张卖点图: 突出轻薄、防水、长续航」
- 「给这个商品生成拼多多详情页, 4 张, 冷色调高级感」

agent 会自动识别意图, 调对应工具, 把生成图的 URL 贴回对话.

---

## SKILL.md (给 Claude / Code 装)

如果你用的是 Claude.ai / Claude Code 等支持 Anthropic Skill 格式的平台, 可以直接装这个仓库 ([SKILL.md](SKILL.md) 里有完整规则):

```
https://github.com/LmaoAgent/tudun-skill
```

---

## 主仓库 + License

主仓库: https://github.com/LmaoAgent/tudun-workshop
License: MIT
