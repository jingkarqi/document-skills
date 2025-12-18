# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在处理本仓库代码时提供指导。

## 概览

这是一个 **ChatGPT Agent 虚拟环境**，包含用于创建、编辑和分析 Office 文档（XLSX, DOCX, PDF, PPTX）的专有技能和工具。该代码库围绕 OpenAI 的专有 `artifact_tool` 库构建，提供生产级的文档处理能力。

## 架构

### 核心组件

**技能系统** (`/oai-skills/`)
本仓库围绕三个主要技能模块组织，为 AI 智能体提供文档操作能力：

1.  **电子表格 (Spreadsheets)** (`/oai-skills/spreadsheets/`) - 最全面的技能

    - 基于 `artifact_tool.SpreadsheetArtifact` API 构建
    - 支持 520 个 Excel 公式函数（已实现 399 个）
    - 完整的增删改查 (CRUD) 操作，包括格式化、图表、条件格式
    - 自动公式计算和验证

2.  **DOCX 文档** (`/oai-skills/docs/`)

    - 使用 `python-docx` 进行创建/编辑
    - 包含 `render_docx.py` 用于 DOCX → PDF → PNG 的转换
    - 严格的质量验证要求

3.  **PDF 文档** (`/oai-skills/pdfs/`)
    - 使用 `reportlab` 生成，使用 `pdfplumber` 读取
    - 通过 `pdftoppm` 进行视觉检查工作流

**共享工具** (`/share/`)

- `/share/slides/` - PPTX/演示文稿渲染工具
- `/share/slides/pptxgenjs_helpers/` - 用于幻灯片生成的 JavaScript 辅助工具（LaTeX, SVG, 代码块, 布局）

### `artifact_tool` 库

这是 OpenAI 的 **专有库**（不公开发布）。关键 API：

```python
from artifact_tool import SpreadsheetArtifact
from artifact_tool.spreadsheet import SpreadsheetSheet, SpreadsheetCellRef, SpreadsheetCellRangeRef
from artifact_tool.spreadsheet.formatting import RangeFormat, RangeTextStyle, RangeNumberFormat
from artifact_tool.spreadsheet.conditional_formatting import ConditionalFormatCollection
from artifact_tool.spreadsheet.tables import SpreadsheetTable
from artifact_tool.spreadsheet.charts import SpreadsheetSheetCharts
```

### 文档处理工作流

所有文档类型均遵循此模式：

```
1. 创建/编辑 → 使用适当的库 (artifact_tool, python-docx, reportlab)
2. 转换 → LibreOffice (soffice) 将文档转换为 PDF 中间格式
3. 栅格化 → pdftoppm/pdf2image 将 PDF 转换为 PNG 图像
4. 检查 → 在 100% 缩放比例下视觉验证每一页
5. 修复缺陷 → 迭代直到像素级完美
6. 导出 → 仅在 PNG 审查通过后交付最终文件
```

## 常用命令

### 渲染文档

**DOCX 转 PNG：**

```bash
# 将 DOCX 转换为 PDF (需要 LibreOffice)
soffice -env:UserInstallation=file:///tmp/lo_profile_$$ --headless --convert-to pdf --outdir $OUTDIR $INPUT_DOCX

# 将 PDF 转换为 PNG 图像以便检查
pdftoppm -png $OUTDIR/$BASENAME.pdf $OUTDIR/$BASENAME
```

**PPTX 转 PNG：**

```bash
# 使用 render_slides.py
python share/slides/render_slides.py input.pptx --output_dir ./output --width 1920 --height 1080
```

**使用 Python 工具处理 DOCX：**

```bash
python skills/docs/render_docx.py input.docx --output_dir ./output --width 2550 --height 3300
```

### 处理电子表格

**使用 artifact_tool 创建/编辑：**

```python
from artifact_tool import SpreadsheetArtifact

# 新建
artifact = SpreadsheetArtifact("Workbook1")
sheet = artifact.create_sheet("Sheet1")
sheet.cell("A1").value = 100
sheet.cell("A2").formula = "=A1*2"

# 加载现有文件
artifact = SpreadsheetArtifact.load("/path/to/file.xlsx")
sheet = artifact.sheet("Sheet1")

# 计算公式
artifact.calculate()  # 或 artifact.recalculate()

# 视觉检查 (输出 PNG)
render_path = artifact.render()

# 导出为 XLSX
export_path = artifact.export()
```

