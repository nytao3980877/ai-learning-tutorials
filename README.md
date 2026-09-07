# AI 学习教程站（公开版）

> 补短板学习教程的**脱敏公开版**，配合 GitHub Pages 在线浏览。
> 完整记录（含内网部署细节）见私有仓库。

## 教程目录

| 编号 | 主题 | 状态 |
|---|---|---|
| 01 | Ollama 本地部署 Qwen2.5-14B | ✅ |
| 02 | AI 底层原理（Transformer / 注意力） | 待出 |
| 03 | LangChain / LangGraph 入门 | 待出 |
| 04 | RAG 全链路 | 待出 |
| 05 | MCP Server 开发 | 待出 |
| 06 | 企业知识库问答 Agent（整合项目） | 待出 |
| 07 | 分布式爬虫项目 | 待出 |

## 在线访问

开启 GitHub Pages 后：`https://<你的用户名>.github.io/<仓库名>/`

## 说明

- 教程内容为通用 AI / Agent 学习知识，**已脱敏**（内网 IP、SSH 密钥名用占位符替换）
- 直接双击 `index.html` 本地也能看
- 学习路线：爬虫/自动化底子 × AI Agent 交叉点补强

## 本地生成命令

```bash
# 教程 md 转 html
node md2html.js 教程.md

# 脱敏（公开前）
node sanitize-tutorial.js 原始.md 脱敏.md
```
