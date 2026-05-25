# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目特点

纯 vanilla HTML/CSS/JS 项目，无框架、无构建工具、无 package.json。两个文件：

- `index.html`（1288 行）— 主 SPA，含所有 CSS + HTML + JS
- `login.html`（272 行）— 登录/注册页

## 运行方式

直接通过后端 `resources/static/` 目录托管，或浏览器打开（需后端在 `localhost:8086` 运行以调用 API）。无构建/编译步骤。

## 架构概览

**API 通信：** 后端基地址硬编码为 `const API = 'http://localhost:8086'`，在 `index.html` 和 `login.html` 中各定义一次。认证请求通过 `Authorization: Bearer <token>` 头。

**认证流程：**
- JWT Token 存储在 `localStorage.getItem('token')`
- `index.html` 加载时检查 token 存在性，无 token 则跳转 `login.html`
- 客户端 base64 解码 JWT payload 获取手机号用于显示，不做签名验证
- 登出时清除 token 并跳回 `login.html`

**主页面三标签页（`switchTab()` 切换）：**
1. **查词** — 输入英文/中文词，调用 `lookup()`。中文先翻译为英文，再并行查字典 API + 后端保存。渲染含音标/释义/关联词/例句的词条卡片
2. **翻译句子** — 文本区输入句子，Ctrl+Enter 提交，调用后端翻译 API，返回原文+译文+逐词释义，词可点击交互
3. **我的单词本** — 表格列出已保存单词，支持单条删除和全部清空

**核心交互模式：**
- 句子中的单词渲染为 `<span class="sword">` 可交互元素——悬停显示中文释义 tooltip，点击触发完整查词
- 侧边栏（CSS `transform: translateX`）用于在句子中展示单词详情
- 使用 `Promise.allSettled` 并行调用多个 API（允许部分失败）

**外部 API（客户端直调，不经后端代理）：**
- `api.dictionaryapi.dev` — 英文词典定义/音标/例句
- `api.datamuse.com` — 关联词

**安全：** XSS 防护通过自定义 `escapeHtml()` 函数手动完成。Token 存 localStorage（有 XSS 泄露风险）。
