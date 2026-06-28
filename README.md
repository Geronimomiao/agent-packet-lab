# agent-packet-lab · Codex CLI vs Claude Code 抓包实验台

用 [claude-tap](https://github.com/liaohch3/claude-tap)(本地代理抓包工具)拦截 **Codex CLI** 和 **Claude Code** 跟各自后端的真实 API 流量,逐包分析,搞清楚一个 AI 编码 Agent 到底是怎么运作的,并把两者并排对比。

> 🔍 **在线查看(GitHub Pages)**:打开仓库的 Pages 链接即可,或本地 `python3 -m http.server` 后访问 `index.html`。

---

## 这是什么

一个**带中文讲解的交互式抓包查看器**(`index.html`)。同一个 prompt 分别喂给 Codex 和 Claude Code,把真实请求/响应抓下来,做成可点击、可折叠、**每个字段旁带中文注释**的实验台。

### 5 个实验(同 prompt,两个 Agent)

| 实验 | Prompt | 看点 |
|---|---|---|
| ① 一句 Hi | `hi` | 最小请求里都塞了什么(系统提示 / 工具 / 鉴权 / 预热) |
| ② Node 版本 | `Node.js 最新稳定版?` | web_search:Codex 服务端一次闭环 vs Claude 客户端壳 + 服务端子调用 |
| ③ 生成一张图 | `生成红苹果 PNG` | Codex 原生生图 vs Claude 没有、委托 codex exec |
| ④ 第一性原理网页 | `做一个单页 HTML` | 完整 agent loop:读 skill → 写 → 自验;input 滚雪球 |
| ⑤ Thinking 开/关 | 硬币陷阱题 | 开思考 → 答案前多一个 reasoning 块;关 → 直接答 |

打开实验台:5 张卡选实验 → 进去看带注释的抓包 → `💬 概括 / { } 原始抓包` 切换 → 变体(Codex/Claude 或 关/开)切换。

---

## 核心结论(从抓包反推)

- **Agent = ReAct 内核 + 一圈外围工程**(工具 / Skill / 上下文管理 / 沙箱 / 传输)。
- **服务端工具 vs 客户端工具**是主线:客户端工具(跑命令/改文件)在你本机跑、可移植;服务端工具(web_search/生图)绑后端,换模型就丢。
- **`store: false`** → 服务端不存对话,客户端每轮把完整历史重发(聊越长 input 越大)。
- **store ≠ prompt cache**:前者是"存不存对话对象",后者是"前缀 KV 缓存几分钟"。
- **Skill = 渐进披露**:清单只带"名字+描述+路径",模型选中才读全文。Codex 放 `input` 的 developer 消息,Claude 放 `messages` 的 `<system-reminder>`。
- **传输**:Codex `/v1/responses` 走 WebSocket + 启动预热;Claude `/v1/messages` 走普通 HTTP POST。
- **Thinking**:Codex 参数驱动(`reasoning.effort`),Claude prompt 驱动 + adaptive。

---

## 目录

```
index.html                 抓包实验台(主页)
trace_<实验>_<变体>.json    各实验的抓包数据(已脱敏)
compare.html               两个 Agent 生成的"第一性原理网页"左右对照
cmp_codex/ cmp_claude/     各自生成的网页成品
```

## 关于数据脱敏

抓包数据为教学目的保留了 Codex / Claude Code 的**系统提示词、工具定义、skill 清单**等结构,但已做脱敏:
- `Authorization` 等鉴权头 → `[REDACTED]`
- 所有 UUID 类 id(account / session / thread / installation / prompt_cache_key)→ 一致化替换成假 id(保留配对关系)
- `user` / `safety_identifier` → `[REDACTED]`
- 文件路径中的用户名 → `user`
- 超大 base64(如生成的图片)→ 截断

## 致谢

抓包工具:[liaohch3/claude-tap](https://github.com/liaohch3/claude-tap)。本仓库为基于其输出的学习/分享材料。
