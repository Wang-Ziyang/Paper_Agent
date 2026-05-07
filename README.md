# Paper_Agent
A simple tool for reading, analyzing, organizing and checking a thesis paper!
# Paper Agent 工具集

这是一个面向科研论文工作的多 Agent 工具集，包含论文精读、目标期刊写作风格学习、论文框架构建、初稿严格检查四个子 Agent，并提供一个类似菜单式工具箱的统一命令入口 `paper`。

项目当前主要服务于能源、热科学、微纳尺度传热、界面热输运、TDTR、材料/半导体相关论文工作流。你也可以通过修改各子目录中的 `*_SKILL.md`、`*_TEMPLATE.md` 和 `*_CHUNK_SCHEMA.md`，把它扩展到其他学科方向。

## 功能总览

| 命令 | Agent | 作用 | 默认大模型后端 |
| --- | --- | --- | --- |
| `paper read` | Paper Reading Agent | 读取 PDF/DOCX/TXT/MD 论文，生成结构化中文精读报告 | Ollama |
| `paper learn` | Journal Style Agent | 从目标期刊参考论文中蒸馏写作风格、章节逻辑和表达模式 | DashScope API，可切换 Ollama |
| `paper construct` | Article Outline Agent | 从碎片化研究材料中构建新论文框架、大纲和图表规划 | DashScope API，可切换 Ollama |
| `paper check` | Check Paper Agent | 对论文初稿进行投稿前严格审查，给出问题清单和修改路线图 | DashScope API，可切换 Ollama |

推荐使用流程：

```text
参考论文/目标期刊样例
        |
        v
learn：学习目标期刊写作风格
        |
        v
construct：结合自己的实验/模拟/想法材料生成论文框架
        |
        v
撰写论文初稿
        |
        v
check：投稿前严格检查

read 可独立使用，用于日常论文精读和文献调研。
```

## 项目结构

```text
paper/
├── Paper_Agent/
│   └── paper_cli_launcher/
│       ├── paper.bat              # Windows 命令入口
│       ├── paper.py               # 统一启动器
│       ├── paper_config.json      # 四个 Agent 的路径配置
│       └── README.md
├── test-read/
│   ├── SKILL.md
│   ├── TEMPLATE.md
│   ├── CHUNK_SCHEMA.md
│   ├── paper/                     # read agent 默认输入目录
│   ├── outputs/                   # read agent 默认输出目录
│   ├── requirements.txt
│   └── scripts/paper_agent.py
├── test-learn/
│   ├── STYLE_SKILL.md
│   ├── STYLE_TEMPLATE.md
│   ├── STYLE_CHUNK_SCHEMA.md
│   ├── refs/                      # learn agent 默认输入目录
│   ├── outputs_style/             # learn agent 默认输出目录
│   ├── requirements.txt
│   └── scripts/journal_style_agent.py
├── test-construct/
│   ├── OUTLINE_SKILL.md
│   ├── OUTLINE_TEMPLATE.md
│   ├── OUTLINE_CHUNK_SCHEMA.md
│   ├── input/                     # construct agent 默认输入目录
│   ├── outputs_outline/           # construct agent 默认输出目录
│   ├── requirements.txt
│   └── scripts/article_outline_agent.py
└── test-check/
    ├── CHECK_SKILL.md
    ├── CHECK_TEMPLATE.md
    ├── CHECK_CHUNK_SCHEMA.md
    ├── input/                     # check agent 默认输入目录
    ├── outputs_check/             # check agent 默认输出目录
    ├── requirements.txt
    └── scripts/check_paper_agent.py
```

## 环境安装

建议使用独立 Conda 环境：

```powershell
conda create -n paper_agent python=3.11 -y
conda activate paper_agent
```

分别安装各 Agent 依赖：

```powershell
cd /d D:\path\to\paper\test-read
pip install -r requirements.txt

cd /d D:\path\to\paper\test-learn
pip install -r requirements.txt

cd /d D:\path\to\paper\test-construct
pip install -r requirements.txt

cd /d D:\path\to\paper\test-check
pip install -r requirements.txt
```

其中常用依赖包括：

| 依赖 | 用途 |
| --- | --- |
| `requests` | 调用 Ollama HTTP API |
| `openai` | 调用 DashScope 或其他 OpenAI-compatible API |
| `pypdf` | 读取 PDF 文本 |
| `PyMuPDF` / `pymupdf` | 提取 PDF 图像证据，主要用于 read/learn |
| `python-docx` | 读取和导出 DOCX |
| `markdown` | Markdown 转 HTML |
| `tqdm` | 进度条 |

