# 🛠️ 动手实操清单

> ⚠️ **重要**: 不要让我（AI）帮你做完。本清单的目的是你**亲手**完成每一步。

---

## 🎯 学习目标

完成本清单后，你应该能够：
1. 独立运行一个 AI Agent 应用
2. 理解双 Agent 协作的代码实现
3. 通过破坏实验理解 Agent 各组件的作用
4. 在现有架构上扩展新功能

---

## 预备知识：项目文件在哪？

你已经把 `awesome-llm-apps` 大仓库 clone 到本地了，本项目的源码就在这里：

```
awesome-llm-apps/starter_ai_agents/ai_travel_agent/
├── travel_agent.py           # 云端版（OpenAI + SerpAPI）
├── local_travel_agent.py     # 本地版（Ollama，无 API 费用）
├── requirements.txt          # Python 依赖清单
└── README.MD                 # 官方说明文档
```

建议：把整个目录复制一份出来单独练习，不污染原仓库：
```bash
cp -r awesome-llm-apps/starter_ai_agents/ai_travel_agent ~/my-travel-agent-lab
cd ~/my-travel-agent-lab
```

---

## 第1步：亲手跑起来（预计30分钟）

### 1.1 先看文件结构

```bash
ls -la
# 你应该看到 4 个文件
```

### 1.2 读依赖文件

```bash
cat requirements.txt
```

观察：
- 有哪些依赖？
- `ics` 是做什么用的？猜猜看。
- 有没有 `agno`、`langchain`？为什么没有？（入门项目不需要重框架）

### 1.3 安装依赖

```bash
pip install -r requirements.txt
```

### 1.4 运行本地模式（推荐从这里开始）

```bash
# 先确保 Ollama 已安装并运行
ollama serve   # 如果还没启动

# 拉取模型（二选一）
ollama pull qwen2.5:14b    # 中文推荐
ollama pull llama3.1:8b    # 英文推荐

# 启动
streamlit run local_travel_agent.py
```

### 1.5 浏览器打开

```
http://localhost:8501
```

填一个目的地，生成你的第一个 AI 行程。

### 1.6 (选做) 运行云端版

需要 OpenAI API Key 和 SerpAPI Key：
```bash
export OPENAI_API_KEY="sk-..."
export SERPAPI_API_KEY="..."
streamlit run travel_agent.py
```

### ⚡ 预期会遇到的问题

| 报错 | 自己排查方向 |
|------|-------------|
| `ModuleNotFoundError` | `pip install` 漏了哪个？ |
| `Connection refused` | Ollama 启动了吗？`ollama list` 看看 |
| `Address already in use` | 换端口 `streamlit run ... --server.port 8502` |
| 模型回复乱码/英文 | 换 `qwen2.5:14b` 试试，它对中文理解更好 |

**先自己 Google/看文档解决，实在不行再问我。**

---

## 第2步：拆代码（预计1小时）

打开 `travel_agent.py`（云端版），逐段回答以下问题。

> 💡 **代码在哪里？** → `awesome-llm-apps/starter_ai_agents/ai_travel_agent/travel_agent.py`
> 💡 **阅读策略**：先通读一遍。然后一次只看一个问题，把答案写下来。

### 2.0 整体速览（5分钟）

先快速扫一眼文件整体：
- 文件大概多少行？
- 哪些 `import` 你不认识？（记下来，查一下）
- 有几个主要的函数？它们的调用顺序是什么？
- 用笔在纸上画：用户输入 → ? → ? → 输出的流向图

### 2.1 System Prompt 分析

```python
# 📝 找到 Researcher 的 system prompt，用自己的话回答：
# 1. Researcher 被定义成了什么角色？
# 2. 它被允许做什么？被禁止做什么？
# 3. 它的输出格式有什么要求？
```

### 2.2 Agent 调用链

```python
# 📝 找到调用 SerpAPI 的代码，回答：
# 1. 搜索关键词是怎么拼出来的？（用户输入 → 搜索 query 的转换）
# 2. Researcher 拿到搜索结果后做了什么处理？
# 3. 如果搜索失败了（断网/额度用完），代码怎么处理的？
```

### 2.3 Planner 分析

```python
# 📝 找到 Planner 的代码，回答：
# 1. Planner 拿到了什么？（Researcher 传过来的数据长什么样？）
# 2. Planner 的 prompt 和 Researcher 有什么本质区别？
#    提示：一个是"收集信息的"，一个是"组织输出的"
# 3. 每天行程的结构（上午/下午/晚上）是怎么被组织出来的？
```

### 2.4 ICS 日历是怎么生成的？

```python
# 📝 找到生成 .ics 文件的代码，回答：
# 1. 用了哪个 Python 库？
# 2. 行程文本 → ICS 格式的转换做了什么？
# 3. 为什么输出日历文件比输出纯文本更"产品化"？
```

### 2.5 Streamlit 交互

```python
# 📝 看 Streamlit 部分，回答：
# 1. 侧边栏（sidebar）收了哪些用户参数？
# 2. 主页面展示了哪几块内容？
# 3. 用户点"生成行程"按钮后，代码执行顺序是什么？
```

---

