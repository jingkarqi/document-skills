# 代码库清理指南

**目的**：为另一个分析者提供精简的对比分析环境

---

## 📦 保留的核心文件夹

### 必须保留（分析的核心）

```
oai/
├── skills/                    # ⭐ ChatGPT 的 skills 定义（264K）
│   ├── spreadsheets/         # 电子表格 skill + API 文档
│   ├── docs/                 # DOCX 处理 skill
│   └── pdfs/                 # PDF 处理 skill
│
├── share/                     # ⭐ 共享工具和辅助脚本（102K）
│   └── slides/               # PPTX 渲染工具
│
├── cc-skills/                 # ⭐⭐⭐ Claude 的 skills 实现（2.7M）
│   ├── docx/                 # DOCX skill 完整实现
│   ├── pdf/                  # PDF skill 完整实现
│   ├── pptx/                 # PPTX skill 完整实现
│   └── xlsx/                 # XLSX skill 完整实现
│
├── CLAUDE.md                  # ⭐ ChatGPT 环境说明文档
└── report.md                  # ⭐⭐ 已完成的对比分析报告
```

**总大小**：约 3.1MB（非常精简）

---

## 🗑️ 可以安全删除的文件夹

### 运行环境配置（与 skills 分析无关）

```bash
# 删除这些目录
rm -rf .chromium/      # Chromium 浏览器配置
rm -rf .pki/           # SSL 证书数据库
rm -rf .config/        # 系统配置
rm -rf .ipython/       # IPython 配置
rm -rf .claude/        # Claude Code 临时配置
```

### 环境配置文件（可删除）

```bash
# 删除这些文件
rm -f .bash_logout
rm -f .bashrc
rm -f .profile
rm -f .wgetrc
rm -f .nssdbp
rm -f redirect.html    # 无关的 HTML 文件
```

---

## ✅ 清理后的最终结构

```
oai/                              # 项目根目录
│
├── skills/                       # ChatGPT skills（OpenAI）
│   ├── spreadsheets/
│   │   ├── skill.md             # 使用指南
│   │   ├── spreadsheet.md       # API 简介
│   │   ├── artifact_tool_spreadsheets_api.md  # 完整 API（1186 行）
│   │   ├── artifact_tool_spreadsheet_formulas.md  # 520 个公式
│   │   └── examples/            # 26 个示例代码
│   ├── docs/
│   │   ├── skill.md
│   │   └── render_docx.py       # DOCX 渲染工具
│   └── pdfs/
│       └── skill.md
│
├── share/                        # 共享工具
│   └── slides/
│       ├── render_slides.py     # PPTX 渲染工具
│       ├── create_montage.py    # 图像蒙太奇
│       └── pptxgenjs_helpers/   # JavaScript 辅助库
│
├── cc-skills/                    # Claude skills（Anthropic）
│   ├── docx/
│   │   ├── SKILL.md             # 主要使用指南
│   │   ├── ooxml.md             # OOXML 技术文档
│   │   ├── docx-js.md           # docx-js 库文档
│   │   ├── scripts/             # Document Library（Python）
│   │   └── ooxml/               # OOXML 操作工具
│   │       ├── scripts/         # unpack/pack/validate
│   │       └── schemas/         # XSD 模式文件
│   │
│   ├── pdf/
│   │   ├── SKILL.md
│   │   ├── forms.md             # PDF 表单处理
│   │   ├── reference.md         # 详细参考
│   │   └── scripts/             # 表单处理工具
│   │
│   ├── pptx/
│   │   ├── SKILL.md
│   │   ├── html2pptx.md         # HTML→PPTX 文档
│   │   ├── ooxml.md             # OOXML 技术文档
│   │   ├── scripts/             # 模板管理工具链
│   │   │   ├── html2pptx.js    # HTML 转换器
│   │   │   ├── rearrange.py    # 幻灯片重排
│   │   │   ├── inventory.py    # 文本清单提取
│   │   │   ├── replace.py      # 批量替换
│   │   │   └── thumbnail.py    # 缩略图生成
│   │   └── ooxml/              # OOXML 工具和模式
│   │
│   └── xlsx/
│       ├── SKILL.md
│       └── recalc.py           # LibreOffice 公式计算
│
├── CLAUDE.md                    # ChatGPT 环境文档
└── report.md                    # 技术对比分析报告
```

---

## 🎯 清理命令（一键执行）

在 `oai/` 目录下执行：

```bash
# 删除运行环境配置
rm -rf .chromium .pki .config .ipython .claude

# 删除环境配置文件
rm -f .bash_logout .bashrc .profile .wgetrc .nssdbp redirect.html

# 验证清理结果
du -sh */ CLAUDE.md report.md
```

