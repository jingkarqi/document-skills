# 仓库指南

本仓库收集了 ChatGPT 和 Claude Code 两种不同的智能体运行时所使用的文档处理“技能”（包括 Markdown 指南以及小型的 Python/JS 实用工具）。

## 项目结构与模块组织

- `oai-skills/` — 主要的技能文档和辅助工具：
  - `oai-skills/docs/` (DOCX) 包含 `render_docx.py`
  - `oai-skills/pdfs/` (PDF) 指南
  - `oai-skills/spreadsheets/` (XLSX) 指南及 `oai-skills/spreadsheets/examples/`
- `share/` — 跨技能共享的可复用工具：
  - `share/slides/` PPTX 渲染/检查工具及 `pptxgenjs_helpers/`
- `cc-skills/` — Claude Code 技能的参考快照（请将其视为 vendor 风格/第三方依赖；除非为了同步/更新，否则请避免编辑）。
- 顶层文档：`CLAUDE.md`（架构/质量规则），`report.md`（分析笔记）。

## 构建、测试与开发命令

这里没有单一的“构建”步骤；请直接使用 Python 3 运行脚本。

- 渲染 PPTX → PNG：`python share/slides/render_slides.py deck.pptx --output_dir ./out`
- 幻灯片溢出检查（必须在文件夹内运行）：`cd share/slides && python slides_test.py ../../deck.pptx`
- 渲染 DOCX → PNG：`python oai-skills/docs/render_docx.py doc.docx --output_dir ./out`
- 电子表格示例（需要专有的 `artifact_tool` 运行时）：`python oai-skills/spreadsheets/examples/create_basic_spreadsheet.py`

大多数渲染流程都要求 `PATH` 环境中包含 LibreOffice (`soffice`) 和 Poppler/PDF 工具。

## 代码风格与命名约定

- Python：4 空格缩进，文件/函数使用 `snake_case`（蛇形命名法），涉及共享逻辑时使用类型提示 (type hints)，CLI 通过 `argparse` 实现并包含 `if __name__ == "__main__"` 入口点。
- Linting/格式化：未提交全仓库统一的配置；部分文件使用了 `ruff` 指令——请保持修改与周边代码一致。
- Markdown：保持指南聚焦于任务；包含可运行的命令片段和具体路径。
- JS (`share/slides/pptxgenjs_helpers/`)：遵循现有模块风格；未经讨论请勿引入新的构建工具。

## 测试指南

测试是轻量级的，且主要为手动测试：

- `python cc-skills/pdf/scripts/check_bounding_boxes_test.py`
- `cd share/slides && python slides_test.py path/to/deck.pptx`

对于新的可复用逻辑，请使用 `unittest` 添加 `*_test.py`。