## 第3步：破坏实验（预计1小时）

> 💡 这是**最重要的步骤**。只有搞坏了才知道每部分到底有什么用。

### 实验①：拆掉 Researcher

```
操作:
  注释掉 Researcher Agent 的整个调用
  直接把用户输入传给 Planner

观察:
  Planner 还能生成行程吗？行程内容有什么变化？
  GPT 自己不知道东京有什么景点吗？为什么还要搜？

理解:
  Researcher 不只是"获取信息"，它让信息是"真实的"而非"幻觉的"
```

### 实验②：劣化 Prompt

```
操作:
  把 Researcher 的 system prompt 改成一句话: "You are a helper."

观察:
  搜索行为变了没有？输出质量差了多少？

理解:
  Prompt 不是摆设。越模糊的 prompt，产出越平庸。
```

### 实验③：切断外部搜索

```
操作:
  把 SerpAPI 调用注释掉
  换成一个固定字符串: mock_data = "东京塔、浅草寺、秋叶原"

观察:
  Planner 用固定数据 vs 真实搜索数据，行程有什么不同？

理解:
  如果 Agent 只靠 LLM 记忆回答，信息可能过时、不准确
```

### 实验④：改中文 Prompt

```
操作:
  把所有 prompt 从英文改成中文
  （Researcher 的 system prompt、Planner 的 system prompt 都改）

观察:
  搜索 query 变了吗？输出行程的语言变了吗？

理解:
  Prompt 的语言影响模型行为，特别是搜索时用什么语言
```

### 实验⑤：只用 Researcher，去掉 Planner

```
操作:
  只保留 Researcher，让 Researcher 直接输出最终行程

观察:
  Researcher 能写出好行程吗？和双 Agent 版本差在哪？

理解:
  分工的价值 → 专注搜索的人不一定擅长组织和写作
```

### 📊 实验记录模板

| 实验 | 改了什么 | 预期变化 | 实际变化 | 学到什么 |
|------|----------|----------|----------|----------|
| ① | | | | |
| ② | | | | |
| ③ | | | | |
| ④ | | | | |
| ⑤ | | | | |

---

## 第4步：自己加功能（预计1-2小时）

### 目标：添加「美食推荐」模块

```
需求:
  - 在 sidebar 加一个 checkbox: ☑ 美食推荐
  - 如果勾选，Researcher 多搜一次 "{destination} 必吃餐厅"
  - Planner 在每天行程里增加一段"🍽️ 今日美食推荐"
```

### 4.1 修改 UI（练习 Streamlit）

```python
# 在 sidebar 里加一行
enable_food = st.sidebar.checkbox("🍽️ 美食推荐")
```

提示：去看 `travel_agent.py` 里 `st.sidebar` 开头的那些行，照猫画虎。

### 4.2 修改 Researcher（练习 Agent 搜索逻辑）

```python
# 在 Researcher 的搜索逻辑里加：
if enable_food:
    food_query = f"{destination} 必吃餐厅 推荐"
    food_results = search(food_query)
    # 把 food_results 追加到传给 Planner 的数据里
```

提示：先找到原本搜索是怎么构造 query 的，然后模仿着加一个新 query。

### 4.3 修改 Planner（练习输出格式控制）

```python
# 在 Planner 的 prompt 里加：
# "如果提供了美食数据，每天推荐 2-3 家当地特色餐厅"
```

提示：不改变代码结构，只改 prompt 就能控制输出。

### 4.4 验证

```
1. 不勾选美食 → 行程和原来一样，没有美食相关内容
2. 勾选美食   → 每天多了"🍽️ 今日美食推荐"，且内容是搜索来的
3. 交叉验证   → 挑一家推荐餐厅去 Google 验证，看是不是真实存在的
```

---

## 🎯 学会了吗？自检清单

用大白话回答以下三个问题：

### Q1: Researcher + Planner 是怎么合作的？

```
你的答案（用自己的话，不用术语）:
```

### Q2: 如果让你从零做一个「AI 电影推荐 Agent」，你会设计几个 Agent？各自负责什么？

```
你的答案:
```

### Q3: 为什么这个项目要分两个 Agent，不是一个 Agent 全做了？

```
你的答案:
```

**三个都能用自己的话讲清楚 → 这个项目你学会了**
**讲不清楚 → 回去再做一轮破坏实验，重点做实验①和实验⑤**

---

## 🚪 通关后下一步

| 如果你对...感兴趣 | 下一个项目 | 在你的大仓库里找 |
|-------------------|-----------|-----------------|
| 多步骤 Pipeline | 🎙️ AI Blog to Podcast | `starter_ai_agents/ai_blog_to_podcast/` |
| Agent 操作数据 | 📊 AI Data Analysis Agent | `starter_ai_agents/ai_data_analysis/` |
| RAG 检索增强 | Simple RAG → Agentic RAG | `rag_tutorials/` |
| 多 Agent 团队 | AI Finance Agent Team | `ai_agent_tutorials/ai_finance_agent_team/` |

---

> 📅 更新日期: 2026-06-10
> 💡 核心原则: **亲手敲，亲手改，亲手破坏，亲手修。** AI 只是你的技术顾问，不是你的手。
