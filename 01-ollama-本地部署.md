# 教程 01｜Ollama 本地部署 Qwen2.5-14B（零成本跑通本地大模型）

> 目标：在 <你的服务器IP> 上装好 Ollama，拉取 Qwen2.5-14B，跑通对话和 API
> 耗时：约 40-60 分钟（大部分时间在下载 9GB 模型）
> 难度：入门

---

## 一、目标（做完这步你得到什么）

1. 一台能本地跑大模型的机器（<你的服务器IP>）
2. 能命令行和 Qwen2.5-14B 对话
3. 有一个本地 API 端点（`http://<你的服务器IP>:11434`），后面 LangChain/Agent 直接接它

---

## 二、原理（动手前先搞懂）

### 1. Ollama 是什么？
Ollama 是一个「本地大模型运行时」，本质是给 `llama.cpp` 套了个易用外壳。

- 没有 Ollama 之前：要自己编译 llama.cpp、下载 GGUF 权重、写一堆参数才能跑模型
- 有了 Ollama：`ollama pull qwen2.5:14b` 一条命令拉模型，`ollama run` 直接对话

### 2. 为什么选 Ollama 而不是 vLLM / llama.cpp？
| 工具 | 特点 | 适合谁 |
|---|---|---|
| **Ollama** | 最简单，一条命令搞定，自带 API | 入门/开发 ⭐ |
| llama.cpp | 底层库，CPU 也能跑，要自己编译 | 极致性能/嵌入式 |
| vLLM | 生产级，高吞吐，要 GPU | 公司正式部署 |

你现在是「学习 + 验证」，选 Ollama 最省事。

### 3. 为什么用 INT4 量化？
- 模型原始权重是 FP16（每个参数 2 字节）：14B 模型要 28GB
- INT4 量化（每个参数 0.5 字节）：14B 模型只要 9GB，**内存占用降到 1/3**
- 代价：精度略降，但问答场景几乎无感
- Ollama 拉 `qwen2.5:14b` 默认就是 4-bit 量化（GGUF 格式）

### 4. 为什么 14B 而不是 7B/32B？
（回顾上一条对话）这台机器是「强 CPU + 128GB 内存 + 弱 GPU」：
- 14B 是 CPU 推理的「效果/速度平衡点」——答得比 7B 好，速度还能接受（4-8 token/s）
- 32B 效果好但 CPU 太慢（2-4 token/s）

---

## 三、前置条件

- 能 SSH 到 <你的服务器IP>（用户 `Administrator` + 密钥 `<你的SSH密钥名>`）
- 那台机器磁盘空闲 1.5TB（够放 9GB 模型）
- 本机有 `ssh` 命令（Windows 10 自带）

---

## 四、操作步骤

### 步骤 1：SSH 连上 <你的服务器IP>

在你自己的电脑（Windows）打开 PowerShell，执行：

```powershell
ssh -i $env:USERPROFILE\.ssh\<你的SSH密钥名> Administrator@<你的服务器IP>
```

看到类似 `Administrator@xxx C:\Users\Administrator>` 提示符 = 连上了。

> 💡 如果不想每次输这串，可以建个 alias。先跑通，后面再优化。

### 步骤 2：下载 Ollama 安装包

在**远程机器**（已 SSH 进去）执行，用 PowerShell 下载：

```powershell
Invoke-WebRequest -Uri "https://ollama.com/download/OllamaSetup.exe" -OutFile "C:\OllamaSetup.exe"
```

### 步骤 3：静默安装

```powershell
Start-Process "C:\OllamaSetup.exe" -ArgumentList "/S" -Wait
```

`/S` = 静默安装（不弹界面）。装完 Ollama 会注册到系统，默认数据目录 `C:\Users\Administrator\.ollama`。

### 步骤 4：验证安装

新开一个 cmd/powershell（重新加载 PATH）：

```powershell
ollama --version
```

打印版本号 = 安装成功。

### 步骤 5：配置环境变量（可选但推荐）

默认 Ollama 只监听 `127.0.0.1:11434`（本机）。以后你想从**别的机器**（比如你自己的电脑）调它的 API，需要让它监听所有网卡：

```powershell
# 设置环境变量（用户级，永久生效）
[Environment]::SetEnvironmentVariable("OLLAMA_HOST", "0.0.0.0:11434", "User")
```

> ⚠️ 设完后要重启 Ollama 服务才生效（重启机器或重启服务）。这一步先记着，等跑通再配。

### 步骤 6：拉取 Qwen2.5-14B 模型

```powershell
ollama pull qwen2.5:14b
```

- 会下载约 9GB（4-bit 量化 GGUF），时间取决于网速，耐心等
- 看到 `success` = 拉取完成

### 步骤 7：对话测试

```powershell
ollama run qwen2.5:14b
```

进入交互对话，输入任意问题（比如「用一句话解释什么是 RAG」），看回复。

- 第一次回答前会有「首 token 延迟」几秒，之后逐字输出
- 输入 `/bye` 退出

### 步骤 8：测 API（验证 LangChain 能接）

另开一个窗口，执行：

```powershell
Invoke-RestMethod -Uri "http://127.0.0.1:11434/api/generate" -Method Post -Body '{"model":"qwen2.5:14b","prompt":"你好","stream":false}' -ContentType "application/json"
```

返回 JSON（含 `response` 字段）= API 通了，后面 LangChain 直接 `base_url="http://127.0.0.1:11434"` 就能接。

---

## 五、验收标准（做到这些才算过）

- [ ] `ollama --version` 有版本号
- [ ] `ollama list` 能看到 `qwen2.5:14b`
- [ ] `ollama run qwen2.5:14b` 能正常问答
- [ ] 步骤 8 的 API 返回 JSON
- [ ] 记下实测速度：一个 100 字的回答大概等了多少秒（截图/记录，方便后面对比）

---

## 六、常见问题

| 问题 | 原因 | 解决 |
|---|---|---|
| `ollama` 不是内部命令 | PATH 没刷新 | 重开终端，或手动加 PATH |
| 下载很慢/中断 | 网络问题 | 重跑 `ollama pull`，支持断点续传 |
| 回答特别慢（>1分钟）| CPU 推理正常现象 | 14B CPU 就是这么慢，先跑通再说 |
| API 连不上 | 只监听 127.0.0.1 | 配 `OLLAMA_HOST=0.0.0.0` 后重启 |
| 磁盘空间不足 | 模型 9GB + 缓存 | 检查 C 盘，确认有 20GB+ 空闲 |

---

## 七、做完后的下一步

跑通后告诉我，我给你**教程 02**。按顺序大概是：
- 教程 02：AI 底层原理（Transformer + 注意力，手写 mini demo）
- 教程 03：LangChain/LangGraph 入门
- 教程 04：RAG 全链路
- 教程 05：MCP Server 开发
- 教程 06：企业知识库问答 Agent（整合项目，公司落地）