## 统一启动器配置

统一启动器位于：

```text
Paper_Agent/paper_cli_launcher/
```

第一次使用前，需要检查两个文件。

### 1. 修改 `paper_config.json`

`workspace` 必须指向包含 `test-read`、`test-learn`、`test-construct`、`test-check` 的项目根目录。

示例：

```json
{
  "workspace": "D:/path/to/paper",
  "agents": {
    "read": {
      "folder": "test-read",
      "script": "scripts/paper_agent.py",
      "label": "Read and analyze papers"
    },
    "learn": {
      "folder": "test-learn",
      "script": "scripts/journal_style_agent.py",
      "label": "Learn target-journal writing style from refs"
    },
    "construct": {
      "folder": "test-construct",
      "script": "scripts/article_outline_agent.py",
      "label": "Construct manuscript framework from fragments"
    },
    "check": {
      "folder": "test-check",
      "script": "scripts/check_paper_agent.py",
      "label": "Check a manuscript draft critically"
    }
  }
}
```

Windows 路径可以写成：

```json
"workspace": "G:/Ph.D_data/Agent_Test/paper"
```

建议在 JSON 中使用 `/`，减少反斜杠转义问题。

### 2. 修改 `paper.bat`

`paper.bat` 里有一行 Python 解释器路径：

```bat
set "PYTHON_EXE=C:\Users\WJY\.conda\envs\paper_agent\python.exe"
```

请改成你自己电脑上的 Conda 环境 Python 路径。可以用下面命令查看：

```powershell
conda activate paper_agent
where python
```

然后把输出路径填到 `PYTHON_EXE`。

## 把 `paper.bat` 写进环境变量

这样做之后，你可以在任意目录直接运行 `paper`。

### 方法一：PowerShell 推荐方式

把启动器目录加入当前用户的 `Path`：

```powershell
$launcher = "D:\path\to\paper\Paper_Agent\paper_cli_launcher"
$userPath = [Environment]::GetEnvironmentVariable("Path", "User")
[Environment]::SetEnvironmentVariable("Path", "$userPath;$launcher", "User")
```

然后关闭并重新打开 PowerShell，测试：

```powershell
paper --show-config
paper
```

### 方法二：CMD / setx

```cmd
setx PATH "%PATH%;D:\path\to\paper\Paper_Agent\paper_cli_launcher"
```

重新打开终端后运行：

```cmd
paper --show-config
paper
```

注意：`setx` 会写入用户环境变量，但它读取的是当前展开后的 `PATH`，极长 PATH 可能被截断；更推荐使用 PowerShell 方法。

## 基本使用

打开菜单：

```powershell
paper
```

查看当前配置：

```powershell
paper --show-config
```

直接运行某个 Agent：

```powershell
paper read
paper learn
paper construct
paper check
```

给子 Agent 传参数：

```powershell
paper read --paper-file "D:\papers\example.pdf" --verbose
paper learn --refs-dir "D:\papers\target_journal_refs" --project-info "D:\project\idea.md"
paper construct --input-dir "D:\project\fragments" --style-report "D:\project\journal_style_report.md"
paper check --manuscript-file "D:\project\draft.docx" --project-info "D:\project\project_info.txt"
```

只打印将要执行的命令，不真正运行：

```powershell
paper read --dry-run --paper-file "D:\papers\example.pdf"
```

## 本地大模型用法：Ollama

本项目的本地模型调用基于 Ollama HTTP API，默认地址：

```text
http://127.0.0.1:11434
```

启动 Ollama 并确认模型存在：

```powershell
ollama list
ollama run gemma4:26b
```

如果你使用其他模型，例如 `qwen2.5:32b` 或其他本地模型，运行时传入 `--model` 或对应参数即可。

### read agent 使用本地模型

`read` 目前只实现了 Ollama 后端：

```powershell
paper read --model gemma4:26b --host http://127.0.0.1:11434
```

### learn agent 使用本地模型

```powershell
paper learn --backend ollama --model gemma4:26b --host http://127.0.0.1:11434
```

### construct agent 使用本地模型

```powershell
paper construct --backend ollama --model gemma4:26b --host http://127.0.0.1:11434
```

### check agent 使用本地模型

```powershell
paper check --backend ollama --ollama-model gemma4:26b --ollama-host http://127.0.0.1:11434
```

本地模型上下文不足或输出不完整时，可以调大分块策略或生成长度，例如：

