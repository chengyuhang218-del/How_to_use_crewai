# Mycrew Project Guide

本文档用于解释 `/Users/chengyuhang2/mycrew` 这个已验证可运行的 CrewAI 项目。目标是帮助读者理解项目结构、运行流程、关键配置和常见问题，不修改任何已有 Python 代码、YAML 配置、DeepSeek 配置或 Serper 配置。

## 1. CrewAI 基础概念

CrewAI 是一个用于构建多 Agent 协作流程的框架。它把复杂任务拆分为多个角色和步骤，让不同 Agent 按照预设流程完成各自任务。

在本项目中，CrewAI 的核心概念包括：

| 概念 | 含义 | 本项目中的位置 |
| --- | --- | --- |
| Agent | 执行任务的智能体，拥有角色、目标、背景、模型和工具 | `agents.yaml`、`crew.py` |
| Task | 交给 Agent 执行的具体任务 | `tasks.yaml`、`crew.py` |
| Tool | Agent 可以调用的外部能力，例如搜索工具 | `crew.py`、`tools/` |
| Crew | 多个 Agent 和 Task 组成的协作团队 | `crew.py` |
| Process | Task 的执行方式，例如顺序执行 | `crew.py` |
| LLM | Agent 使用的大语言模型 | `crew.py`、`.env` |

## 2. Agent 解释

Agent 是 CrewAI 中的工作角色。每个 Agent 通常包含：

- `role`：角色名称。
- `goal`：角色目标。
- `backstory`：背景设定，用于影响 Agent 的回答风格和工作重点。
- `llm`：该 Agent 使用的语言模型。
- `tools`：该 Agent 可调用的工具。
- `verbose`：是否输出详细执行日志。

本项目有两个 Agent。

### 2.1 researcher

配置位置：

```text
src/mycrew/config/agents.yaml
```

代码位置：

```text
src/mycrew/crew.py
```

`researcher` 的角色是 `AI Research Scientist`，目标是围绕 `{topic}` 查找有影响力和权威性的研究论文，识别经典或重要工作，总结关键贡献并解释影响。

在 `crew.py` 中，`researcher` 绑定了：

- DeepSeek LLM。
- `SerperDevTool` 搜索工具。
- `verbose=True`。

因此，`researcher` 是本项目中负责联网检索和初步研究的 Agent。

### 2.2 reporting_analyst

配置位置：

```text
src/mycrew/config/agents.yaml
```

代码位置：

```text
src/mycrew/crew.py
```

`reporting_analyst` 的角色是 `Scientific Literature Analyst`，目标是基于研究结果生成清晰、结构化、技术准确的报告。

在 `crew.py` 中，`reporting_analyst` 绑定了：

- DeepSeek LLM。
- `verbose=True`。

它没有绑定搜索工具，主要职责是根据前一个任务的研究结果进行整理、分析和写作。

## 3. Task 解释

Task 是 CrewAI 中的任务单元。一个 Task 通常包含：

- `description`：任务描述。
- `expected_output`：期望输出。
- `agent`：执行该任务的 Agent。
- `output_file`：可选，指定结果写入文件。

本项目有两个 Task。

### 3.1 research_task

配置位置：

```text
src/mycrew/config/tasks.yaml
```

`research_task` 要求：

- 使用搜索工具。
- 查找最近 3 天发布的 AI 论文。
- 必须提供发布日期。
- 必须先使用搜索工具再回答。
- 如果没有使用搜索工具，则任务不完整。

期望输出是一个基于互联网搜索结果的、关于 `{topic}` 的 5 篇有影响力论文排名列表。

执行者是：

```text
researcher
```

### 3.2 reporting_task

配置位置：

```text
src/mycrew/config/tasks.yaml
```

`reporting_task` 要求基于研究发现生成综合报告，内容包括：

- Historical background
- Key papers
- Major innovations
- Impact on subsequent research
- Current relevance

期望输出是详细 Markdown 报告。

执行者是：

```text
reporting_analyst
```

在 `crew.py` 中，这个任务还设置了：

```python
output_file='report.md'
```

因此最终报告会写入项目根目录的 `report.md`。

## 4. Tool 解释

Tool 是 Agent 可以调用的外部能力。本项目当前实际使用的工具是：

```python
SerperDevTool()
```

定义位置：

```text
src/mycrew/crew.py
```

