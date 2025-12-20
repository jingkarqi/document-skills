# oai-skills 依赖清单

## CLI / 系统工具
- LibreOffice `soffice`
- Poppler `pdftoppm`
- Granola CLI（`granola-cli`）

## Python 包（必需）
- `python-docx`
- `pdf2image`
- `reportlab`
- `pdfplumber`
- `artifact_tool`
- `openpyxl`
- `pandas`

## Python 包（可选）
- `pypdf`
- `PyMuPDF`

## 内部 / 专有 SDK
- `oaiproto.coworker.*`

## 用途说明（简要）
- `soffice`：将 DOCX 转为 PDF，便于渲染与目检。
- `pdftoppm`：将 PDF 转为 PNG，用于视觉质检。
- `granola-cli`：通过 Granola 引擎渲染/序列化电子表格 artifact。
- `python-docx`：创建与编辑 DOCX。
- `pdf2image`：在 DOCX 渲染脚本中将 PDF 光栅化为图片。
- `reportlab`：程序化生成 PDF。
- `pdfplumber`：按需读取/提取 PDF 内容。
- `artifact_tool`：创建/编辑/重算/渲染电子表格。
- `openpyxl`：电子表格编辑的替代方案，支持格式。
- `pandas`：电子表格数据处理的替代方案。
- `pypdf`：可选的 PDF 处理工具。
- `PyMuPDF`：可选的 PDF 处理与渲染工具。
- `oaiproto.coworker.*`：电子表格示例里使用的内部 proto 类型。