```powershell
paper read --chunk-chars 3000 --num-ctx 16000 --final-num-predict 7000
paper check --chunk-chars 3500 --final-num-predict 8000
```

## API 用法：DashScope / OpenAI-compatible

`learn`、`construct`、`check` 默认支持 DashScope Qwen API，并通过 OpenAI SDK 的兼容接口调用。

先设置 API Key。临时设置，仅当前 PowerShell 窗口有效：

```powershell
$env:DASHSCOPE_API_KEY="your_api_key_here"
```

永久写入当前用户环境变量：

```powershell
[Environment]::SetEnvironmentVariable("DASHSCOPE_API_KEY", "your_api_key_here", "User")
```

重新打开终端后测试。

默认 API 地址：

```text
https://dashscope.aliyuncs.com/compatible-mode/v1
```

默认模型：

```text
qwen3.6-plus
```

### learn agent 使用 API

```powershell
paper learn --backend dashscope --model qwen3.6-plus
```

也可以指定 API 地址和环境变量名：

```powershell
paper learn --backend dashscope --base-url "https://dashscope.aliyuncs.com/compatible-mode/v1" --api-key-env DASHSCOPE_API_KEY
```

### construct agent 使用 API

```powershell
paper construct --backend dashscope --model qwen3.6-plus
```

### check agent 使用 API

```powershell
paper check --backend dashscope --model qwen3.6-plus
```

如果你使用其他 OpenAI-compatible 服务，可以把 `--base-url`、`--model`、`--api-key-env` 换成对应服务的配置。注意：`read` agent 当前没有 API backend 参数，需要使用 Ollama，或后续在代码中扩展 API Client。

## Agent 细节

### 1. Paper Reading Agent：论文精读

目录：

```text
test-read/
```

入口脚本：

```text
test-read/scripts/paper_agent.py
```

核心提示词和模板：

| 文件 | 作用 |
| --- | --- |
| `SKILL.md` | 定义专家角色、证据优先、机制解释、范围边界、中文输出等规则 |
| `TEMPLATE.md` | 定义最终精读报告结构，包括论文基本信息、摘要重构、技术路线、方法论、图表解读、研究洞察等 |
| `CHUNK_SCHEMA.md` | 分块证据卡格式说明，可作为长文分块分析的扩展参考；当前脚本主要使用内置分块提示逻辑 |

默认输入：

```text
test-read/paper/
```

支持格式：

```text
.pdf, .docx, .txt, .md
```

常用输入参数：

| 参数 | 说明 |
| --- | --- |
| `--paper-dir` | 批量读取论文的目录 |
| `--paper-file` | 只分析单篇论文 |
| `--skill` | 指定 `SKILL.md` |
| `--template` | 指定 `TEMPLATE.md` |
| `--output-dir` | 指定输出目录 |
| `--model` | Ollama 模型名 |
| `--host` | Ollama 服务地址 |
| `--max-papers` | 最多处理多少篇，`0` 表示不限制 |
| `--non-recursive` | 不递归读取子目录 |
| `--no-html` | 不导出 HTML |
| `--no-docx` | 不导出 DOCX |
| `--no-resume` | 不使用已有中间结果续跑 |

默认输出：

```text
test-read/outputs/<论文文件名>/
```

每篇论文输出：

| 输出文件 | 说明 |
| --- | --- |
| `extracted_text.txt` | 从论文中抽取并清洗后的全文文本 |
| `chunk_notes.json` | 长论文分块分析的中间证据笔记 |
| `analysis.md` | 主要 Markdown 精读报告 |
| `analysis.html` | 浏览器友好的 HTML 报告 |
| `analysis.docx` | 可编辑和分享的 Word 报告 |
| `visual_evidence.json` | 图表引用、图表证据和图像分析结果 |
| `figure_assets/` | 从 PDF 中提取的图像素材 |
| `metadata.json` | 输入、模型、分块参数、输出路径等元数据 |

失败任务会记录到：

```text
test-read/outputs/failed_jobs.json
```

示例：

```powershell
paper read --paper-file "D:\papers\paper1.pdf" --model gemma4:26b --verbose
```

### 2. Journal Style Agent：目标期刊风格学习

目录：

```text
test-learn/
```

入口脚本：

```text
test-learn/scripts/journal_style_agent.py
```

核心提示词和模板：

| 文件 | 作用 |
| --- | --- |
| `STYLE_SKILL.md` | 定义“蒸馏目标期刊写作风格”的角色和规则，不复制原文，只提炼结构、修辞和章节逻辑 |
| `STYLE_TEMPLATE.md` | 定义最终风格报告结构，包括 Introduction、Methods、Results、Conclusion、图表、引用策略和写作清单 |
| `STYLE_CHUNK_SCHEMA.md` | 定义每个参考论文分块应提取的写作功能、可迁移模式和章节观察 |

