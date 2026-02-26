# markdown-url

一个 AI Agent Skill，让 Agent 在访问网页时自动通过 [`markdown.new`](https://markdown.new) 代理获取干净的 Markdown 内容，并强制保存到本地文件。

---

## 原理

`markdown.new` 是一个 URL 代理服务，能将任意网页转换为 Markdown 格式返回。本 Skill 指示 Agent 在需要阅读网页时，将目标 URL 重写为：

```
https://markdown.new/<原始URL>
```

相比直接抓取原始 HTML，这样做的好处是：内容更干净、结构更清晰、易于提取摘要或笔记。

---

## 文件结构

```
markdown-url/
└── SKILL.md        # Agent 行为规范，核心文件
```

---

## 使用方式

将本 Skill 安装到支持 Skill 的 AI Agent 环境（如 Codex、Claude Code、Gemini CLI 等）：

```bash
npx skills add https://github.com/am-will/codex-skills --skill markdown-url
```

安装后，Agent 在需要访问网页时会自动遵循本 Skill 的工作流。

---

## Agent 工作流

> **5 个步骤全部强制执行，尤其是第 4 步（保存文件）不可跳过。**

| 步骤 | 动作 |
|------|------|
| 1 | 将目标 URL 重写为 `markdown.new` 前缀形式 |
| 2 | 访问重写后的 URL（而非原始地址） |
| 3 | 判断成功：**HTTP 200 + 有内容 = 成功**，不以内容大小或格式判断 |
| 4 | **强制保存**响应内容到本地 `.md` 文件，并打印确认信息 |
| 5 | 以保存的文件为来源，生成摘要、引用或笔记 |

### 文件命名规则

文件名从原始 URL 路径自动推导：

```
https://example.com/docs/intro  →  example-com-docs-intro.md
https://example.com/            →  index.md
```

保存位置优先使用 `.markdown-cache/` 目录，不存在则保存到当前工作目录。

保存成功后 Agent 会输出确认行：

```
[markdown-url] Saved → ./example-com-docs-intro.md (4312 bytes)
```

---

## 适用场景

✅ **走 markdown.new 代理**

- 文档页面
- 博客文章 / 更新日志 / 公告
- GitHub Issue / PR 讨论
- 技术文章和指南

❌ **直接访问原始 URL**

- 登录、OAuth、支付等需要认证的页面
- 依赖 JS 动态渲染、CAPTCHA 或 Paywall 的页面
- 表单、编辑器、仪表盘等交互型页面
- 任何需要保留 Cookie / Session 状态的页面

---

## 失败信号与回退

以下情况视为代理失败，回退到原始 URL：

- HTTP `401 / 403 / 429`、"Access denied"、"rate limited"
- CAPTCHA / "verify you are human"
- 响应体为空
- 重定向循环

**以下情况不是失败，不得回退：**

- HTTP 200 但内容只有几 KB —— 这是成功
- 内容难以解析或看起来像原始 HTML —— 直接使用，不重试
- 任何 HTTP 200 响应 —— 禁止描述为"失败"或"网络问题"

---

## 常见问题

**Q：为什么有时 Agent 没有保存文件？**  
A：这是早期版本的问题。当前 SKILL.md 已将保存步骤设为独立的强制步骤（第 4 步），并要求 Agent 输出确认行，若没有保存则视为违反 Skill 规范。

**Q：Agent 说"访问失败"但日志显示 HTTP 200？**  
A：这是 Agent 误判了成功标准。当前版本已在第 3 步明确：判断成功的唯一依据是 HTTP 状态码和响应体是否非空，与内容格式无关。

**Q：可以用 firecrawl 或 puppeteer 补充抓取吗？**  
A：不可以。成功保存后禁止调用任何第三方爬取工具，避免重复抓取和不必要的失败。

---

## License

MIT
