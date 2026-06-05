# nekou-skills

[English](#english) | [中文](#中文)

---

## English

A collection of **Claude Code Skills** — structured workflow guides that teach AI assistants how to use specific MCP (Model Context Protocol) tool sets.

This repository currently includes database, API, and presentation-generation skills.

### Skills

| Skill | Description |
|-------|-------------|
| [kingbase-skill](./kingbase-skill/) | Connect to and manage KingBase databases (PostgreSQL-compatible) via `kb_*` MCP tools. Supports querying, schema exploration, DML/DDL with two-phase confirmation, and performance analysis. |
| [swagger-api-skill](./swagger-api-skill/) | Explore and call REST APIs from Swagger/OpenAPI specs via `swagger_*` MCP tools. Supports spec loading, endpoint discovery, schema inspection, and API calls with preview/confirm. |
| [image2ppt](./image2ppt/) | Convert images, screenshots, diagrams, flowcharts, architecture charts, and infographics into PowerPoint/PPTX decks. Supports fast static-image slides and editable reconstructions using `python-pptx` plus the bundled scaffold script. |

### Skill Structure

```
<skill-name>/
├── SKILL.md              # Main skill definition (YAML frontmatter + workflow guide)
├── references/           # Detailed reference docs
├── assets/               # Supporting assets
└── scripts/              # Helper scripts
```

### Installation

Copy the skill directory into `~/.claude/skills/`:

```bash
cp -r kingbase-skill ~/.claude/skills/
cp -r swagger-api-skill ~/.claude/skills/
cp -r image2ppt ~/.claude/skills/
```

After installation, Claude Code will automatically detect and use the skills when relevant MCP tools are available.

### Image2PPT Quick Usage

Use `image2ppt` when you want Claude Code to turn a source image into a PowerPoint deck:

- **Static PPT**: place one or more images into slides with preserved aspect ratio.
- **Editable PPT**: recreate diagrams and infographics with PowerPoint shapes, text boxes, arrows, connectors, and layout primitives.

The skill includes a reusable scaffold at [`image2ppt/scripts/image2ppt.py`](./image2ppt/scripts/image2ppt.py). For editable reconstruction, the assistant should copy the scaffold into the working directory and adapt `build_deck()` for the current image rather than starting from scratch.

Typical workflow:

```bash
cp ~/.claude/skills/image2ppt/scripts/image2ppt.py ./image2ppt.py
python3 image2ppt.py --output result-editable.pptx
```

Required Python package: `python-pptx`. `Pillow` is useful when image dimensions, crops, or color sampling are needed.

### Creating New Skills

1. Create a directory under the skill name
2. Add a `SKILL.md` with YAML frontmatter (`name` and `description` fields)
3. Include workflow decision trees and tool usage patterns in the body
4. Place detailed references in `references/`

### Related Projects

- [kingbase-mcp-server](https://github.com/NekoTarou/kingbase-mcp-server) — KingBase MCP Server
- [MCP Protocol](https://modelcontextprotocol.io/) — Model Context Protocol

### License

MIT

---

## 中文

一组 **Claude Code 技能** —— 结构化的工作流指南，指导 AI 助手如何使用特定的 MCP（模型上下文协议）工具集。

当前仓库包含数据库、API 和演示文稿生成相关技能。

### 技能列表

| 技能 | 说明 |
|------|------|
| [kingbase-skill](./kingbase-skill/) | 通过 `kb_*` MCP 工具连接和管理 KingBase 数据库（兼容 PostgreSQL）。支持查询、Schema 探索、两阶段确认的 DML/DDL 操作以及性能分析。 |
| [swagger-api-skill](./swagger-api-skill/) | 通过 `swagger_*` MCP 工具探索和调用 Swagger/OpenAPI 规范中的 REST API。支持规范加载、端点发现、Schema 查看和预览/确认模式的 API 调用。 |
| [image2ppt](./image2ppt/) | 将图片、截图、流程图、架构图、图表和信息图转换为 PowerPoint/PPTX。支持快速生成静态图片页，也支持基于 `python-pptx` 和内置脚手架脚本重建可编辑 PPT。 |

### 技能目录结构

```
<skill-name>/
├── SKILL.md              # 技能主文件（YAML 头信息 + 工作流指南）
├── references/           # 详细参考文档
├── assets/               # 辅助资源
└── scripts/              # 辅助脚本
```

### 安装方式

将技能目录复制到 `~/.claude/skills/`：

```bash
cp -r kingbase-skill ~/.claude/skills/
cp -r swagger-api-skill ~/.claude/skills/
cp -r image2ppt ~/.claude/skills/
```

安装后，当相关 MCP 工具可用时，Claude Code 会自动检测并使用这些技能。

### Image2PPT 快速使用

当你希望 Claude Code 将源图片转换为 PowerPoint 时，可以使用 `image2ppt`：

- **静态 PPT**：将一张或多张图片按比例放入幻灯片。
- **可编辑 PPT**：用 PowerPoint 形状、文本框、箭头、连接线和布局元素重建流程图、架构图或信息图。

该技能内置可复用脚手架 [`image2ppt/scripts/image2ppt.py`](./image2ppt/scripts/image2ppt.py)。做可编辑重建时，助手应将脚手架复制到当前工作目录，并针对当前图片改写 `build_deck()`，而不是从空脚本开始。

典型流程：

```bash
cp ~/.claude/skills/image2ppt/scripts/image2ppt.py ./image2ppt.py
python3 image2ppt.py --output result-editable.pptx
```

必需 Python 包：`python-pptx`。如果需要读取图片尺寸、裁剪或取色，通常还会用到 `Pillow`。

### 创建新技能

1. 以技能名称创建目录
2. 添加包含 YAML 头信息（`name` 和 `description` 字段）的 `SKILL.md`
3. 在正文中编写工作流决策树和工具使用模式
4. 将详细参考资料放入 `references/`

### 相关项目

- [kingbase-mcp-server](https://github.com/NekoTarou/kingbase-mcp-server) — KingBase MCP 服务器
- [MCP Protocol](https://modelcontextprotocol.io/) — 模型上下文协议

### 许可证

MIT