默认输入：

```text
test-learn/refs/
```

支持格式：

```text
.pdf, .docx, .txt, .md
```

可选输入：

| 输入 | 说明 |
| --- | --- |
| `--project-info your_plan.md` | 你的拟投稿论文设想、研究主题、目标期刊、主要贡献等 |

常用参数：

| 参数 | 说明 |
| --- | --- |
| `--refs-dir` | 参考论文目录 |
| `--output-dir` | 输出目录 |
| `--backend` | `dashscope` 或 `ollama` |
| `--model` | 模型名 |
| `--base-url` | API 地址，DashScope 默认兼容 OpenAI API |
| `--api-key-env` | API Key 所在环境变量名，默认 `DASHSCOPE_API_KEY` |
| `--host` | Ollama 地址，仅 `--backend ollama` 时使用 |
| `--chunk-chars` | 分块长度 |
| `--no-resume` | 不使用已有中间结果续跑 |

默认输出：

```text
test-learn/outputs_style/
```

输出文件：

| 输出文件 | 说明 |
| --- | --- |
| `journal_style_report.md` | 目标期刊写作风格蒸馏报告 |
| `journal_style_report.html` | HTML 版本报告 |
| `reference_style_profiles.json` | 所有参考论文的风格画像汇总 |
| `reference_profiles/<参考论文名>/extracted_text.txt` | 单篇参考论文抽取文本 |
| `reference_profiles/<参考论文名>/style_chunk_notes.json` | 单篇参考论文分块风格笔记 |
| `reference_profiles/<参考论文名>/style_profile.md` | 单篇参考论文风格画像 |
| `metadata.json` | 模型、参考文献数量、输出路径等信息 |

示例：

```powershell
paper learn --refs-dir "D:\refs\nature_energy" --project-info "D:\project\project_info.md" --backend dashscope
```

### 3. Article Outline Agent：论文框架构建

目录：

```text
test-construct/
```

入口脚本：

```text
test-construct/scripts/article_outline_agent.py
```

核心提示词和模板：

| 文件 | 作用 |
| --- | --- |
| `OUTLINE_SKILL.md` | 定义从碎片化研究材料构建论文主线、章节结构和图表规划的规则 |
| `OUTLINE_TEMPLATE.md` | 定义最终论文框架报告结构，包括标题、摘要、Introduction、Methods、Results、Conclusions、图表规划、风险与补充建议 |
| `OUTLINE_CHUNK_SCHEMA.md` | 定义从每个材料分块中提取研究对象、创新点、方法、结果、机制、缺失信息等 |

默认输入：

```text
test-construct/input/
```

支持格式：

```text
.pdf, .docx, .txt, .md, .html, .htm
```

推荐放入的材料：

| 输入材料 | 说明 |
| --- | --- |
| 实验记录 | 样品、测试条件、数据现象、误差说明 |
| 模拟/建模说明 | 方程、边界条件、参数、假设、验证 |
| 初步结果 | 图表、趋势、物理机制、对比分析 |
| 文献笔记 | 领域背景、已有方法、研究空白 |
| `project_info.txt` | 可选，描述论文主题、目标期刊、核心贡献和当前痛点 |
| `--style-report` | 可选，接入 learn agent 生成的 `journal_style_report.md` |

常用参数：

| 参数 | 说明 |
| --- | --- |
| `--input-dir` | 碎片化材料目录 |
| `--output-dir` | 输出目录 |
| `--project-info` | 单独指定项目说明文件 |
| `--style-report` | 指定目标期刊风格报告 |
| `--backend` | `dashscope` 或 `ollama` |
| `--model` | 模型名 |
| `--base-url` | API 地址 |
| `--api-key-env` | API Key 环境变量名 |
| `--host` | Ollama 地址 |
| `--recursive` | 递归读取输入目录 |
| `--no-resume` | 不使用已有中间结果续跑 |

默认输出：

```text
test-construct/outputs_outline/
```

输出文件：

| 输出文件 | 说明 |
| --- | --- |
| `manuscript_outline.md` | 新论文框架设计报告 |
| `manuscript_outline.html` | HTML 版本报告 |
| `fragment_notes.json` | 输入材料分块提炼出的证据和结构笔记 |
| `metadata.json` | 输入文件、项目说明、风格报告路径、输出路径等元数据 |