用途：

- 让 `researcher` 可以联网搜索。
- 支撑 `research_task` 中“必须使用搜索工具”的要求。
- 获取最近论文、发布日期和相关信息。

项目中还存在示例工具文件：

```text
src/mycrew/tools/custom_tool.py
```

该文件定义了一个 `MyCustomTool` 示例类，但当前 `crew.py` 没有导入或使用它。因此当前运行流程依赖的是 `SerperDevTool`，不是 `MyCustomTool`。

## 5. Crew 解释

Crew 是 Agent 和 Task 的组合。它定义“有哪些人、做哪些事、按什么方式执行”。

本项目的 Crew 定义在：

```text
src/mycrew/crew.py
```

核心代码是：

```python
return Crew(
    agents=self.agents,
    tasks=self.tasks,
    process=Process.sequential,
    verbose=True,
)
```

含义：

- `agents=self.agents`：使用通过 `@agent` 装饰器注册的 Agent。
- `tasks=self.tasks`：使用通过 `@task` 装饰器注册的 Task。
- `process=Process.sequential`：任务按顺序执行。
- `verbose=True`：输出详细运行日志。

## 6. Process 解释

Process 是 CrewAI 中的执行流程控制方式。

本项目使用：

```python
Process.sequential
```

这表示任务按声明顺序依次执行：

1. 先执行 `research_task`。
2. 再执行 `reporting_task`。

这种方式适合当前项目，因为报告任务依赖研究任务的结果。如果没有前面的搜索和研究，后面的报告生成就缺少资料来源。

## 7. 当前项目工作流程图

```python
flowchart TD
    A["用户运行 crewai run 或 uv run mycrew"] --> B["main.py: run()"]
    B --> C["设置 inputs: topic=AI LLMs, current_year=当前年份"]
    C --> D["Mycrew().crew().kickoff(inputs=inputs)"]
    D --> E["crew.py 创建 Crew"]
    E --> F["research_task"]
    F --> G["researcher Agent"]
    G --> H["DeepSeek reasoner"]
    G --> I["SerperDevTool 联网搜索"]
    I --> J["论文搜索结果和发布日期"]
    H --> J
    J --> K["reporting_task"]
    K --> L["reporting_analyst Agent"]
    L --> M["DeepSeek reasoner 整理分析"]
    M --> N["输出 report.md"]
```

## 8. crew.py 逐行讲解

文件路径：

```text
src/mycrew/crew.py
```

### 导入模块

```python
import os
```

用于读取环境变量，例如 `DEEPSEEK_API_KEY`。

```python
from crewai import LLM
```

导入 CrewAI 的 LLM 封装，用来配置 DeepSeek 模型。

```python
from crewai import Agent, Crew, Process, Task
```

导入 CrewAI 核心类：

- `Agent`：定义智能体。
- `Crew`：定义团队。
- `Process`：定义执行流程。
- `Task`：定义任务。

```python
from crewai.project import CrewBase, agent, crew, task
```

导入 CrewAI 项目装饰器：

- `@CrewBase`：标记 Crew 项目类。
- `@agent`：注册 Agent 方法。
- `@task`：注册 Task 方法。
- `@crew`：注册 Crew 方法。

```python
from crewai.agents.agent_builder.base_agent import BaseAgent
```

导入 Agent 类型声明，用于标注 `agents` 列表。

```python
from crewai_tools import SerperDevTool
```

导入 Serper 搜索工具。

### 创建工具和模型

```python
search_tool = SerperDevTool()
```

创建 Serper 搜索工具实例，后续会传给 `researcher`。

```python
llm = LLM(
    model="deepseek/deepseek-reasoner",
    api_key=os.getenv("DEEPSEEK_API_KEY")
)
```

创建 CrewAI LLM 实例：

- `model` 指定 DeepSeek reasoner。
- `api_key` 从环境变量 `DEEPSEEK_API_KEY` 读取。
- 文档中不应写入真实 Key。

### 定义 CrewBase 类

```python
@CrewBase
class Mycrew():
    """Mycrew crew"""
```

声明 `Mycrew` 是 CrewAI 项目类。CrewAI 会基于这个类加载配置、Agent 和 Task。

```python
agents: list[BaseAgent]
tasks: list[Task]
```

这是类型标注，表示该类会拥有 Agent 列表和 Task 列表。

