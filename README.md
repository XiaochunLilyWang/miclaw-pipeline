# Miclaw 用户体验case分析工作流

基于 Claude Code + Gemini 的 miclaw（小米龙虾AI助手）舆情分析流水线，自动提取用户笔记中的任务执行场景（Sheet1）和主观评价（Sheet2），并输出至飞书表格。

## 文件说明

| 文件 | 说明 |
|------|------|
| `miclaw_pipeline.py` | 分析流水线主脚本 |
| `分析prompt.md` | 分析规则与场景判断标准 |
| `能力分类.md` | 能力标签分类体系 |
| `.env.example` | 环境变量模板 |
| `.claude/commands/miclaw-analyze.md` | Claude Code `/miclaw-analyze` 技能 |

## 环境准备

**依赖**

```bash
pip install openai openpyxl python-dotenv requests
```

**配置密钥**

复制 `.env.example` 为 `.env`，填入 API Key：

```
MICLAW_GEMINI_API_KEY=your_api_key_here
```

## Claude Code 技能安装

技能文件位于 `.claude/commands/miclaw-analyze.md`，支持两种安装方式：

**方式一：项目级（默认）**

将本仓库文件夹作为 Claude Code 工作目录打开，即可使用 `/miclaw-analyze`。仅在该目录下有效。

**方式二：用户级（推荐，任意目录可用）**

将技能文件复制到用户级命令目录，之后无论在哪个项目下都能调用：

```bash
# Windows
copy ".claude\commands\miclaw-analyze.md" "%USERPROFILE%\.claude\commands\miclaw-analyze.md"

# macOS / Linux
cp .claude/commands/miclaw-analyze.md ~/.claude/commands/miclaw-analyze.md
```

复制后重启 Claude Code，在任意工作目录输入 `/miclaw-analyze` 即可启动分析向导。

## 使用方式

### 方式一：Claude Code 技能（推荐）

在 Claude Code 中输入：

```
/miclaw-analyze
```

按提示操作，支持小红书 JSON、Excel（小米社区/酷安）两种数据源。

### 方式二：命令行直接调用

**检查 Excel 列信息**

```bash
python miclaw_pipeline.py inspect "数据文件.xlsx"
```

**运行分析**

```bash
# 小红书 JSON（自动按"是否分析=1"过滤）
python miclaw_pipeline.py run "data.json" --source-label 小红书

# Excel 文件，指定过滤条件
python miclaw_pipeline.py run "data.xlsx" --source-label 小米社区 \
  --filter-col "类型" --filter-val "产品体验,功能需求"

# 断点续跑
python miclaw_pipeline.py run "data.xlsx" --source-label 小米社区 --resume
```

**为 Sheet1 Cases 打能力标签**

```bash
python miclaw_pipeline.py tag-abilities pipeline_results.json
```

**导出为飞书写入格式**

```bash
python miclaw_pipeline.py prepare-export pipeline_results.json \
  --s1-output s1_data.json --s2-output s2_data.json
```

## 输出

- `pipeline_results.json`：完整分析结果
- `s1_data.json`：Sheet1（任务执行场景），可直接写入飞书
- `s2_data.json`：Sheet2（用户评价），可直接写入飞书
