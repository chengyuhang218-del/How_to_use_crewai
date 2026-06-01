# Mycrew

Mycrew 是一个已经验证可运行的 CrewAI 项目，用于自动检索近期 AI 大语言模型相关论文，并生成结构化 Markdown 研究报告。

项目当前使用 DeepSeek 作为大语言模型提供方，使用 Serper 作为联网搜索工具。运行后，CrewAI 会按顺序调度两个 Agent：先由研究员 Agent 联网查找论文，再由报告分析 Agent 整理并输出 `report.md`。

## 功能介绍

- 使用 CrewAI 定义多 Agent 协作流程。
- 使用 DeepSeek `deepseek/deepseek-reasoner` 作为推理模型。
- 使用 `SerperDevTool` 执行联网搜索。
- 通过 YAML 配置管理 Agent 角色、目标、背景和 Task 描述。
- 按顺序执行研究任务和报告任务。
- 自动生成 Markdown 报告文件 `report.md`。
- 支持 CrewAI 标准入口：运行、训练、回放、测试和触发器运行。

## 环境配置

项目要求：

- Python `>=3.10,<3.14`
- uv
- CrewAI `1.14.6`
- crewai-tools `>=1.14.6`
- DeepSeek API Key
- Serper API Key

项目根目录的 `.env` 需要包含以下变量名：

```env
MODEL=...
DEEPSEEK_API_KEY=...
SERPER_API_KEY=...
CREWAI_TRACING_ENABLED=...
```

注意：

- 不要把真实 API Key 写入文档、代码仓库或公开聊天记录。
- 当前 `crew.py` 中实际使用的模型是 `deepseek/deepseek-reasoner`。
- 当前 `crew.py` 通过 `os.getenv("DEEPSEEK_API_KEY")` 读取 DeepSeek Key。
- `SerperDevTool` 会从环境变量中读取 Serper 相关配置。

## 安装步骤

进入项目目录：

```bash
cd /Users/chengyuhang2/mycrew
```

安装 uv：

```bash
pip install uv
```

安装 CrewAI 项目依赖：

```bash
crewai install
```

如果使用项目中已有的虚拟环境，也可以先激活：

```bash
source .venv/bin/activate
```

## 运行方法

在项目根目录运行：

```bash
crewai run
```

也可以使用 `pyproject.toml` 中声明的脚本入口：

```bash
uv run mycrew
```

或：

```bash
uv run run_crew
```

运行成功后，项目根目录会生成或更新：

```text
report.md
```

## 目录结构

```text
mycrew/
├── .env
├── .gitignore
├── AGENTS.md
├── README.md
├── PROJECT_GUIDE.md
├── knowledge/
│   └── user_preference.txt
├── pyproject.toml
├── report.md
├── src/
│   └── mycrew/
│       ├── __init__.py
│       ├── crew.py
│       ├── config/
│       │   ├── agents.yaml
│       │   └── tasks.yaml
│       └── tools/
│           ├── __init__.py
│           └── custom_tool.py
├── tests/
│   └── test1.py
└── uv.lock
```

关键文件说明：

| 文件 | 作用 |
| --- | --- |
| `src/mycrew/crew.py` | 定义 LLM、搜索工具、Agent、Task 和 Crew 执行流程 |
| `src/mycrew/main.py` | 定义运行入口，包括 `run`、`train`、`replay`、`test`、`run_with_trigger` |
| `src/mycrew/config/agents.yaml` | 配置研究员 Agent 和报告分析 Agent |
| `src/mycrew/config/tasks.yaml` | 配置研究任务和报告生成任务 |
| `.env` | 存放模型、DeepSeek、Serper、Trace 等环境变量 |
| `report.md` | CrewAI 运行后生成的 Markdown 报告 |
| `PROJECT_GUIDE.md` | 项目详细教程和运行流程说明 |

## 示例输出

项目运行后会输出 `report.md`。当前示例报告的主题是近期 AI LLM 论文，报告结构大致如下：

```markdown
# Comprehensive Report on Recent Advances in AI Large Language Models

**Period:** July 28-31, 2025
**Scope:** Five most influential LLM research papers released in this window

## 1. Introduction

## 2. Historical Background

## 3. Key Papers and Major Innovations

## 4. Impact on Subsequent Research

## 5. Current Relevance
```

报告内容由两个任务协作生成：

1. `research_task`：使用搜索工具查找近 3 天发布的 AI 论文，并提供发布日期。
2. `reporting_task`：基于研究结果生成完整 Markdown 报告，并写入 `report.md`。

## 更多说明

完整教程请阅读：

```text
PROJECT_GUIDE.md
```
