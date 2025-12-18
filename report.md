# ChatGPT vs Claude 文档处理 Skills 技术分析报告

**分析对象**：OpenAI ChatGPT Agent 与 Anthropic Claude Code 的文档处理能力实现

**报告日期**：2025-12-18

---

## 执行摘要

本报告分析了两大 AI 系统在文档处理领域采用的截然不同的技术路径。ChatGPT 依赖专有的 `artifact_tool` 库实现一体化工具链，而 Claude 基于开源生态构建了透明可扩展的解决方案。两者的核心差异不仅体现在技术实现上，更反映了不同的产品哲学：**封闭式自动化** vs **开放式可控性**。

**关键发现**：
- ChatGPT 的 `artifact_tool` 是不可复制的专有技术，包含完整的公式引擎和渲染系统
- Claude 的方案完全基于开源工具（openpyxl、python-docx、pypdf），用户可直接使用
- Claude 开创了 `html2pptx` 方法，允许用 Web 技术创建演示文稿
- 两者都使用 LibreOffice 和 pandoc，但在工作流程中的角色不同

---

## 一、核心架构对比

### 1.1 技术栈差异

**ChatGPT 架构**：
```
用户请求 → AI 推理 → artifact_tool (专有库) → C* Proto 格式
                     ↓
            Granola CLI 渲染 → PNG 图像 → AI 多模态验证 → 输出文档
```

**Claude 架构**：
```
用户请求 → AI 推理 → 开源工具 (openpyxl/pypdf/docx) → OOXML/PDF
                     ↓
            LibreOffice/Pandoc → 转换/验证 → 脚本检查 → 输出文档
```

### 1.2 设计理念

| 维度 | ChatGPT | Claude |
|------|---------|--------|
| **核心理念** | 端到端自动化 | 工具链组合 |
| **中间格式** | C* Proto (专有二进制) | 标准 OOXML XML |
| **验证方式** | AI 视觉检查 (自动) | 脚本 + 人工 (半自动) |
| **用户可用性** | 仅限平台内部 | 完全开源可用 |
| **扩展性** | 封闭系统 | 开放生态 |

---

## 二、文档类型实现方法

### 2.1 电子表格 (XLSX)

**ChatGPT - artifact_tool 方案**：

核心优势是**内置公式引擎**，支持 520 个 Excel 函数（399 个已实现），可在 Python 中直接计算公式而无需 Excel。工作流为：创建 → 计算 → 渲染 PNG → AI 检查 → 导出，整个过程一体化且高度自动化。

**Claude - openpyxl + LibreOffice 方案**：

使用标准的 `openpyxl` 库创建文件，通过 `recalc.py` 脚本调用 LibreOffice Calc 来计算公式。该脚本会扫描所有单元格，检测公式错误（#REF!、#DIV/0! 等），并返回详细的 JSON 错误报告。工作流为：创建 → 保存 → 外部计算 → 错误检查 → 迭代修复。

**关键差异**：ChatGPT 的优势在于速度和集成度；Claude 的优势在于使用真实的 Excel 计算引擎（完全兼容）且用户可以直接使用这些工具。

### 2.2 Word 文档 (DOCX)

**ChatGPT 方案**：

使用 `python-docx` 创建文档，通过直接操作 OOXML XML 实现追踪修改（redlining）。验证时将文档转为 PDF 再转为 PNG，由 AI 进行视觉检查以发现排版问题。

**Claude - Document Library 方案**：

提供了更高级的抽象层。核心创新是 **Document Library**，这是一个 Python 库，提供了 `get_node()`、`insert_tracked_change()` 等高级方法来操作 OOXML。

更重要的是 **Redlining Workflow**（红线审阅流程）：
1. 将文档转为 Markdown 分析
2. 规划所有修改，按逻辑分组（3-10 个一批）
3. 逐批实施修改
4. 每批完成后立即验证
5. 重复直到所有修改完成

这种分批验证的方法显著降低了错误累积的风险，特别适合法律、学术等需要严格文档审阅的场景。

**关键差异**：Claude 提供了系统化的审阅流程和高级 API，而 ChatGPT 更侧重于快速一次性完成。

