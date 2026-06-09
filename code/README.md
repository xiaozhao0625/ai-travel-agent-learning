# AI Travel Agent — 项目源码

> ⚠️ 本项目源码来自 [Shubhamsaboo/awesome-llm-apps](https://github.com/Shubhamsaboo/awesome-llm-apps)，位于 `starter_ai_agents/ai_travel_agent/`。

## 源码位置

如果你已经 clone 了 awesome-llm-apps 全仓库，源码在这里：

```
awesome-llm-apps/starter_ai_agents/ai_travel_agent/
├── travel_agent.py           # 云端版（使用 OpenAI API）
├── local_travel_agent.py     # 本地版（使用 Ollama）
├── requirements.txt          # Python 依赖
└── README.MD                 # 官方说明
```

## 两种学习方式

### 方式 A：直接在大仓库里学
```bash
cd awesome-llm-apps/starter_ai_agents/ai_travel_agent
streamlit run local_travel_agent.py
```

### 方式 B：复制出来学（不污染原仓库）
```bash
cp -r awesome-llm-apps/starter_ai_agents/ai_travel_agent ~/my-travel-agent-lab
cd ~/my-travel-agent-lab
streamlit run local_travel_agent.py
```

---

## 文件说明

| 文件 | 用途 | 学习重点 |
|------|------|----------|
| `travel_agent.py` | 云端版，用 GPT-4o + SerpAPI | 理解 Agent 协作模式 |
| `local_travel_agent.py` | 本地版，用 Ollama | 理解模型切换、零成本运行 |
| `requirements.txt` | 依赖清单 | 了解项目依赖哪些库 |
| `README.MD` | 官方文档 | 对照官方说明和自己的理解 |

---

> 💡 本项目代码版权归 Shubhamsaboo/awesome-llm-apps 所有，Apache 2.0 许可证。