### researcher Agent

```python
@agent
def researcher(self) -> Agent:
```

使用 `@agent` 注册一个名为 `researcher` 的 Agent。

```python
return Agent(
    config=self.agents_config['researcher'],
    llm=llm,
    tools=[search_tool],
    verbose=True
)
```

创建 Agent：

- 从 `agents.yaml` 读取 `researcher` 配置。
- 使用 DeepSeek LLM。
- 绑定 Serper 搜索工具。
- 开启详细日志。

### reporting_analyst Agent

```python
@agent
def reporting_analyst(self) -> Agent:
```

使用 `@agent` 注册报告分析 Agent。

```python
return Agent(
    config=self.agents_config['reporting_analyst'],
    llm=llm,
    verbose=True
)
```

创建 Agent：

- 从 `agents.yaml` 读取 `reporting_analyst` 配置。
- 使用 DeepSeek LLM。
- 不绑定搜索工具。
- 开启详细日志。

### research_task

```python
@task
def research_task(self) -> Task:
```

使用 `@task` 注册研究任务。

```python
return Task(
    config=self.tasks_config['research_task'],
)
```

创建 Task，并从 `tasks.yaml` 读取 `research_task` 配置。

### reporting_task

```python
@task
def reporting_task(self) -> Task:
```

使用 `@task` 注册报告任务。

```python
return Task(
    config=self.tasks_config['reporting_task'],
    output_file='report.md'
)
```

创建报告 Task：

- 从 `tasks.yaml` 读取 `reporting_task` 配置。
- 将输出写入 `report.md`。

### crew

```python
@crew
def crew(self) -> Crew:
```

使用 `@crew` 注册 Crew 创建方法。

```python
return Crew(
    agents=self.agents,
    tasks=self.tasks,
    process=Process.sequential,
    verbose=True,
)
```

创建最终 Crew：

- 包含所有注册 Agent。
- 包含所有注册 Task。
- 顺序执行。
- 输出详细日志。

## 9. agents.yaml 讲解

文件路径：

```text
src/mycrew/config/agents.yaml
```

该文件定义两个 Agent 的自然语言配置。

### researcher

```yaml
researcher:
  role: >
    AI Research Scientist
```

表示该 Agent 的身份是 AI 研究科学家。

```yaml
  goal: >
    Find the most influential and authoritative research papers related to {topic},
    identify seminal works, summarize their key contributions, and explain their impact.
```

目标是围绕输入变量 `{topic}` 查找权威论文、识别重要工作、总结贡献并解释影响。

```yaml
  backstory: >
    You are an experienced AI researcher specializing in machine learning,
    deep learning, and large language models...
```

背景设定让该 Agent 更像经验丰富的 AI 研究员，擅长机器学习、深度学习和大语言模型文献研究。

### reporting_analyst

```yaml
reporting_analyst:
  role: >
    Scientific Literature Analyst
```

表示该 Agent 的身份是科学文献分析师。

```yaml
  goal: >
    Produce a clear, structured, and technically accurate report on {topic}
    based on the research findings.
```

目标是基于研究结果生成清晰、结构化、技术准确的报告。

```yaml
  backstory: >
    You are an expert scientific writer who transforms complex research papers
    into concise and easy-to-understand reports...
```

背景设定强调科学写作能力，包括提炼创新、实验结果、优点、局限性和历史意义。

## 10. tasks.yaml 讲解

文件路径：

```text
src/mycrew/config/tasks.yaml
```

该文件定义两个任务。

### research_task

```yaml
research_task:
  description: >
    Use the search tool.
```

明确要求使用搜索工具。

```yaml
    Find AI papers released in the last 3 days.
```

要求查找最近 3 天发布的 AI 论文。

```yaml
    You MUST provide publication dates.
```

要求必须提供发布日期。

```yaml
    You MUST use the search tool before answering.
```

强调回答前必须使用搜索工具。

```yaml
  expected_output: >
    A ranked list of 5 influential papers about {topic}, based on internet search results.
```

期望输出是 5 篇有影响力论文的排名列表。

```yaml
  agent: researcher
```

该任务由 `researcher` 执行。

### reporting_task

```yaml
reporting_task:
  description: >
    Based on the research findings, create a comprehensive report.
```

要求基于研究发现生成综合报告。