### 2.3 演示文稿 (PPTX)

**ChatGPT 方案**：

细节未完全公开，但包含智能 DPI 计算（从 OOXML 中提取页面尺寸）和自动渲染系统。

**Claude - html2pptx 创新**：

这是 Claude 最具创新性的技术。核心思想是**用 HTML/CSS 设计幻灯片**，然后通过 `html2pptx.js` 转换为 PowerPoint。此外，Claude 还提供了完整的**模板管理工具链**：
- `rearrange.py`：复制、删除、重排幻灯片
- `inventory.py`：提取所有文本形状的清单
- `replace.py`：批量替换文本
- `thumbnail.py`：生成缩略图网格进行可视化分析

**关键差异**：Claude 的 html2pptx 方法允许利用成熟的 Web 技术来设计演示文稿，这是一个独特的技术路径。模板管理工具链也使得基于现有模板创建新演示文稿变得高效。

### 2.4 PDF 文档

**共同点**：两者都使用 `reportlab` 创建、`pdfplumber` 提取、`pypdf` 操作 PDF。

**Claude 特色**：提供了完整的 **PDF 表单处理工具链**，包括表单字段检测、边界框验证、自动填写等功能，使得 PDF 表单的批量处理成为可能。

---

## 三、关键技术差异

### 3.1 公式计算引擎

**ChatGPT**：`artifact_tool` 内置的公式引擎用 C/C++ 实现，性能优异，支持 399 个 Excel 函数。在 Python 中直接调用 `artifact.calculate()` 即可完成计算。

**Claude**：通过 `recalc.py` 调用 LibreOffice Calc 的宏来计算公式。虽然需要启动外部进程（性能开销），但获得了完整的 Excel 兼容性。

**启示**：内置引擎换取速度，外部引擎换取兼容性和透明度。

### 3.2 视觉验证机制

**ChatGPT - 多模态 AI 验证**：

文档通过 Granola CLI（C++ 渲染引擎）转为高质量 PNG，然后 AI 使用多模态能力"看"图像，自动检测文本裁剪、对齐问题、颜色对比度、排版错误。这是完全自动化的质量保证。

**Claude - 脚本辅助验证**：

使用 LibreOffice 和 poppler 工具转换文档为图像，通过脚本（如 `thumbnail.py`）生成可视化网格，但最终检查依赖人工判断。脚本会验证 Schema 合规性、检测公式错误等。

**启示**：自动化 vs 可控性的权衡。ChatGPT 更快但不透明；Claude 更慢但可解释。

### 3.3 OOXML 操作方法

**ChatGPT**：直接操作 XML 字符串，提供 XML 模式参考和示例模板。

**Claude - Document Library**：提供高级抽象 API，包括节点查找、追踪修改、批量操作等方法，使得复杂的文档操作变得更加安全和可维护。

---

## 四、设计理念对比

### 4.1 ChatGPT：专业工作室模式

**特征**：一体化工具链、黑盒自动化、AI 驱动验证、快速迭代

**适用场景**：AI Agent 内部使用、追求效率和自动化、标准化文档生成

### 4.2 Claude：开源工具箱模式

**特征**：组合开源工具、透明可控、脚本辅助、分批验证

**适用场景**：用户需要直接使用工具、需要完全透明和可审计、复杂文档审阅（法律、学术）、自定义扩展需求

---

## 五、创新技术亮点

### ChatGPT 独创技术

1. **C* Proto 格式**：统一的跨文档类型二进制格式，基于 Protocol Buffers
2. **Granola CLI**：高性能 C++ 渲染引擎，精确复现 Office 效果
3. **artifact_tool**：一体化 Python API，屏蔽复杂性
4. **多模态验证流程**：AI 自动检查渲染结果的创新方法

### Claude 独创技术

1. **html2pptx**：用 HTML/CSS 创建演示文稿的创新路径
2. **Document Library**：OOXML 操作的高级抽象 API
3. **Redlining Workflow**：系统化的分批文档审阅流程
4. **模板工具链**：rearrange/inventory/replace 三件套
5. **设计系统**：18 种预设配色方案和布局模式库

