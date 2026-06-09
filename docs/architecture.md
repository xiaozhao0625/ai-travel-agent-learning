# AI Travel Agent — 架构设计文档

## 1. 概述

本文档详细描述了 AI Travel Agent 的架构设计，包括数据流、组件交互、Agent 协作模式等。

## 2. 系统架构图

### 2.1 整体架构

```
┌────────────────────────────────────────────────────────────────┐
│                        Streamlit UI Layer                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │ 目的地输入   │  │ 天数/预算   │  │ 偏好设置    │             │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
│                           │                                     │
│                    ┌──────┴──────┐                              │
│                    │  生成行程    │                              │
│                    └──────┬──────┘                              │
└───────────────────────────┼────────────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────────────┐
│                     Agent Orchestration Layer                   │
│                                                                 │
│   ┌─────────────────────┐       ┌─────────────────────┐       │
│   │   Researcher Agent   │       │    Planner Agent     │       │
│   │                      │       │                      │       │
│   │  Model: GPT-4o       │──────▶│  Model: GPT-4o       │       │
│   │  Tool: SerpAPI       │ 搜索  │  Output: 行程文本    │       │
│   │  Role: 信息收集      │ 结果  │  Output: ICS 日历    │       │
│   └─────────────────────┘       └─────────────────────┘       │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────────────┐
│                      External Services Layer                    │
│                                                                 │
│   ┌──────────┐   ┌──────────┐   ┌──────────┐                  │
│   │  SerpAPI  │   │  OpenAI   │   │  Ollama   │                  │
│   │  (Search) │   │  (GPT-4o) │   │  (Local)  │                  │
│   └──────────┘   └──────────┘   └──────────┘                  │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

### 2.2 Agent 内部流程

```
Researcher Agent 内部:
─────────────────────
  用户查询: "东京3天行程，喜欢美食和动漫"
       │
       ▼
  [System Prompt] 你是一个旅行研究专家...
       │
       ▼
  [分解查询]
  ├── "东京 美食推荐 必吃" → SerpAPI
  ├── "东京 动漫圣地 巡礼" → SerpAPI
  ├── "东京 3天 行程"      → SerpAPI
  └── "东京 6月 天气"      → SerpAPI
       │
       ▼
  [合并搜索结果] → 结构化 JSON
       │
       ▼
  传递给 Planner Agent
```

```
Planner Agent 内部:
───────────────────
  接收: 搜索结果 JSON
       │
       ▼
  [System Prompt] 你是一个资深旅行规划师...
       │
       ▼
  [行程组织]
  ├── Day 1: 抵达 + 新宿区域
  │   ├── 上午: 明治神宫
  │   ├── 午餐: 一兰拉面
  │   ├── 下午: 秋叶原动漫街
  │   └── 晚餐: 筑地市场寿司
  ├── Day 2: ...
  └── Day 3: ...
       │
       ▼
  [输出格式]
  ├── Markdown 文本（展示用）
  └── ICS 格式（日历导入用）
```

## 3. 数据流设计

### 3.1 输入数据结构

```python
# 用户输入（来自 Streamlit 侧边栏）
user_input = {
    "destination": str,      # 目的地城市/国家
    "days": int,             # 旅行天数 (1-30)
    "budget": str,           # 预算等级: 经济/标准/豪华
    "interests": List[str],  # 兴趣标签: ["美食", "文化", "购物"...]
    "start_date": date,      # 出发日期
    "travelers": int,        # 出行人数
}
```

### 3.2 中间数据结构

```python
# Researcher 输出的搜索结果
research_result = {
    "attractions": [
        {"name": str, "description": str, "rating": float, "location": str}
    ],
    "weather": {
        "temperature": str, "condition": str, "season": str
    },
    "events": [
        {"name": str, "date": str, "location": str, "description": str}
    ],
    "transportation": [str],  # 交通建议列表
    "food_recommendations": [str],  # 美食推荐
}
```

### 3.3 输出数据结构

```python
# Planner 输出的行程结构
itinerary = {
    "title": str,              # "东京3日动漫美食之旅"
    "summary": str,            # 行程概述
    "days": [
        {
            "day": int,
            "date": str,
            "theme": str,      # "秋叶原动漫巡礼"
            "activities": [
                {
                    "time": str,        # "09:00-12:00"
                    "activity": str,    # "参观秋叶原动漫街"
                    "location": str,    # "秋叶原站"
                    "notes": str,       # "推荐逛Radio会馆"
                }
            ],
            "meals": {
                "breakfast": str,
                "lunch": str,
                "dinner": str,
            }
        }
    ],
    "tips": [str],             # 旅行贴士
    "budget_breakdown": {
        "flights": float,
        "hotels": float,
        "food": float,
        "activities": float,
        "transport": float,
    }
}
```

## 4. 设计模式分析

### 4.1 模式: 分工协作 (Divide & Conquer)

```
问题: 一个 Agent 同时负责搜索+规划 → 输出质量低，容易遗漏

解决方案:
  Researcher → 专注搜索，深度挖掘信息
  Planner    → 专注组织，利用搜索信息写出优质行程

好处:
  - 每个 Agent 的 prompt 更聚焦，表现更好
  - 搜索质量可控（可独立评估 Researcher 的输出）
  - 行程质量可控（可独立评估 Planner 的输出）
```

### 4.2 模式: 工具赋予 (Tool Augmentation)

```
没有工具:
  LLM → 只能基于训练数据"回忆"信息 → 可能过时/不准确

有工具:
  LLM → 调用 SerpAPI → 获取实时准确信息 → 基于真实数据回答
```

### 4.3 模式: 云端/本地双通道

```
这种设计让同一个架构支持两种运行模式:
  - 云端: 最佳质量，按量付费
  - 本地: 零成本，完全隐私

核心: 通过抽象 LLM 接口，使得切换模型只需改一行配置
```

## 5. 扩展方向

### 5.1 短期扩展
- 添加更多搜索工具（Google Maps API、TripAdvisor、Weather API）
- 支持多语言行程输出
- 添加行程分享功能（生成链接）

### 5.2 中期扩展
- 接入 MCP (Model Context Protocol) 标准化工具接口
- 升级为 Agno 多 Agent 团队架构
- 添加用户反馈机制（行程打分 → 优化 prompt）

### 5.3 长期愿景
- 语音输入出行需求 → 自动生成行程
- 实时行程调整（航班延误 → 自动重新规划）
- 多平台发布（iOS/Android App）

---

> 📅 创建日期: 2026-06-10