```yaml
    Include:
    - Historical background
    - Key papers
    - Major innovations
    - Impact on subsequent research
    - Current relevance
```

规定报告需要覆盖的主要部分。

```yaml
  expected_output: >
    A detailed markdown report.
```

期望输出是详细 Markdown 报告。

```yaml
  agent: reporting_analyst
```

该任务由 `reporting_analyst` 执行。

```yaml
  output_file: report.md
```

该配置中也声明了输出文件。当前 `crew.py` 中同样指定了 `output_file='report.md'`，因此最终结果会写入项目根目录的 `report.md`。

## 11. .env 配置讲解

文件路径：

```text
.env
```

当前项目使用以下环境变量名：

```env
MODEL=...
DEEPSEEK_API_KEY=...
SERPER_API_KEY=...
CREWAI_TRACING_ENABLED=...
```

说明：

| 变量名 | 作用 |
| --- | --- |
| `MODEL` | 通常用于声明模型名称；当前 `crew.py` 中模型名称直接写为 `deepseek/deepseek-reasoner` |
| `DEEPSEEK_API_KEY` | DeepSeek API Key，供 CrewAI LLM 调用 DeepSeek |
| `SERPER_API_KEY` | Serper API Key，供 `SerperDevTool` 执行联网搜索 |
| `CREWAI_TRACING_ENABLED` | 控制 CrewAI Trace 或可观测性相关功能是否启用 |

安全提醒：

- `.env` 已在 `.gitignore` 中忽略。
- 不要提交真实 `.env`。
- 不要在 README、教程、Issue、聊天记录中粘贴真实 Key。

## 12. DeepSeek 接入说明

本项目在 `crew.py` 中通过 CrewAI 的 `LLM` 类接入 DeepSeek：

```python
llm = LLM(
    model="deepseek/deepseek-reasoner",
    api_key=os.getenv("DEEPSEEK_API_KEY")
)
```

关键点：

- 模型名是 `deepseek/deepseek-reasoner`。
- API Key 来自环境变量 `DEEPSEEK_API_KEY`。
- 两个 Agent 都使用同一个 `llm` 实例。
- 当前项目已经验证可运行，不需要修改 DeepSeek 配置。

如果运行时出现 DeepSeek 相关错误，优先检查：

- `.env` 是否存在 `DEEPSEEK_API_KEY`。
- Key 是否有效。
- 当前网络是否能访问 DeepSeek 服务。
- 账户余额、额度或权限是否正常。
- 模型名称是否仍与当前代码保持一致。

## 13. Serper 联网说明

本项目在 `crew.py` 中创建 Serper 搜索工具：

```python
search_tool = SerperDevTool()
```

并将其绑定给 `researcher`：

```python
tools=[search_tool]
```

关键点：

- `researcher` 可以使用 Serper 联网搜索。
- `reporting_analyst` 没有绑定 Serper。
- `research_task` 明确要求必须使用搜索工具。
- `SERPER_API_KEY` 应配置在 `.env` 中。

如果运行时出现搜索相关错误，优先检查：

- `.env` 是否存在 `SERPER_API_KEY`。
- Serper Key 是否有效。
- Serper 账户额度是否充足。
- 当前网络是否能访问 Serper 服务。

## 14. Trace 页面使用说明

项目 `.env` 中存在：

```env
CREWAI_TRACING_ENABLED=...
```

这说明项目可能启用了 CrewAI 的 Trace 或可观测性功能。Trace 页面通常用于查看一次 Crew 运行中的详细轨迹，包括：

- Crew 启动时间。
- Agent 执行顺序。
- Task 输入和输出。
- 工具调用记录。
- LLM 调用过程。
- 错误堆栈或失败位置。

使用建议：

1. 在项目根目录运行 `crewai run`。
2. 观察终端输出中的 Trace、Telemetry、URL 或 Dashboard 提示。
3. 打开终端提示的 Trace 链接。
4. 按任务顺序查看 `research_task` 和 `reporting_task`。
5. 重点检查 `researcher` 是否调用了 `SerperDevTool`。
6. 如果报告内容异常，查看 `reporting_analyst` 接收到的上游研究结果是否完整。

排查时可以重点看：

- `research_task` 是否真的发生了搜索工具调用。
- 搜索结果是否包含发布日期。
- `reporting_task` 是否基于搜索结果生成报告。
- 是否存在 API Key、网络、额度或模型调用错误。