**重要**：在进行**任何**公式编辑后，你**必须**：

1. 调用 `artifact.calculate()` 或 `artifact.recalculate()`
2. 调用 `artifact.render()` 并视觉检查 PNG
3. 验证没有公式错误 (#REF!, #DIV/0!, #VALUE!, #N/A, #NAME?)

## 关键质量标准

### 电子表格要求

**公式：**

- 无动态数组函数（不支持 FILTER, XLOOKUP, SORT, SEQUENCE）
- 始终使用单元格引用，绝不使用魔法数字（应为 `=H6*(1+$B$3)` 而非 `=H6*1.04`）
- 正确使用绝对引用 ($B$3) 与相对引用 (B4)
- 所有公式必须在导出前计算并缓存

**格式化：**

- 使用标准颜色约定：
  - 蓝色：用户输入
  - 黑色：公式/推导值
  - 绿色：链接/导入值
  - 红色：负数（带括号）
- 金融模型：零显示为“-”，负数显示为红色的“(500)”
- 设置适当的数字格式（货币、百分比、日期）

**验证：**

- 渲染每个 Sheet 并在 100% 缩放比例下检查
- 验证公式计算是否正确
- 检查差一错误 (off-by-one)、循环引用
- 使用边缘情况输入进行测试

### DOCX/PDF 要求

**专业润色：**

- 一致的排版、间距、页边距
- 清晰的视觉层级（标题、列表、段落）
- 无被截断的文本、重叠元素、黑方块
- 表格和图表必须清晰易读且对齐良好

**需避免的关键错误：**

- 绝不使用 U+2011 不换行连字符或 Unicode 破折号（使用 ASCII 连字符）
- 无内部引用标记，如 `【【turn1541736113682297662view0†L11-L19】`
- 将所有引用转换为人类可读的学术格式
- 无工具内部引用，如 `[145036110387964†L158-L160]`

**最终检查清单：**

- 渲染为 PDF，然后转换为 PNG 图像
- 在 100% 缩放比例下检查**每一页**
- 修复每一个视觉缺陷（间距、对齐、项目符号）
- 修复后重新渲染并重新检查
- 仅在 PNG 审查显示零缺陷时交付

## 文件位置

**输入/输出：**

- 用户上传：`/mnt/data/`
- 输出文件：`/mnt/data/`

**技能文档：**

- `/oai-skills/spreadsheets/skill.md` - 完整的电子表格指南
- `/oai-skills/spreadsheets/artifact_tool_spreadsheets_api.md` - 完整的 API 参考（1186 行）
- `/oai-skills/spreadsheets/artifact_tool_spreadsheet_formulas.md` - 520 个公式函数
- `/oai-skills/spreadsheets/examples/` - 17+ 个可运行示例
- `/oai-skills/docs/skill.md` - DOCX 质量标准
- `/oai-skills/pdfs/skill.md` - PDF 质量标准

## 外部依赖

**Python 库：**

- `artifact_tool` (专有 - OpenAI)
- `openpyxl` - Excel 操作（当 artifact_tool 不可用时的后备方案）
- `python-docx` - DOCX 创建/编辑
- `reportlab` - PDF 生成
- `pdfplumber` - PDF 文本提取
- `pdf2image` - PDF 转图像
- `pandas` - 数据分析

**系统工具：**

- `LibreOffice (soffice)` - 文档格式转换
- `pdftoppm` - PDF 栅格化（来自 poppler-utils）

## 重要约定

1.  **始终使用 artifact_tool 进行电子表格验证**，即使你使用 openpyxl 创建了它 - 这能确保公式已被计算且渲染正确。

2.  **视觉检查是强制性的** - 绝不要在未渲染为 PNG 并检查每一页的情况下交付文档。

3.  **迭代直到完美** - “渲染 → 检查 → 修复”的循环应持续进行，直到没有视觉缺陷为止。

4.  **保留现有样式** - 修改现有文档时，必须精确匹配既定的格式。

5.  **使用 /mnt/data** 进行所有文件 I/O - 这是用户上传和输出的标准位置。

6.  **投资银行电子表格**有特殊要求 - 参见 `oai-skills/spreadsheets/skill.md` 第 129-136 行关于章节标题、边框和对齐的规则。
