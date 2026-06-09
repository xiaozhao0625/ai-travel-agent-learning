# AI Travel Agent — 快速上手指南

## 环境准备

### 1. 确认 Python 版本

```bash
python --version
# 需要 Python 3.10 或更高版本
```

### 2. 获取必需的 API Keys

#### OpenAI API Key
1. 访问 [platform.openai.com](https://platform.openai.com)
2. 注册/登录账号
3. 进入 API Keys 页面，创建新 Key
4. 新用户有 $5 免费额度

#### SerpAPI Key
1. 访问 [serpapi.com](https://serpapi.com)
2. 注册账号
3. 在 Dashboard 获取 API Key
4. 免费套餐: 100 次搜索/月

### 3. (可选) 安装 Ollama 本地模式

1. 访问 [ollama.com](https://ollama.com) 下载安装
2. 拉取模型:
```bash
ollama pull llama3.1    # Meta Llama 3.1 (推荐)
# 或
ollama pull qwen2.5     # 通义千问 (中文更佳)
```

---

## 运行项目

### 方式一: 云端版（OpenAI + SerpAPI）

```bash
# 1. Clone 仓库
git clone https://github.com/Shubhamsaboo/awesome-llm-apps.git
cd awesome-llm-apps/starter_ai_agents/ai_travel_agent

# 2. 安装依赖
pip install -r requirements.txt

# 3. 设置环境变量
# Windows (PowerShell)
$env:OPENAI_API_KEY="sk-..."
$env:SERPAPI_API_KEY="..."

# Mac/Linux (Terminal)
export OPENAI_API_KEY="sk-..."
export SERPAPI_API_KEY="..."

# 4. 启动
streamlit run travel_agent.py

# 5. 浏览器打开 http://localhost:8501
```

### 方式二: 本地版（Ollama，零 API 费用）

```bash
# 确保 Ollama 已安装并运行
ollama serve  # 如未自动启动

# 拉取模型
ollama pull llama3.1

# 启动本地版
streamlit run local_travel_agent.py
```

---

## 使用教程

### 界面说明

Streamlit 界面分为两部分:

| 区域 | 内容 |
|------|------|
| **左侧边栏** | 参数输入区 |
| **主区域** | 行程展示区 + 下载按钮 |

### 输入参数

1. **目的地** — 城市或国家名，如 "Tokyo, Japan" 或 "东京"
2. **旅行天数** — 1-30 天
3. **预算** — 经济 / 标准 / 豪华
4. **兴趣偏好** — 多选: 美食、文化、购物、自然、冒险...
5. **出发日期** — 日历选择器

### 输出内容

点击"生成行程"后:
- 逐日详细行程（上午/下午/晚上）
- 餐厅推荐
- 交通建议
- 预估花费
- **[下载日历]** 按钮 → 导入 Google Calendar

---

## 常见问题

### Q: OpenAI API 报错 "insufficient_quota"
**A:** 检查账户余额，新账号有 $5 免费额度。充值或等待额度重置。

### Q: SerpAPI 搜索返回空
**A:** 检查是否超出免费配额（100次/月），或切换到 DuckDuckGo。

### Q: Ollama 运行很慢
**A:** 本地推理依赖 GPU。如果只有 CPU:
- 使用较小的模型: `ollama pull llama3.2:1b`
- 或改用云端版

### Q: Streamlit 端口被占用
```bash
streamlit run travel_agent.py --server.port 8502
```

### Q: 生成的行程质量不好
**A:** 尝试:
- 提供更具体的目的地（"东京新宿区" 而非 "东京"）
- 选择具体的兴趣标签
- 换成更强大的模型（GPT-4o → GPT-4-turbo）
- 本地版换成更大的模型（llama3.1:8b → llama3.1:70b）

---

## 下一步

学完这个项目后，建议按以下顺序继续:

1. 🎙️ [AI Blog to Podcast](../ai_blog_to_podcast/) — 学习 TTS + LLM 串联
2. 📊 [AI Data Analysis Agent](../ai_data_analysis/) — Agent 操作数据
3. 🔍 [Agentic RAG with Reasoning](../../rag_tutorials/) — RAG 进阶

---

> 📅 更新日期: 2026-06-10
