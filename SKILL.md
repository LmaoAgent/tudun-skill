---
name: tudun-image-generation
description: |
  电商商品图生成. 用图盾·工坊 (Tudun Workshop) 同步 API 一键产出主图 / 卖点图 / 促销图 / 详情图.
  适用场景: 用户给一张产品照片, 要求生成「白底主图」「N 张卖点图」「618/双11 促销图」「淘宝/京东/拼多多详情页」.
  触发关键词: 商品主图, 白底图, 卖点图, 促销图, 详情图, 电商图, 产品图, 淘宝图, 拼多多详情, 京东详情.
  E-commerce product image generation for Taobao/JD/Pinduoduo/Tmall sellers.
license: complete
---

# 图盾·工坊 — 电商商品图生成

把产品照变成 4 类电商图: **主图** / **卖点图** / **促销图** / **详情图**.

## 何时用这个 skill

用户提了产品图 (商品照片 URL 或上传) **且** 提到下面任一情况:
- 想要白底主图 / 电商主图 / 商品主图
- 想要卖点图 / 卖点海报 / N 张突出 XX 卖点
- 想要促销图 / 大促图 / 双11 / 618 海报
- 想要详情页 / 详情图 / 长图
- 提到淘宝 / 天猫 / 京东 / 拼多多 / 抖店 这些电商平台

如果用户只是要修图 / 一般 AI 绘图 / 非电商场景, **不要用此 skill**.

---

## 接入前 (用户做一次)

1. 让用户去 https://image.bjaydmy.com 注册并登录账号
2. 账户设置 → API Keys → 创建一把密钥 (格式 `tdgz_live_xxxxxxxx`)
3. 把这把密钥告诉你 (或写到 env)

如果用户还没账号, 引导他先注册再回来.

---

## 调用方式

API 基地址: `https://image.bjaydmy.com`
鉴权: `Authorization: Bearer tdgz_live_xxx`

### 同步生图 (主接口)

`POST /api/v1/images/generations`

请求体:

```json
{
  "image_url": "https://公网可访问的原图.jpg",
  "workflow": "main-image",
  "prompt": "可选, 整体方向",
  "prompts": ["可选, 数组每张独立", "..."],
  "negative_prompt": "可选, 反向约束",
  "n": 4,
  "platform_profile": "taobao"
}
```

30~120 秒内返回:

```json
{
  "id": "gen-xxx",
  "credits_used": 4,
  "credits_remaining": 96,
  "workflow": "selling-points",
  "data": [
    { "url": "https://image.bjaydmy.com/api/v1/files/<token>", "expires_at": 1716710400 }
  ]
}
```

签名 URL **24h 内有效**, 用户点开就能看图.

### 查余额 (不扣点, 调用前推荐)

`GET /api/v1/account` → `{ credits_remaining, license_status, ... }`

如果 `credits_remaining` 不够本次预计扣点数, **先告诉用户去充值, 不要硬调**.

---

## 4 个 workflow 怎么选

| workflow | 出图数 | 适用 | 扣点 |
|---|---|---|---|
| `main-image` | 1 | 白底主图, 第一张电商展示图 | 1 |
| `selling-points` | 4 (默认) | 多张卖点海报, 围绕产品优势 | =出图数 |
| `promotion-images` | 1 | 促销主视觉, 带主标语 / 价格 | 1 |
| `detail-images` | 4 (默认) | 详情页长图, 按平台规格 | =出图数 |

---

## Prompt 写作规则 (重要 — 直接决定生成质量)

### 5 条铁律

1. **聚焦差异, 省略默认**: 不要写「白底图, 柔和光线, 无阴影」(系统已经管), 写「冷蓝色调, 金属质感」
2. **促销图 / 详情图必传 prompt 且要具体**: 模板不知道用户的产品和活动, 必须提供「主标语 / 价格 / 风格关键词」
3. **不要描述输入图**: prompt 是「想要什么结果」, 不是「输入图是什么」. 模型读 image_url 已经知道
4. **单条 ≤ 200 字**: 超了被截断
5. **中英文都行**

