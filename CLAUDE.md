# DevForgeAI 项目指南

## 项目简介

DevForgeAI 是一个基于 LangGraph 和 Docker 的多智能体编程框架。用户只需描述需求，系统会自动完成规划、编码、测试的完整流程。

## 技术栈

- **后端**: Python 3.10+, LangGraph, FastAPI
- **前端**: React 18, TypeScript, Vite
- **沙盒**: Docker
- **LLM**: 支持 OpenAI、Anthropic、Ollama、DeepSeek

## 开发命令

```bash
# 安装依赖
pip install -r requirements.txt

# 运行测试
pytest

# 启动 Web UI（推荐）
streamlit run web_ui.py

# 启动 React 前端 + FastAPI 后端
python api_server.py

# CLI 模式
python run.py
```

前端开发模式：
```bash
cd frontend
npm install
npm run dev  # 启动 Vite 开发服务器，端口 3000
```

## 项目结构

```
DevForgeAI/
├── run.py                      # LangGraph 工作流入口
├── web_ui.py                   # Streamlit Web UI
├── api_server.py               # FastAPI 后端 + React 前端服务
├── requirements.txt            # Python 依赖
│
├── src/
│   ├── agents/
│   │   ├── Planner.py          # 规划师：理解需求，制定计划
│   │   ├── Coder.py            # 工程师：编写和修改代码
│   │   ├── Reviewer.py         # 审查员：分析错误，给出修复建议
│   │   └── Sandbox.py          # 沙盒：Docker 隔离测试
│   │
│   ├── core/
│   │   ├── config.py           # 全局配置
│   │   ├── context_manager.py  # 上下文管理（三层记忆策略）
│   │   ├── state.py            # 状态定义
│   │   ├── llm_engine.py       # LLM 初始化和调用
│   │   ├── logger.py           # 日志系统
│   │   ├── repo_map.py         # AST 代码解析
│   │   ├── routing.py          # 工作流路由逻辑
│   │   ├── recovery.py         # 错误恢复和快照
│   │   ├── metrics.py          # 指标收集
│   │   ├── language_support.py # 多语言支持
│   │   └── git_integration.py  # Git 集成
│   │
│   └── tools/
│       └── file_tools.py       # 文件操作工具
│
├── frontend/                   # React 前端
│   └── src/
│       ├── pages/
│       │   ├── ChatPage.tsx    # 聊天页面
│       │   ├── MetricsPage.tsx # 指标面板
│       │   ├── FileBrowserPage.tsx # 文件浏览器
│       │   └── ConfigPage.tsx  # 配置面板
│       └── context/
│           └── AppContext.tsx  # 全局状态管理
│
├── tests/                      # 单元测试
└── workspace/                  # Agent 工作区
```

## 核心功能

### 四个 Agent

| Agent | 职责 |
|-------|------|
| Planner | 理解需求，探索工作区，制定开发计划 |
| Coder | 执行代码修改，读写文件 |
| Sandbox | Docker 隔离测试，自动发现测试文件 |
| Reviewer | 分析测试失败原因，给出修复建议 |

### 上下文管理

三层记忆策略：
- **核心记忆**: 用户需求、执行计划、错误信息（永久保留）
- **工作记忆**: 最近 N 轮对话（滑动窗口）
- **参考记忆**: 历史对话摘要（LLM 压缩）

### 文件操作

- 大文件（>5000字符）返回 AST 结构大纲
- `read_function` / `read_class` 精确定位代码
- `edit_file` 支持精确匹配和模糊匹配
- 每次修改自动备份

## 配置

在 `src/core/.env` 中配置：

```bash
# 选择 LLM 提供商
LLM_PROVIDER=deepseek

# DeepSeek 配置
DEEPSEEK_API_KEY=你的密钥
DEEPSEEK_MODEL=deepseek-chat
DEEPSEEK_BASE_URL=https://api.deepseek.com

# 或者使用 Ollama 本地模型
# OLLAMA_BASE_URL=http://localhost:11434
# OLLAMA_MODEL=qwen2.5-coder
```

## 工作流程

```
用户需求 → Planner 规划 → Coder 编码 → Sandbox 测试
                                    ↑           ↓
                                    └─ Reviewer 审查（失败时）
```

## API 接口

| 接口 | 说明 |
|------|------|
| POST /api/run | 启动工作流 |
| GET /api/run/{thread_id}/events | SSE 事件流 |
| GET /api/run/{thread_id}/state | 获取最终状态 |
| GET /api/files | 工作区文件列表 |
| GET /api/metrics | 指标数据 |
| GET /api/config | 配置信息 |