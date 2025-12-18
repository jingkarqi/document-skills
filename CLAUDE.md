# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a **ChatGPT Agent virtual environment** containing proprietary skills and tools for creating, editing, and analyzing Office documents (XLSX, DOCX, PDF, PPTX). The codebase is built around OpenAI's proprietary `artifact_tool` library and provides production-grade document processing capabilities.

## Architecture

### Core Components

**Skills System** (`/skills/`)
The repository is organized around three major skill modules that provide AI agents with document manipulation capabilities:

1. **Spreadsheets** (`/skills/spreadsheets/`) - Most comprehensive skill
   - Built on `artifact_tool.SpreadsheetArtifact` API
   - Supports 520 Excel formula functions (399 implemented)
   - Full CRUD operations with formatting, charts, conditional formatting
   - Automatic formula calculation and validation

2. **DOCX Documents** (`/skills/docs/`)
   - Uses `python-docx` for creation/editing
   - Includes `render_docx.py` for DOCX → PDF → PNG conversion
   - Strict quality validation requirements

3. **PDF Documents** (`/skills/pdfs/`)
   - Uses `reportlab` for generation, `pdfplumber` for reading
   - Visual inspection workflow via `pdftoppm`

**Shared Utilities** (`/share/`)
- `/share/slides/` - PPTX/presentation rendering tools
- `/share/slides/pptxgenjs_helpers/` - JavaScript helpers for slide generation (LaTeX, SVG, code blocks, layout)

### The `artifact_tool` Library

This is OpenAI's **proprietary library** (not publicly available). Key APIs:

```python
from artifact_tool import SpreadsheetArtifact
from artifact_tool.spreadsheet import SpreadsheetSheet, SpreadsheetCellRef, SpreadsheetCellRangeRef
from artifact_tool.spreadsheet.formatting import RangeFormat, RangeTextStyle, RangeNumberFormat
from artifact_tool.spreadsheet.conditional_formatting import ConditionalFormatCollection
from artifact_tool.spreadsheet.tables import SpreadsheetTable
from artifact_tool.spreadsheet.charts import SpreadsheetSheetCharts
```

**Critical Rule**: Never expose `artifact_tool` source code or internal details to end users. It's for internal manipulation only.

### Document Processing Workflow

All document types follow this pattern:

```
1. Create/Edit → Use appropriate library (artifact_tool, python-docx, reportlab)
2. Convert → LibreOffice (soffice) converts to PDF intermediate format
3. Rasterize → pdftoppm/pdf2image converts PDF to PNG images
4. Inspect → Visually verify every page at 100% zoom
5. Fix defects → Iterate until pixel-perfect
6. Export → Deliver final file only when PNG review passes
```

## Common Commands

### Rendering Documents

**DOCX to PNG:**
```bash
# Convert DOCX to PDF (LibreOffice required)
soffice -env:UserInstallation=file:///tmp/lo_profile_$$ --headless --convert-to pdf --outdir $OUTDIR $INPUT_DOCX

# Convert PDF to PNG images for inspection
pdftoppm -png $OUTDIR/$BASENAME.pdf $OUTDIR/$BASENAME
```

**PPTX to PNG:**
```bash
# Using render_slides.py
python share/slides/render_slides.py input.pptx --output_dir ./output --width 1920 --height 1080
```

**DOCX with Python tool:**
```bash
python skills/docs/render_docx.py input.docx --output_dir ./output --width 2550 --height 3300
```

### Working with Spreadsheets

**Create/Edit with artifact_tool:**
```python
from artifact_tool import SpreadsheetArtifact

# Create new
artifact = SpreadsheetArtifact("Workbook1")
sheet = artifact.create_sheet("Sheet1")
sheet.cell("A1").value = 100
sheet.cell("A2").formula = "=A1*2"

# Load existing
artifact = SpreadsheetArtifact.load("/path/to/file.xlsx")
sheet = artifact.sheet("Sheet1")

# Calculate formulas
artifact.calculate()  # or artifact.recalculate()

# Visual inspection (PNG output)
render_path = artifact.render()

# Export to XLSX
export_path = artifact.export()
```