示例：

```powershell
paper construct --input-dir "D:\project\fragments" --project-info "D:\project\project_info.txt" --style-report "D:\project\journal_style_report.md"
```

### 4. Check Paper Agent：论文初稿严格检查

目录：

```text
test-check/
```

入口脚本：

```text
test-check/scripts/check_paper_agent.py
```

核心提示词和模板：

| 文件 | 作用 |
| --- | --- |
| `CHECK_SKILL.md` | 定义审稿人式严格检查角色，覆盖创新性、研究问题、文献定位、方法严谨性、数据可信度、讨论深度和语言表达 |
| `CHECK_TEMPLATE.md` | 定义最终检查报告结构，包括总体判断、问题清单、逐章节修改路线图和投稿前检查清单 |
| `CHECK_CHUNK_SCHEMA.md` | 定义长稿件分块审查时每个 chunk 应提取的问题证据和严重程度 |

默认输入：

```text
test-check/input/
```

支持格式：

```text
.docx, .txt, .md
```

可选输入：

| 输入 | 说明 |
| --- | --- |
| `--manuscript-file` | 指定单篇初稿文件 |
| `--project-info` | 目标期刊、论文目标、担心的问题、希望重点检查的内容 |

常用参数：

| 参数 | 说明 |
| --- | --- |
| `--input-dir` | 初稿输入目录 |
| `--manuscript-file` | 指定单个初稿文件 |
| `--project-info` | 项目说明 |
| `--output-dir` | 输出目录 |
| `--backend` | `dashscope` 或 `ollama` |
| `--model` | DashScope/Qwen 模型 |
| `--ollama-model` | Ollama 模型 |
| `--ollama-host` | Ollama 服务地址 |
| `--chunk-chars` | 长稿件分块长度 |
| `--single-shot-chars` | 低于该长度时一次性审查 |
| `--recursive` | 递归读取输入目录 |
| `--no-resume` | 不使用已有中间结果续跑 |

默认输出：

```text
test-check/outputs_check/
```

输出文件：

| 输出文件 | 说明 |
| --- | --- |
| `check_report.md` | 论文初稿严格检查报告 |
| `check_report.html` | HTML 版本检查报告 |
| `extracted_manuscript_text.txt` | 从初稿中抽取出的正文文本 |
| `review_notes.json` | 长稿件分块审查笔记 |
| `review_notes.partial.json` | 中断或续跑时的临时笔记 |
| `metadata.json` | 输入文件、文本长度、分块数量、输出路径等元数据 |

示例：

```powershell
paper check --manuscript-file "D:\project\draft.docx" --project-info "D:\project\project_info.txt" --backend dashscope
```

## 直接运行子 Agent

不使用统一启动器时，也可以进入对应目录直接运行脚本。

```powershell
cd /d D:\path\to\paper\test-read
python scripts\paper_agent.py --paper-file "D:\papers\paper1.pdf"

cd /d D:\path\to\paper\test-learn
python scripts\journal_style_agent.py --refs-dir refs --backend dashscope

cd /d D:\path\to\paper\test-construct
python scripts\article_outline_agent.py --input-dir input --backend dashscope

cd /d D:\path\to\paper\test-check
python scripts\check_paper_agent.py --manuscript-file "D:\project\draft.docx"
```

## 常见问题

### `paper` 不是内部或外部命令

说明 `paper_cli_launcher` 目录还没有加入 `Path`，或终端没有重启。重新执行“把 `paper.bat` 写进环境变量”的步骤，然后打开新终端。

### 找不到 Python executable

检查 `paper.bat` 中的：

```bat
set "PYTHON_EXE=..."
```

确保路径指向真实存在的 `python.exe`。

### 找不到 Agent folder 或 Agent script

检查 `paper_config.json` 中的 `workspace`。它必须指向项目根目录，也就是包含 `test-read`、`test-learn`、`test-construct`、`test-check` 的目录。

### DashScope API key was not found

需要设置：

```powershell
$env:DASHSCOPE_API_KEY="your_api_key_here"
```

或永久写入：

```powershell
[Environment]::SetEnvironmentVariable("DASHSCOPE_API_KEY", "your_api_key_here", "User")
```

### Failed to reach Ollama

确认 Ollama 已启动、模型已下载、端口正确：

```powershell
ollama list
ollama run gemma4:26b
```

如果 Ollama 不在默认地址，运行时传入对应 `--host`、`--ollama-host`。

## 
这里用的我自己的路径和api配置，需要替换一下
