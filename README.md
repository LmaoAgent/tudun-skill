# 图盾·工坊 Claude Skill

> 让 AI agent 自动产出电商商品图. 一张产品照 → 主图 / 卖点图 / 促销图 / 详情图. 5 分钟接入 Hermes / OpenClaw / Claude Desktop / Cherry Studio / 扣子 / Dify / n8n.

**仓库链接** (发给 agent 用户): `https://github.com/LmaoAgent/tudun-skill`

---

## 目录

- [这是什么](#这是什么)
- [4 种图速览](#4-种图速览)
- [准备工作 (3 步, 一次)](#准备工作-3-步-一次)
- [装到你的 agent 平台](#装到你的-agent-平台)
  - [Hermes Agent](#hermes-agent) · [OpenClaw](#openclaw) · [Claude Desktop](#claude-desktop) · [Cherry Studio](#cherry-studio) · [Continue](#continue-vs-code--jetbrains) · [扣子/Dify/n8n/秒哒](#扣子--dify--n8n--秒哒-低代码平台) · [自家后端](#自家后端集成)
- [装好以后怎么用](#装好以后怎么用)
- [4 种 workflow 详解](#4-种-workflow-详解)
- [Prompt 写作 5 条铁律](#prompt-写作-5-条铁律-直接决定生成质量)
- [完整请求参数表](#完整请求参数表)
- [完整响应字段表](#完整响应字段表)
- [错误处理](#错误处理-调用方常踩坑)
- [幂等](#幂等-避免重复扣费)
- [响应头 (限流信号)](#响应头-限流信号)
- [接通后的调试 tips](#接通后的调试-tips)
- [安全 / 最佳实践](#安全--最佳实践)
- [FAQ](#faq)
- [反馈 / 报障](#反馈--报障)
- [Changelog](#changelog)
- [相关链接](#相关链接)
- [License](#license)

---

## 这是什么

[图盾·工坊](https://image.bjaydmy.com) 是电商商品图批量生成 SaaS. 这个仓库把它的能力封装成 **AI agent 通用工具**, 你装到自己的 agent 平台上 (Hermes / OpenClaw / Claude Desktop 等), 就能直接对 agent 说人话:

> 「这是我的产品图 https://example.com/x.jpg, 做 3 张卖点图突出轻薄、防水、长续航」

agent 自动调图盾 API, 30 秒后把生成图的 URL 贴回对话.

### 适合谁

- ✅ 淘宝 / 天猫 / 京东 / 拼多多 / 抖店 卖家, 每天需要产出大量商品图
- ✅ 电商代运营 / 美工外包 / 跨境卖家
- ✅ 用 Hermes / OpenClaw 等 AI agent 做自动化工作流的人
- ❌ 一般 AI 绘图爱好者 (不是商品图场景, 用 Midjourney / SD 更合适)
- ❌ 单次手工出图 (不如直接登 https://image.bjaydmy.com 工作台拖拽)

### 数据流

```
你 (人话) → agent 平台 (Hermes/OpenClaw/Claude Desktop/...)
              ↓ MCP / OpenAPI 调用
        图盾·工坊 API (image.bjaydmy.com)
              ↓ 拉原图 + 内部生成
        签名 URL 24h 有效 → agent → 你
```

整链路是同步的, 30~120 秒. 不需要 webhook 回调, 不需要轮询.

---

## 4 种图速览

| workflow | 出几张 | 扣几点 | 典型用法 |
|---|---|---|---|
| `main-image` | 1 | 1 | **白底主图**, 第一张电商展示图 |
| `selling-points` | 4 (1~6 可调) | =出图数 | **卖点海报**, 围绕产品优势, 适合详情页前几张 |
| `promotion-images` | 1 | 1 | **促销主视觉**, 618 / 双 11 大促, 带主标语和价格 |
| `detail-images` | 4 (1~6 可调) | =出图数 | **详情页长图**, 按平台 (淘宝 / 京东 / 拼多多) 规格自动适配 |

每张图 = 1 点积分. 积分从 [图盾官网](https://image.bjaydmy.com) 充值, 价格见会员页. 试用账户每天有免费额度.

---

## 准备工作 (3 步, 一次)

1. 浏览器开 **https://image.bjaydmy.com** 注册并登录
2. 进 **账户设置 → API Keys → 「+ 创建新 Key」**, 起个名字
3. **立即复制完整密钥** (格式 `tdgz_live_xxxxxxxxxxxxxxxx`) — 出于安全考虑只显示一次, 离开页面就再也看不到

> 万一密钥丢了或泄露了, 回账户设置删掉重建即可, 30 秒的事.

---

## 装到你的 agent 平台

跳到你用的平台:

- [Hermes Agent](#hermes-agent) ★
- [OpenClaw](#openclaw) ★
- [Claude Desktop](#claude-desktop)
- [Cherry Studio](#cherry-studio)
- [Continue (VS Code / JetBrains)](#continue-vs-code--jetbrains)
- [扣子 / Dify / n8n / 秒哒 (低代码平台)](#扣子--dify--n8n--秒哒-低代码平台)
- [自家后端 (Node / Python / 任意 HTTP 客户端)](#自家后端集成)

不同平台配置文件路径和字段名略有差异, 但**核心配置都是同三件套**:

- **MCP server URL**: `https://image.bjaydmy.com/api/v1/mcp`
- **Transport**: `streamable-http`
- **HTTP Header**: `Authorization: Bearer tdgz_live_你的密钥`

---

### Hermes Agent

[Hermes Agent](https://hermesagent.org.cn) — 开源自托管 AI agent 框架, 原生 MCP, 接飞书 / 钉钉 / 企业微信 / Telegram / WhatsApp / Discord 等多平台.

在 Hermes 的 MCP 配置文件里加一项 (具体路径见 Hermes 官方文档):

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

重启 `hermes mcp serve` 即生效. 在飞书/钉钉群 @ bot 试一句 「做张白底主图, 原图 https://...」.

---

### OpenClaw

[OpenClaw](https://www.openclaw.ai) — 开源自托管 AI agent 平台, 自带 MCP 客户端能力, 接 20+ 消息平台, 集成 100+ 工具.

在 OpenClaw 的 MCP server 配置里加 (具体路径见 [OpenClaw 配置文档](https://docs.openclaw.ai/zh-CN/cli/acp)):

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

重启 OpenClaw, 在你接的群聊里直接说话试.

---

### Claude Desktop

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

完全退出 Claude Desktop (cmd+Q / 退出), 再打开. 对话框看到 🔌 图标即装好.

---

### Cherry Studio

[Cherry Studio](https://cherry-ai.com) — 国内开源多模型 GUI 客户端, 支持 MCP.

设置 → MCP 服务器 → 添加, 填:

- **类型**: streamable HTTP
- **URL**: `https://image.bjaydmy.com/api/v1/mcp`
- **请求头**: `Authorization: Bearer tdgz_live_你的密钥`

保存. 新对话里选用即可.

---

### Continue (VS Code / JetBrains)

[Continue](https://continue.dev) — 开源 AI 编程助手, VS Code 和 JetBrains 都有插件.

编辑 `~/.continue/config.json`:

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

重启编辑器即生效.

---

### 扣子 / Dify / n8n / 秒哒 (低代码平台)

这类平台一般不直接吃 MCP, 但吃 **OpenAPI 3.0**. 我们提供了开箱即用的 spec:

```
https://image.bjaydmy.com/api/v1/openapi.json
```

各平台操作大同小异:

| 平台 | 操作路径 |
|---|---|
| **扣子 (Coze)** | 工作流 → 插件 → 导入插件 → 粘贴 OpenAPI URL |
| **Dify** | 工作室 → 工具 → API 工具 → 导入 → 粘贴 OpenAPI URL |
| **n8n** | HTTP Request 节点, 或用 Import OpenAPI 节点 |
| **秒哒** | 智能体编辑器 → 添加工具 → 自定义 API → 粘贴 OpenAPI URL |

鉴权一律选 `Bearer Token`, 值填你的密钥 `tdgz_live_xxx`.

---

### 自家后端集成

如果你在自己写代码, 直接调 REST API:

**Node.js**:

```js
import crypto from 'node:crypto';

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
    prompts: ['突出轻薄', '突出防水', '突出长续航'],
  }),
});
const data = await res.json();
if (res.ok) {
  console.log('图片 URL:', data.data.map(d => d.url));
} else {
  console.error(`[${data.code}] ${data.message}`);
}
```

**Python**:

```python
import os, uuid, requests

res = requests.post(
    "https://image.bjaydmy.com/api/v1/images/generations",
    headers={
        "Authorization": f"Bearer {os.environ['TUDUN_API_KEY']}",
        "Idempotency-Key": str(uuid.uuid4()),
    },
    json={
        "image_url": "https://your-cdn.com/source.jpg",
        "workflow": "main-image",
        "prompt": "纯白背景, 柔和顶光",
    },
    timeout=180,  # 注意 ≥ 120s
)
data = res.json()
if res.status_code == 200:
    print([d["url"] for d in data["data"]])
else:
    print(f"[{data['code']}] {data['message']}")
```

完整 REST API 文档:
- 中文: https://github.com/LmaoAgent/tudun-workshop/blob/main/docs/api-v1.md
- English: https://github.com/LmaoAgent/tudun-workshop/blob/main/docs/api-v1.en.md

---

## 装好以后怎么用

对 agent 说人话, 例如:

| 你说什么 | agent 自动做什么 |
|---|---|
| 「这是产品图 https://...jpg, 做张白底主图」 | 调 `generate_main_image`, 30 秒返签名 URL |
| 「用这张图做 3 张卖点图: 轻薄/防水/长续航」 | 调 `generate_selling_points`, `prompts: [...]`, 出 3 张 |
| 「给这个商品做拼多多详情页, 4 张, 冷色调高级感」 | 调 `generate_detail_images`, `platform_profile: pdd` |
| 「618 大促主图, 主标语满 199 减 30」 | 调 `generate_promotion_image` |
| 「看下我账户还剩多少点」 | 调 `get_account_balance`, 不扣点 |

---

## 4 种 workflow 详解

### `main-image` — 白底主图

电商列表里那张第一展示图. 框架自动去原图背景, 居中商品, 加柔光.

- **何时用**: 新品上架第一步, 替换乱糟糟的拍摄底图
- **默认行为**: 1 张, 1280×1280, 纯白背景, 柔和顶光, 无阴影
- **必传**: 只需 `image_url` + `workflow: "main-image"`
- **可选**: `prompt` (覆盖默认, 例如「米白色背景, 软光线」)

```json
{ "image_url": "https://...jpg", "workflow": "main-image" }
```

### `selling-points` — 多张卖点图

围绕产品优势出多张海报, 适合详情页前几屏吸引点击.

- **何时用**: 商品有多个卖点要单独突出 (材质 / 工艺 / 功能 / 适用场景)
- **默认行为**: 4 张, 1280×1280, 围绕 prompt 自动 split
- **必传**: `image_url` + `workflow: "selling-points"`
- **可选**:
  - `prompt` (整体方向, 框架自动 split 成 4 张)
  - `prompts: [...]` (每张独立, 数组长度锁定出图数 1~6)
  - `n` (出图数 1~6, 与 prompts 互斥)
  - `negative_prompt` (反向约束)

两种粒度选一种:

```json
// A: 单 prompt 整体方向
{ "image_url": "...", "workflow": "selling-points", "prompt": "突出三大卖点, 蓝白配色" }

// B: prompts 数组每张独立
{ "image_url": "...", "workflow": "selling-points",
  "prompts": [ "突出轻薄, 仅 8mm 厚", "突出防水, IP68", "突出长续航" ] }
```

### `promotion-images` — 促销主视觉

大促 / 节日 / 活动用的单张主视觉. **prompt 必传且要具体**.

- **何时用**: 618 / 双11 / 春节 / 店庆 / 新品发布
- **默认行为**: 1 张, 1280×1280
- **必传**: `image_url` + `workflow: "promotion-images"` + `prompt`
- **prompt 要包含**: 活动主题 + 主标语 + 价格点 + 配色风格

```json
{
  "image_url": "https://...jpg",
  "workflow": "promotion-images",
  "prompt": "618 大促主视觉, 主标语「满 199 减 30」, 红黄渐变背景, 节日喜庆氛围"
}
```

### `detail-images` — 详情页长图

详情页用的长图集合, 框架按目标电商平台自动适配尺寸和质量规则.

- **何时用**: 详情页内容图 (材质介绍 / 使用场景 / 工艺细节 / 售后)
- **默认行为**: 4 张, 1536×2048 (竖版长图), 按 taobao 规格
- **必传**: `image_url` + `workflow: "detail-images"`
- **可选**:
  - `platform_profile`: `taobao` (默认) / `tmall` / `jd` / `pdd` — 影响图片规格
  - `prompt` 或 `prompts` (同 selling-points 两种粒度)
  - `n` (1~6)

```json
{
  "image_url": "https://...jpg",
  "workflow": "detail-images",
  "platform_profile": "pdd",
  "prompt": "冷色调高级感, 强调材质细节",
  "n": 6
}
```

---

## Prompt 写作 5 条铁律 (直接决定生成质量)

1. **聚焦差异, 省略默认**: 别写「白底图、柔和光线、无阴影」(系统已经管这些), 写「冷蓝色调, 金属质感」
2. **促销 / 详情图必传 prompt 且要具体**: 模板不知道你的产品和活动, 必须提供「主标语 / 价格 / 风格关键词」
3. **不要描述输入图**: prompt 是「你想要什么结果」, 不是「输入图是什么」. 模型读 image_url 已经知道
4. **单条 ≤ 200 字**: 超了会被模型截断
5. **中英文都行**: 按你想表达的语义自由选

两种 prompt 粒度 (多图 workflow):

- **单 prompt**: `"prompt": "突出三大卖点, 蓝白配色"` (框架自动 split 成 N 张)
- **prompts 数组**: `"prompts": ["第一张...", "第二张...", "第三张..."]` (每张独立, 数组长度锁定出图数)

两者**互斥**, 同时传返 400.

---

## 完整请求参数表

`POST /api/v1/images/generations`

| 字段 | 类型 | 是否必传 | 默认 | 含义 |
|---|---|---|---|---|
| `image_url` | string | **必传** | — | 原图公网 URL, https only, ≤ 20MB |
| `workflow` | string | **必传** | — | `main-image` / `selling-points` / `promotion-images` / `detail-images` |
| `prompt` | string | 可选 | workflow 默认 | 整体方向, ≤ 200 字, 与 `prompts` 互斥 |
| `prompts` | string[] | 可选 | — | 每张图独立 prompt, 长度 1~6, 仅多图 workflow, 与 `prompt` / `n` 互斥 |
| `negative_prompt` | string | 可选 | — | 反向约束 (「不要出现什么」) |
| `raw` | bool | 可选 | `false` | `true` 时跳过框架模板, prompt 100% 由 agent 控制 |
| `model` | string | 可选 | workflow 默认 | 见 `GET /api/v1/models` |
| `size` | string | 可选 | workflow 默认 | 输出尺寸, 如 `1280x1280` / `1536x2048` |
| `n` | int | 可选 | workflow 默认 | 出图数 1~6, 仅多图 workflow, 与 `prompts` 互斥 |
| `platform_profile` | string | 可选 | `taobao` | `taobao` / `tmall` / `jd` / `pdd`, **仅 detail-images** |

**请求头**:

| 头 | 是否必传 | 含义 |
|---|---|---|
| `Authorization: Bearer tdgz_live_xxx` | **必传** | API Key |
| `Content-Type: application/json` | **必传** | — |
| `Idempotency-Key: <uuid>` | 强烈推荐 | 防重复扣费, 24h 内同 key + 同 body 返缓存 |

---

## 完整响应字段表

**成功** (HTTP 200):

| 字段 | 类型 | 含义 |
|---|---|---|
| `id` | string | job id, 用于 admin 查日志 |
| `created` | int | unix 秒 |
| `credits_used` | int | 本次实际扣点 = 生成图数 |
| `credits_remaining` | int | 账户剩余余额 (近似, 精确查 `/api/v1/account`) |
| `workflow` | string | 调用的 workflow |
| `data` | array | 生成图列表 |
| `data[].url` | string | 签名 URL, 24h 内有效, 拿到链接就能看 |
| `data[].expires_at` | int | unix 秒, 链接过期时间 |

**失败** (HTTP 400/401/402/...):

| 字段 | 类型 | 含义 |
|---|---|---|
| `code` | string | 错误码, 见下表 |
| `message` | string | 人类可读的错误描述 |
| `id` | string? | job id (如已分配) |

---

## 错误处理 (调用方常踩坑)

agent 调用失败时按错误码分类处理:

| HTTP | code | 怎么办 |
|---|---|---|
| 401 | `AUTH` | 密钥缺失/无效/已删. 提示用户检查或重建 |
| 402 | `INSUFFICIENT_CREDIT` | 余额不足. 提示用户去充值, **不要硬重试** |
| 429 | `RATE-LIMIT` | 按 `Retry-After` 头给的秒数退避后再试 |
| 429 | `QUOTA-LIMITED` | 上游模型限流. 等几秒重试 |
| 400 | `IMG-FORMAT` | image_url 不可访问 / 不是图片 / 私有 IP. 让用户换个 URL |
| 400 | `IMG-TOO-LARGE` | 输入图 > 20MB. 让用户压缩后再传 |
| 504 | `NET-TIMEOUT` | 生成超时 (>120s). **积分自动退还**, 可以重试 |
| 502 | `PROVIDER-ERROR` | 上游模型故障. 等几秒重试. 失败 3 次提示用户报障 |
| 403 | `MODERATION-REJECT` | 内容审核拒绝. 让用户换图或调 prompt 避开敏感词 |
| 409 | `IDEMPOTENCY-CONFLICT` | 用了相同 Idempotency-Key 但请求体不同. 换个新 key |

---

## 幂等 (避免重复扣费)

agent 网络抖动重试容易重复扣费. 解决: 加 `Idempotency-Key` 头.

- 同 key + 同 body, 24h 内返缓存, **不再扣点**
- 同 key + 不同 body, 返 `409 IDEMPOTENCY-CONFLICT`
- 并发同 key, 后续请求 hold 等首发结果

```bash
curl -H "Idempotency-Key: $(uuidgen)" ...
```

---

## 响应头 (限流信号)

所有响应 (含错误) 都带这三个头, 用于 agent 自适应退避:

| 响应头 | 含义 |
|---|---|
| `X-RateLimit-Limit` | 你的并发上限 (默认 2) |
| `X-RateLimit-Remaining` | 当前剩余可用槽位 |
| `X-RateLimit-Reset-At` | unix 秒, 槽位恢复时间 |
| `Retry-After` | 仅 429 时带, 秒数 |

---

## 接通后的调试 tips

第一次集成时按这个顺序调试, 比逐个试错快得多:

1. **先 whoami, 验通鉴权链路**
   ```bash
   curl https://image.bjaydmy.com/api/v1/whoami -H "Authorization: Bearer $KEY"
   ```
   返你的 email / role / credits, 不扣点. 401 就是密钥错.

2. **再 GET /api/v1/account 看余额**
   ```bash
   curl https://image.bjaydmy.com/api/v1/account -H "Authorization: Bearer $KEY"
   ```
   余额 < 1 先去充, 不然第一次调就 402.

3. **第三步 GET /api/v1/models 看可选 model / size / platform_profile**
   ```bash
   curl https://image.bjaydmy.com/api/v1/models
   ```
   公开接口, 不需要鉴权.

4. **最后试一次最简 main-image (扣 1 点)**
   ```bash
   curl -X POST https://image.bjaydmy.com/api/v1/images/generations \
     -H "Authorization: Bearer $KEY" \
     -H "Content-Type: application/json" \
     -d '{"image_url":"https://picsum.photos/1024","workflow":"main-image"}'
   ```
   返签名 URL, 浏览器打开能看图就是全链路通了.

接通后再加 Idempotency-Key、prompts 数组、多 workflow 组合等高级用法.

---

## 安全 / 最佳实践

- **密钥别提交 git**: 写进 `.env`, `.gitignore` 加 `.env`. 别贴聊天截图 / Issue / 公开仓库
- **生产用 env 变量, 不要硬编码**: `process.env.TUDUN_API_KEY` 这种
- **给每个集成项目一把独立 key**: 万一某把泄漏只删那一把, 别牵连
- **总带 Idempotency-Key**: 网络抖动重试不会重复扣费
- **监控 4xx**: 持续 401 = 密钥被吊销 / 持续 402 = 用户余额不足, 及时告警用户
- **respect 限流头**: 看到 `X-RateLimit-Remaining: 0` 就别再硬冲, 等 `Reset-At`
- **超时设 ≥ 180s**: 业务层 120s + 网络缓冲. 客户端断开服务端不会取消 job, 同 Idempotency-Key 重连可拿结果
- **生产代理超时**: 如果你架了 Nginx 反代我们 API, 别忘 `proxy_read_timeout 180s+`

---

## FAQ

**Q: 生成图的链接能分享给客户看吗?**
A: 可以, 24h 内有效, 拿到链接就能看. 如需长期保存, 客户端下载后转存自家 CDN/OSS.

**Q: 怎么知道当前余额?**
A: agent 调 `get_account_balance` 工具, 或 `GET /api/v1/account`, 不扣点.

**Q: 限流是怎么样的?**
A: 单用户默认并发 2 个任务, 24h 内 30 个任务. 内测期统一上限, 企业用户联系客服调.

**Q: trial / 试用账户能用所有功能吗?**
A: 试用账户只能调 `main-image`. 卖点图 / 促销图 / 详情图需要付费会员.

**Q: 用的是什么模型?**
A: 默认 `gpt-image-2`. 内部模型选用细节我们不另行公开, 你不用关心.

**Q: 支持哪些图片格式?**
A: 输入: JPEG / PNG / WEBP, ≤ 20MB. 输出: PNG.

**Q: image_url 必须是公网 URL 吗? 能传 base64 或文件上传吗?**
A: 本期只支持公网 URL (https). base64 / multipart 上传留作后续.

**Q: 能给 OpenAI Custom GPT 用吗?**
A: 能, 配 Custom GPT Action, URL 填 `https://image.bjaydmy.com/api/v1/openapi.json`, Bearer 鉴权.

**Q: 一次能传多张原图吗?**
A: 当前一次单张. 未来按需求扩 `image_urls: []`.

**Q: 同步等待 120 秒, 客户端中途断了怎么办?**
A: 服务端不取消 job, 会跑完. 同 Idempotency-Key 重连立即拿到结果 (24h 内). 所以放心断/重连, 不会浪费点数.

**Q: agent 平台调起来不工作怎么排查?**
A: 第一步先 curl `/api/v1/whoami` 验通鉴权链路 (见 [调试 tips](#接通后的调试-tips)). 鉴权通 = 我们这边 OK, 问题在 agent 平台的 MCP 配置.

**Q: MCP server 是否会有 stdio transport?**
A: 当前只支持 streamable HTTP. 大部分主流 MCP 客户端 (Claude Desktop / Cherry Studio / Continue / Hermes / OpenClaw) 都支持, 覆盖 95% 场景.

**Q: 能不能在自己代码里用 Anthropic Skill 格式?**
A: 能, 这个仓库 [SKILL.md](SKILL.md) 就是 Anthropic Skill 格式, Claude Code / Claude.ai 直接装即可.

---

## 反馈 / 报障

- **客服微信** (有问题直接问最快):

  <img src="assets/wechat-qr.png" alt="客服微信二维码" width="220">

- 仓库 Issue (公开问题, 别人也能看): https://github.com/LmaoAgent/tudun-skill/issues
- 主项目 Issue: https://github.com/LmaoAgent/tudun-workshop/issues
- 官网: https://image.bjaydmy.com

---

## Changelog

- **2026-05-26**: 初版上线 — REST + MCP 双通道, 6 个 lib 模块, 4 个 workflow, 5 个 MCP tool. 主仓库 PR [#37](https://github.com/LmaoAgent/tudun-workshop/pull/37).

后续版本计划:
- v1.1: 异步 + webhook 回调
- v1.2: 多图输入 (`image_urls: []`)
- v1.3: base64 / multipart 上传

字段冻结承诺: **v1 已暴露字段永不破坏**. 新字段进 v1.1 走 RFC, 老字段语义不变.

---

## 相关链接

- **图盾·工坊主仓库**: https://github.com/LmaoAgent/tudun-workshop
- **API v1 完整文档 (中文)**: https://github.com/LmaoAgent/tudun-workshop/blob/main/docs/api-v1.md
- **API v1 完整文档 (English)**: https://github.com/LmaoAgent/tudun-workshop/blob/main/docs/api-v1.en.md
- **OpenAPI 3.0 spec**: https://image.bjaydmy.com/api/v1/openapi.json
- **MCP server endpoint**: https://image.bjaydmy.com/api/v1/mcp
- **官方网站**: https://image.bjaydmy.com

## License

MIT. 见 [LICENSE](LICENSE).
