# 🛫 AI Travel Agent — 深度解析与学习笔记

> 基于 [awesome-llm-apps](https://github.com/Shubhamsaboo/awesome-llm-apps) 仓库中 `starter_ai_agents/ai_travel_agent` 项目的深度学习分析。

---

## 📊 项目概况

| 项目 | 说明 |
|------|------|
| **来源仓库** | [Shubhamsaboo/awesome-llm-apps](https://github.com/Shubhamsaboo/awesome-llm-apps) |
| **项目路径** | `starter_ai_agents/ai_travel_agent/` |
| **难度级别** | ⭐ 入门级 |
| **作者** | Shubham Saboo (Senior AI PM @ Google Cloud) |
| **许可证** | Apache 2.0 |
| **主语言** | Python |

---

## 🎯 项目简介

AI Travel Agent 是 awesome-llm-apps 仓库中**最经典的入门项目**，被设计为"30 秒即可跑通"的第一手体验。它基于 **双 Agent 协作架构**，能够根据用户输入的目的地、天数、预算和兴趣偏好，自动搜索真实旅行信息并生成个性化行程。

### 核心功能
- 🗺️ **个性化行程生成** — 逐日详细计划，含景点、活动、交通、住宿建议
- 🌐 **实时网络搜索** — Agent 调用 SerpAPI 获取真实信息（非 LLM 幻觉）
- 📅 **ICS 日历导出** — 一键下载 `.ics` 文件，导入 Google/Apple Calendar
- 🖥️ **云端+本地双模式** — 支持 OpenAI（云端）和 Ollama（本地离线）

---

## 🏗️ 架构设计

### 双 Agent 协作模式

```
┌─────────────────────────────────────────────────┐
│                  用户输入                         │
│     目的地 · 天数 · 预算 · 偏好 · 日期            │
└──────────────────┬──────────────────────────────┘
                   │
                   ▼
     ┌─────────────────────────┐
     │    🔍 Researcher Agent   │
     │                         │
     │  工具: SerpAPI 搜索      │
     │  职责:                   │
     │  · 搜索目的地景点        │
     │  · 查询天气/季节信息      │
     │  · 发现当地活动/节日     │
     │  · 收集交通/住宿参考     │
     └───────────┬─────────────┘
                 │ 结构化搜索结果
                 ▼
     ┌─────────────────────────┐
     │    📋 Planner Agent      │
     │                         │
     │  模型: GPT-4o / Ollama   │
     │  职责:                   │
     │  · 分析搜索结果          │
     │  · 组织逐日行程          │
     │  · 生成时间安排          │
     │  · 输出结构化行程文本    │
     │  · 生成 ICS 日历数据     │
     └───────────┬─────────────┘
                 │
                 ▼
     ┌─────────────────────────┐
     │     🖥️ Streamlit UI      │
     │                         │
     │  · 侧边栏: 参数输入      │
     │  · 主区域: 行程展示      │
     │  · 下载: ICS 日历按钮    │
     └─────────────────────────┘
```

### 关键设计思想

1. **关注点分离** — Researcher 只负责"找信息"，Planner 只负责"写行程"，各司其职
2. **Tool Use** — Researcher 被赋予 SerpAPI 工具，让 Agent 能获取实时真实信息
3. **最小依赖** — 不依赖 LangChain/Agno 等重框架，纯 Python + Streamlit 实现

---

## 📂 文件结构

```
starter_ai_agents/ai_travel_agent/
├── travel_agent.py          # 主程序 — 云端版（OpenAI GPT-4o）
├── local_travel_agent.py    # 本地版 — 使用 Ollama，无需 API Key
├── requirements.txt         # Python 依赖包
└── README.md                # 官方说明文档
```

---

## 🔧 技术栈详解

| 层级 | 技术 | 版本 | 用途 |
|------|------|------|------|
| **Agent 框架** | 自研（简单 Agent 模式） | — | Researcher + Planner 基础协作 |
| **LLM（云端）** | OpenAI GPT-4o | latest | 行程生成、语义理解 |
| **LLM（本地）** | Ollama | — | 完全离线推理，可选任意模型 |
| **搜索工具** | SerpAPI | — | Agent 调用以获取真实网络信息 |
| **UI 框架** | Streamlit | — | 快速搭建 Web 交互界面 |
| **日历导出** | `ics` 库 | — | 生成标准 .ics 日历文件 |
| **HTTP 请求** | `requests` | — | SerpAPI 接口调用 |

---

## 💡 核心技术概念解析

### 1. Agent 的基本设计模式

```
Agent = LLM + 系统提示词 + 工具 + 记忆
         │        │          │       │
         │        │          │       └── (本项目中暂无)
         │        │          └── SerpAPI 搜索
         │        └── "你是一个旅行研究专家..."
         └── GPT-4o / Ollama
```

本项目中的核心模式：
- **Researcher Agent** 通过提示词被定义为"旅行研究员"，赋予搜索工具
- **Planner Agent** 被定义为"行程规划师"，接收搜索结果后组织输出

### 2. Tool Use（工具调用）

这是 AI Agent 的核心能力 — LLM 不仅能"说话"，还能"做事"：

```python
# 伪代码示意
researcher_prompt = """
你是一个旅行研究专家。当用户询问旅行信息时，
使用 SerpAPI 搜索真实信息，不要依赖你的训练数据。
"""

# Agent 看到用户说"我想去东京玩3天"
# → Agent 自动调用: SerpAPI.search("东京热门景点 2024")
# → Agent 自动调用: SerpAPI.search("东京3天行程推荐")
# → 收集结果后返回给 Planner
```

### 3. 本地 vs 云端模式

| 维度 | 云端（travel_agent.py） | 本地（local_travel_agent.py） |
|------|------------------------|------------------------------|
| LLM | GPT-4o（OpenAI API） | Ollama 本地模型 |
| 费用 | 按 Token 计费 | 零 API 费用 |
| 速度 | 快（云端 GPU） | 取决于本地硬件 |
| 隐私 | 数据发送至 OpenAI | 完全本地，数据不出机器 |
| 质量 | 顶级 | 取决于所选本地模型 |
| 硬件要求 | 无 | 建议 16GB+ 显存 |

---

## 🚀 快速开始

### 前提条件

```bash
# 1. 安装 Python 3.10+
python --version

# 2. 获取 API Keys
#    - OpenAI API Key: https://platform.openai.com/api-keys
#    - SerpAPI Key: https://serpapi.com/

# 3. (可选) 安装 Ollama（本地模式）
#    下载地址: https://ollama.com/
```

### 云端版运行

```bash
# Clone 仓库
git clone https://github.com/Shubhamsaboo/awesome-llm-apps.git
cd awesome-llm-apps/starter_ai_agents/ai_travel_agent

# 安装依赖
pip install -r requirements.txt

# 设置环境变量
export OPENAI_API_KEY="your-key-here"
export SERPAPI_API_KEY="your-key-here"

# 启动 Streamlit
streamlit run travel_agent.py
```

### 本地版运行

```bash
# 确认 Ollama 已安装并拉取模型
ollama pull llama3

# 运行本地版本
streamlit run local_travel_agent.py
```

---

## 📚 推荐学习路径

| 步骤 | 任务 | 预计时间 | 学习目标 |
|------|------|----------|----------|
| 1 | Clone 并跑通 `travel_agent.py` | 30 分钟 | 感受 Agent 应用的完整流程 |
| 2 | 阅读源码，理解 Researcher/Planner 分工 | 1 小时 | 掌握双 Agent 协作模式 |
| 3 | 分析 System Prompt 设计 | 30 分钟 | 理解提示词如何控制 Agent 行为 |
| 4 | 试改成中文输出 | 30 分钟 | 理解语言的 prompt 控制 |
| 5 | 切换到 Ollama 本地模式 | 1 小时 | 理解云端/本地部署差异 |
| 6 | 添加"美食推荐"模块 | 2 小时 | 动手扩展 Agent 能力 |
| 7 | 尝试接入新的搜索源（如 DuckDuckGo） | 2 小时 | 理解工具替换的灵活性 |

---

## 🔗 相关链接

- [awesome-llm-apps 主仓库](https://github.com/Shubhamsaboo/awesome-llm-apps)
- [AI Travel Agent 源码目录](https://github.com/Shubhamsaboo/awesome-llm-apps/tree/main/starter_ai_agents/ai_travel_agent)
- [配套教程 — Unwind AI](https://www.theunwindai.com)
- [Streamlit 官方文档](https://docs.streamlit.io/)
- [SerpAPI 文档](https://serpapi.com/)

---

## 📝 学习笔记模板

> 建议在阅读源码和运行项目时，记录以下内容：

```markdown
## 学习日期: YYYY-MM-DD

### 今天跑通了什么
- [ ] travel_agent.py 成功运行
- [ ] 理解了 Researcher Agent 的提示词设计
- [ ] ...

### 遇到的问题
1. ...
2. ...

### 新学到的概念
- Agent Tool Use
- ICS 日历文件格式
- ...

### 下一步计划
- ...
```

---

## ⚠️ 注意事项

- 云端版需要有效的 **OpenAI API Key** 和 **SerpAPI Key**（均有免费额度）
- API 调用会产生费用，建议先用免费额度测试
- 本地版需要 **Ollama 已安装**并有足够显存
- Python 版本建议 3.10+，注意 `requirements.txt` 中的版本兼容性

---

> 📅 创建日期: 2026-06-10
> 👤 维护者: [@xiaozhao0625](https://github.com/xiaozhao0625)