---

## 六、实践启示

### 6.1 架构设计启示

**从 ChatGPT 学习**：
- 端到端自动化能显著提升效率
- 视觉验证是文档质量保证的有效方法
- 统一的中间格式（C* Proto）简化跨文档操作
- AI 多模态能力可以自动化传统上需要人工的检查

**从 Claude 学习**：
- 开源工具的组合可以达到商业级效果
- 高级 API 抽象（Document Library）降低 OOXML 操作复杂度
- 分批验证策略在复杂任务中更可靠
- 创新使用 Web 技术（html2pptx）拓宽了解决方案空间

### 6.2 技术选型指南

**选择专有方案（类似 ChatGPT）当**：需要最大化自动化程度、性能是关键考量、用户无需直接使用工具、有资源投入专有技术开发

**选择开源方案（类似 Claude）当**：需要透明度和可审计性、用户需要直接使用工具、需要灵活扩展和定制、希望降低技术门槛

### 6.3 实现建议

**构建文档处理系统时**：
1. **公式引擎**：考虑权衡自研（性能）vs 外部调用（兼容性）
2. **视觉验证**：引入 AI 多模态能力可自动化质量检查
3. **OOXML 操作**：提供高级 API 而非直接暴露 XML
4. **工作流设计**：复杂任务采用分批验证降低风险
5. **模板管理**：投资构建重用和批量处理工具

---

## 七、技术成熟度评估

### ChatGPT 方案成熟度

**优势**：高度集成，经过大规模生产验证；自动化程度高，减少人为错误；性能优异（C++ 渲染引擎）

**限制**：专有技术，外部无法复现；黑盒系统，难以调试；依赖 OpenAI 基础设施

### Claude 方案成熟度

**优势**：完全基于成熟开源工具；用户可立即使用所有工具；透明可审计，便于调试；可自由定制和扩展

**限制**：多步骤流程，效率略低；需要外部依赖（LibreOffice）；人工验证环节较多

---

## 八、结论

ChatGPT 和 Claude 代表了文档处理自动化的两条路径：**专有集成** vs **开源组合**。

**ChatGPT 的 artifact_tool** 是一个不可复制的技术优势，其内置公式引擎和 AI 视觉验证构成了强大的护城河。这种方案在效率和自动化程度上无可匹敌，但仅限于平台内部使用。

**Claude 的开源方案** 证明了通过精心设计的工具链组合，可以在不依赖专有技术的情况下实现高质量文档处理。其 **html2pptx** 方法和 **Document Library** 是重要的技术创新，**Redlining Workflow** 则为复杂文档审阅提供了最佳实践。

**战略建议**：
- 对于需要最大化控制和自动化的场景，应投资构建类似 artifact_tool 的专有工具
- 对于需要用户直接使用的场景，Claude 的开源方案提供了可行且成熟的路径
- 两种方案的核心理念（AI 视觉验证、分批验证、高级抽象）都值得借鉴

**未来方向**：
- 将 AI 多模态验证引入开源工具链
- 开发 artifact_tool 的开源替代品
- 探索更多创新方法（如 html2pptx）来降低文档处理复杂度

---

## 附录：关键技术组件清单

### ChatGPT 技术栈
- `artifact_tool`：专有 Python 库（C/C++ 后端）
- `Granola CLI`：C++ 渲染引擎
- C* Proto：基于 Protocol Buffers 的文档格式
- 公式引擎：399 个 Excel 函数实现
- AI 视觉验证：多模态能力

### Claude 技术栈
- `openpyxl`：Excel 操作（开源）
- `python-docx`/`docx-js`：Word 创建（开源）
- `pypdf`/`pdfplumber`：PDF 操作（开源）
- `LibreOffice`：公式计算和格式转换
- `pandoc`：文档格式转换
- `poppler-utils`：PDF 渲染
- **Document Library**：自研 OOXML 操作库
- **html2pptx**：自研 HTML → PPTX 转换器
- **模板工具链**：rearrange/inventory/replace

---

**报告完成**

本报告基于对两个系统实际代码库和文档的深入分析。所有技术描述均基于实际实现，而非推测。