## 15. 常见错误排查

### 15.1 找不到 crewai 命令

现象：

```text
command not found: crewai
```

可能原因：

- 没有安装 CrewAI。
- 没有激活虚拟环境。
- 当前 shell 没有加载对应环境。

处理方式：

```bash
cd /Users/chengyuhang2/mycrew
source .venv/bin/activate
crewai run
```

或重新安装依赖：

```bash
crewai install
```

### 15.2 缺少 DEEPSEEK_API_KEY

现象可能包括：

- LLM 调用失败。
- Authentication error。
- API key missing。

检查：

```bash
cd /Users/chengyuhang2/mycrew
awk -F= '/^[A-Za-z_][A-Za-z0-9_]*=/ {print $1}' .env
```

只确认变量名存在，不要把真实值输出到日志或文档。

### 15.3 缺少 SERPER_API_KEY

现象可能包括：

- 搜索工具调用失败。
- Serper authentication error。
- `research_task` 无法完成搜索。

检查 `.env` 是否存在：

```text
SERPER_API_KEY
```

### 15.4 report.md 没有生成

可能原因：

- Crew 运行中途失败。
- DeepSeek 调用失败。
- Serper 搜索失败。
- 当前运行目录不是项目根目录。

建议：

```bash
cd /Users/chengyuhang2/mycrew
crewai run
ls -la report.md
```

### 15.5 研究任务没有使用搜索工具

`tasks.yaml` 中明确要求：

```text
You MUST use the search tool before answering.
```

如果 Trace 中没有看到搜索调用，重点检查：

- `researcher` 是否绑定了 `tools=[search_tool]`。
- `SERPER_API_KEY` 是否有效。
- 搜索工具调用是否报错。

当前代码中 `researcher` 已绑定搜索工具，因此通常应优先检查 Key、网络和额度。

### 15.6 输出报告内容过时或日期不一致

`main.py` 中默认输入：

```python
inputs = {
    'topic': 'AI LLMs',
    'current_year': str(datetime.now().year)
}
```

`research_task` 要求查找最近 3 天的论文，因此报告内容依赖运行当天的联网搜索结果。如果报告日期与预期不同，检查：

- 实际运行日期。
- Serper 搜索结果。
- Trace 页面中工具返回内容。
- `report.md` 是否为本次运行更新后的文件。

### 15.7 Python 版本不兼容

`pyproject.toml` 要求：

```toml
requires-python = ">=3.10,<3.14"
```

如果 Python 版本过低或过高，可能导致依赖安装或运行失败。

检查：

```bash
python --version
```

### 15.8 依赖版本不一致

`pyproject.toml` 中依赖包括：

```toml
"crewai-tools>=1.14.6"
"crewai[tools]==1.14.6"
```

如果出现导入错误或行为差异，建议使用项目已有锁文件和 CrewAI 安装流程：

```bash
cd /Users/chengyuhang2/mycrew
crewai install
```

## 16. 当前项目运行入口

`pyproject.toml` 声明了以下脚本：

| 命令 | 对应函数 | 作用 |
| --- | --- | --- |
| `mycrew` | `mycrew.main:run` | 正常运行 Crew |
| `run_crew` | `mycrew.main:run` | 正常运行 Crew |
| `train` | `mycrew.main:train` | 训练 Crew |
| `replay` | `mycrew.main:replay` | 从指定任务回放 |
| `test` | `mycrew.main:test` | 测试 Crew |
| `run_with_trigger` | `mycrew.main:run_with_trigger` | 使用触发器 JSON 运行 |

最常用的是：

```bash
crewai run
```

或：

```bash
uv run mycrew
```

## 17. 总结

本项目是一个清晰的两 Agent、两 Task、顺序执行的 CrewAI 文献研究与报告生成项目。

核心流程是：

1. `main.py` 提供默认输入。
2. `crew.py` 创建 DeepSeek LLM 和 Serper 搜索工具。
3. `researcher` 执行 `research_task`，联网搜索近期 AI 论文。
4. `reporting_analyst` 执行 `reporting_task`，生成 Markdown 报告。
5. 最终结果写入 `report.md`。

该项目已经成功运行，因此后续维护时应优先保持现有运行逻辑稳定，只在明确需要时再调整配置或代码。