**预期输出**：
```
102K    share/
264K    skills/
2.7M    cc-skills/
7.5K    CLAUDE.md
12K     report.md
```

**总计**：约 3.1MB（已清理 99% 的无关内容）

---

## 📋 分析者需要关注的核心文件

### 起点文档（必读）

1. **`report.md`**（本次分析报告）
   - 执行摘要
   - 核心架构对比
   - 关键技术差异
   - 实践启示

2. **`CLAUDE.md`**（ChatGPT 环境说明）
   - artifact_tool 介绍
   - 渲染管道说明
   - 质量标准

### ChatGPT Skills（oai）

**重点文件**：
- `skills/spreadsheets/artifact_tool_spreadsheets_api.md`（1186 行 API 文档）
- `skills/spreadsheets/artifact_tool_spreadsheet_formulas.md`（520 个公式）
- `skills/spreadsheets/skill.md`（使用指南）
- `skills/docs/render_docx.py`（渲染实现）
- `share/slides/render_slides.py`（PPTX 渲染）

**核心概念**：
- artifact_tool 专有库
- C* Proto 格式
- Granola CLI 渲染
- 多模态验证

### Claude Skills（cc-skills）

**重点文件**：
- `cc-skills/docx/SKILL.md`（DOCX 主指南 + Redlining Workflow）
- `cc-skills/docx/ooxml.md`（Document Library API）
- `cc-skills/pptx/SKILL.md`（html2pptx 方法）
- `cc-skills/pptx/html2pptx.md`（HTML 转换详细文档）
- `cc-skills/xlsx/SKILL.md`（公式计算 + recalc.py）
- `cc-skills/pdf/SKILL.md`（PDF 表单处理）

**核心概念**：
- Document Library（自研 OOXML 操作库）
- html2pptx（HTML → PPTX）
- Redlining Workflow（分批审阅）
- 模板工具链（rearrange/inventory/replace）

---

## 🔍 对比分析的关键维度

### 1. 架构对比
- ChatGPT：专有工具 vs Claude：开源组合
- 文件位置：`report.md` 第一、二部分

### 2. 实现方法
- XLSX：内置公式引擎 vs LibreOffice 调用
- DOCX：直接 XML vs Document Library
- PPTX：未公开 vs html2pptx
- PDF：基本一致，Claude 多表单处理

### 3. 工作流程
- ChatGPT：创建→计算→渲染→AI 验证→导出
- Claude：创建→保存→外部计算→脚本验证→迭代

### 4. 质量保证
- ChatGPT：AI 多模态视觉检查（自动）
- Claude：脚本验证 + 人工检查（半自动）

---

## 💡 分析建议

### 深入分析路径

**如果关注技术实现**：
1. 对比 `skills/spreadsheets/artifact_tool_spreadsheets_api.md` vs `cc-skills/xlsx/recalc.py`
2. 对比 `skills/docs/render_docx.py` vs `cc-skills/docx/scripts/document.py`
3. 研究 `cc-skills/pptx/scripts/html2pptx.js`（Claude 独创）

**如果关注工作流程**：
1. 阅读 `cc-skills/docx/SKILL.md` 的 Redlining Workflow
2. 对比 `skills/spreadsheets/skill.md` 的质量标准
3. 研究 `cc-skills/pptx/scripts/` 的模板管理工具链

**如果关注创新点**：
1. ChatGPT：C* Proto 格式 + Granola 渲染
2. Claude：html2pptx + Document Library + Redlining Workflow

---

## 📦 可选：创建分发包

```bash
# 创建压缩包（方便分享）
cd /path/to/oai
tar -czf skills-comparison.tar.gz \
    skills/ \
    share/ \
    cc-skills/ \
    CLAUDE.md \
    report.md

# 查看压缩包大小
ls -lh skills-comparison.tar.gz
# 预期：约 1-2MB
```

---

## ✅ 总结

**保留**：
- ✅ `skills/`（264K）- ChatGPT 的 skills
- ✅ `share/`（102K）- 渲染工具
- ✅ `cc-skills/`（2.7M）- Claude 的 skills
- ✅ `CLAUDE.md` + `report.md`（文档）

**删除**：
- ❌ `.chromium/`, `.pki/`, `.config/`, `.ipython/`, `.claude/`（环境配置）
- ❌ `.bash*`, `.profile`, `.wgetrc`, `.nssdbp`（配置文件）
- ❌ `redirect.html`（无关文件）

**结果**：从未知大小缩减到 **3.1MB**，保留 100% 分析所需内容。