### 两种 prompt 粒度 (多图 workflow)

**单 prompt** (整体方向, 框架自动 split):

```json
{ "workflow": "selling-points", "prompt": "突出轻薄、防水、便携三大卖点, 蓝白配色" }
```

**prompts 数组** (每张独立, 数组长度 = 出图数):

```json
{ "workflow": "selling-points", "prompts": [
    "突出『轻薄』, 主标语「仅 8mm 厚」, 正侧面对比构图",
    "突出『防水』, 水滴溅射特写, IP68 标识居右下",
    "突出『长续航』, 电池图标 + 30 天文案"
] }
```

两者**互斥**, 同时传返 400.

### 详情图的平台规格

`platform_profile`: `taobao` (默认) / `tmall` / `jd` / `pdd`. 框架按平台规格自动注入图片尺寸和质量约束.

---

## 错误处理

agent 调用失败时按 code 分类处理:

| code | 怎么办 |
|---|---|
| `AUTH` | 提示用户检查密钥 (可能没复制全 / 被吊销) |
| `INSUFFICIENT_CREDIT` | 提示用户去充值, 不要硬重试 |
| `RATE-LIMIT` | 等 `Retry-After` 头给的秒数再试 |
| `IMG-FORMAT` | 输入 image_url 不可访问 / 不是图片 / 私有 IP. 让用户换个 URL |
| `IMG-TOO-LARGE` | 输入图 > 20MB. 让用户压缩 |
| `NET-TIMEOUT` | 生成超时 (>120s). 积分自动退还, 可以重试. 失败 3 次告诉用户报障 |
| `MODERATION-REJECT` | 内容审核拒绝. 让用户换图或调 prompt 避开敏感词 |
| `IDEMPOTENCY-CONFLICT` | 用了相同 Idempotency-Key 但请求体不同. 换个新 key |
| `PROVIDER-ERROR` | 上游模型故障. 等几秒重试 |

---

## 调用示例 (完整 curl)

```bash
KEY='tdgz_live_xxxxxxxxxxxxxxxx'

# 1. 单张主图
curl -s -X POST https://image.bjaydmy.com/api/v1/images/generations \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{ "image_url": "https://example.com/product.jpg", "workflow": "main-image" }'

# 2. 3 张卖点图, 每张独立 prompt
curl -s -X POST https://image.bjaydmy.com/api/v1/images/generations \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "image_url": "https://example.com/product.jpg",
    "workflow": "selling-points",
    "prompts": [
      "突出『轻薄』, 主标语「仅 8mm 厚」",
      "突出『防水』, IP68 水滴特写",
      "突出『长续航』, 电池 + 30 天文案"
    ]
  }'

# 3. 拼多多详情图, 4 张, 整体冷色调
curl -s -X POST https://image.bjaydmy.com/api/v1/images/generations \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "image_url": "https://example.com/product.jpg",
    "workflow": "detail-images",
    "platform_profile": "pdd",
    "prompt": "冷色调高级感, 强调材质细节"
  }'
```

---

## 重试 / 幂等

agent 网络抖动重试容易重复扣费. 解决: 加 `Idempotency-Key` 头:

```bash
-H "Idempotency-Key: $(uuidgen)"
```

- 同 key + 同 body 24h 内返缓存, **不再扣点**
- 同 key + 不同 body 返 `409 IDEMPOTENCY-CONFLICT`
- 并发同 key 后续请求等首发结果

---

## 不支持

- 一次输入多张原图 (`image_url` 单图)
- base64 / multipart 上传 (只支持公网 URL)
- 异步 + webhook 回调 (同步为主)
- 自定义模型 / 永久 OSS 存储

---

## 用户文档

中文: https://github.com/LmaoAgent/tudun-workshop/blob/main/docs/api-v1.md
英文: https://github.com/LmaoAgent/tudun-workshop/blob/main/docs/api-v1.en.md
OpenAPI: https://image.bjaydmy.com/api/v1/openapi.json
MCP server (Claude Desktop / Cherry Studio): https://image.bjaydmy.com/api/v1/mcp