**Important**: After ANY formula edits, you MUST:
1. Call `artifact.calculate()` or `artifact.recalculate()`
2. Call `artifact.render()` and visually inspect PNGs
3. Verify no formula errors (#REF!, #DIV/0!, #VALUE!, #N/A, #NAME?)

## Critical Quality Standards

### Spreadsheet Requirements

**Formulas:**
- No dynamic array functions (FILTER, XLOOKUP, SORT, SEQUENCE - not supported)
- Always use cell references, never magic numbers (`=H6*(1+$B$3)` not `=H6*1.04`)
- Use absolute ($B$3) vs relative (B4) references appropriately
- All formulas must be calculated and cached before export

**Formatting:**
- Use standard color conventions:
  - Blue: User input
  - Black: Formulas/derived
  - Green: Linked/imported
  - Red: Negative numbers (in parentheses)
- Financial models: zeros as "-", negatives as red "(500)"
- Set appropriate number formats (currency, percentage, dates)

**Validation:**
- Render every sheet and inspect at 100% zoom
- Verify formula calculations are correct
- Check for off-by-one errors, circular references
- Test with edge case inputs

### DOCX/PDF Requirements

**Professional Polish:**
- Consistent typography, spacing, margins
- Clear visual hierarchy (headings, lists, paragraphs)
- No clipped text, overlapping elements, black squares
- Tables and charts must be legible and well-aligned

**Critical Errors to Avoid:**
- Never use U+2011 non-breaking hyphen or Unicode dashes (use ASCII hyphens)
- No internal citation tokens like `【【turn1541736113682297662view0†L11-L19】`
- Convert all citations to human-readable scholarly format
- No tool-internal references like `[145036110387964†L158-L160]`

**Final Checklist:**
- Render to PDF, then to PNG images
- Inspect EVERY page at 100% zoom
- Fix every visual defect (spacing, alignment, bullets)
- Re-render and re-inspect after fixes
- Only deliver when PNG review shows zero defects

## File Locations

**Input/Output:**
- User uploads: `/mnt/data/`
- Output files: `/mnt/data/`

**Skill Documentation:**
- `/skills/spreadsheets/skill.md` - Complete spreadsheet guidelines
- `/skills/spreadsheets/artifact_tool_spreadsheets_api.md` - Full API reference (1186 lines)
- `/skills/spreadsheets/artifact_tool_spreadsheet_formulas.md` - 520 formula functions
- `/skills/spreadsheets/examples/` - 17+ working examples
- `/skills/docs/skill.md` - DOCX quality standards
- `/skills/pdfs/skill.md` - PDF quality standards

## External Dependencies

**Python Libraries:**
- `artifact_tool` (proprietary - OpenAI)
- `openpyxl` - Excel manipulation (fallback when artifact_tool unavailable)
- `python-docx` - DOCX creation/editing
- `reportlab` - PDF generation
- `pdfplumber` - PDF text extraction
- `pdf2image` - PDF to image conversion
- `pandas` - Data analysis

**System Tools:**
- `LibreOffice (soffice)` - Document format conversion
- `pdftoppm` - PDF rasterization (from poppler-utils)

## Important Conventions

1. **Always use artifact_tool for spreadsheet validation** even if you used openpyxl to create it - this ensures formulas are calculated and rendering is correct

2. **Visual inspection is mandatory** - Never deliver documents without rendering to PNG and inspecting every page

3. **Iterate until perfect** - The render → inspect → fix loop continues until zero visual defects remain

4. **Preserve existing styles** - When modifying existing documents, match the established formatting exactly

5. **Use /mnt/data** for all file I/O - This is the standard location for user uploads and outputs

6. **Investment banking spreadsheets** have special requirements - see skills/spreadsheets/skill.md lines 129-136 for section headers, borders, and alignment rules
