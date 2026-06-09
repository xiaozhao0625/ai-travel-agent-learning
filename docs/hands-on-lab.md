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

## 第1步：亲手跑起来（预计30分钟）

### 1.1 Clone 仓库

```bash
# 🖐️ 自己敲，别复制我的
git clone https://github.com/Shubhamsaboo/awesome-llm-apps.git
cd awesome-llm-apps/starter_ai_agents/ai_travel_agent
```

### 1.2 读依赖文件

```bash
cat requirements.txt
```

观察：
- 有哪些依赖？
- `agno` 出现了吗？（入门项目不需要它）
- `ics` 是做什么用的？

### 1.3 安装依赖

```bash
pip install -r requirements.txt
```

### 1.4 运行本地模式

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

### ⚡ 预期会遇到的问题

| 报错 | 自己排查方向 |
|------|-------------|
| `ModuleNotFoundError` | `pip install` 漏了哪个？ |
| `Connection refused` | Ollama 启动了吗？`ollama list` 看看 |
| `Address already in use` | 换端口 `streamlit run ... --server.port 8502` |
| 模型回复乱码 | 换 `qwen2.5:14b` 试试 |

**先自己 Google/看文档解决，实在不行再问我。**

---

## 第2步：拆代码（预计1小时）

打开 `travel_agent.py`（云端版），逐段回答以下问题。把你的答案写在注释里。

### 2.1 System Prompt 分析

```python
# 📝 找到 Researcher 的 system prompt，回答：
# 1. Researcher 被定义成了什么角色？
# 2. 它被赋予了什么工具？
# 3. 它被要求输出什么格式？
# 4. 有哪些约束条件？
```

### 2.2 Agent 调用链

```python
# 📝 找到 SerpAPI 调用的代码位置，回答：
# 1. 搜索参数是怎么构造的？
# 2. 搜索了多少次？（不同的 query）
# 3. 搜索结果是怎么格式化后传给 Planner 的？
```

### 2.3 Planner 分析

```python
# 📝 找到 Planner 的代码，回答：
# 1. Planner 接收了什么输入？
# 2. Planner 的 system prompt 和 Researcher 有什么不同？
# 3. 行程是怎么结构化输出的？
```

### 2.4 ICS 日历生成

```python
# 📝 找到 ICS 生成的代码，回答：
# 1. 用了哪个库？
# 2. 日历事件的字段有哪些（标题、时间、地点...）？
# 3. 为什么输出 ICS 比输出纯文本对用户更有价值？
```

### 2.5 Streamlit 交互

```python
# 📝 回答：
# 1. sidebar 放了哪些输入组件？
# 2. main 区域展示了什么内容？
# 3. 用户点击"生成"后触发了什么流程？
```

---

## 第3步：破坏实验（预计1小时）

> 💡 这是**最重要的步骤**。只有搞坏了才知道每部分到底有什么用。

### 实验①：拆掉 Researcher

```
操作: 注释掉 Researcher Agent，直接把用户输入传给 Planner
问题: Planner 还能生成行程吗？行程内容有什么变化？
      → 如果 Researcher 只是搜索，GPT 自己不知道东京有什么景点吗？
      → 为什么要先搜索再规划？
```

### 实验②：劣化 Prompt

```
操作: 把 Researcher 的 system prompt 改成一句话: "You are a helper."
问题: 搜索行为变了没有？结果质量差了多少？
      → Prompt 到底在控制什么？
      → 如果 prompt 不重要，为什么大家花那么多时间写？
```

### 实验③：切换搜索源

```
操作: 把 SerpAPI 搜索注释掉，换成固定字符串: mock_results = "东京塔、浅草寺、秋叶原"
问题: Planner 用固定数据生成的行程和用真实搜索数据有什么不同？
      → 如果 Agent 不搜，只靠 LLM 记忆回答，风险是什么？
```

### 实验④：语言切换

```
操作: 把所有 prompt 改成中文（包括 Researcher 和 Planner）
问题: 搜索 query 变了吗？输出行程的语言变化了吗？
      → 如果用户用中文，但 prompt 是英文，会出现什么？
```

### 实验⑤：去掉 Planner

```
操作: 只保留 Researcher，让它直接输出行程
问题: Researcher 能写出好行程吗？
      → 为什么需要两个 Agent 而不是一个？
      → 分工到底解决了什么问题？
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
# sidebar 里加一行
enable_food = st.sidebar.checkbox("🍽️ 美食推荐")
```

### 4.2 修改 Researcher（练习 Agent 提示词）

```python
# 在 Researcher 的搜索逻辑里加：
if enable_food:
    food_query = f"{destination} 必吃餐厅 推荐"
    food_results = search(food_query)
    # 把 food_results 加到传给 Planner 的数据里
```

### 4.3 修改 Planner（练习输出格式控制）

```python
# 在 Planner 的 prompt 里加：
# "如果提供了美食数据，每天推荐 2-3 家当地特色餐厅"
```

### 4.4 验证

```
1. 不勾选美食 → 行程和原来一样
2. 勾选美食   → 每天多了美食推荐
3. 美食推荐的内容是从搜索来的，不是 GPT 自己编的
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
**讲不清楚 → 回去再做一轮破坏实验**

---

## 🚪 通关后下一步

| 如果你对...感兴趣 | 下一个项目 |
|-------------------|-----------|
| 多步骤 Pipeline | 🎙️ AI Blog to Podcast |
| Agent 操作数据 | 📊 AI Data Analysis Agent |
| RAG 检索增强 | Simple RAG → Agentic RAG |
| 多 Agent 团队 | AI Finance Agent Team |

---

> 📅 创建日期: 2026-06-10
> 💡 核心原则: **亲手敲，亲手改，亲手破坏，亲手修。** AI 只是你的技术顾问，不是你的手。
