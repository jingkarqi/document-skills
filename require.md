# oai-skills dependencies

## CLI / system tools
- LibreOffice `soffice`
- Poppler `pdftoppm`
- Granola CLI (`granola-cli`)

## Python packages (required)
- `python-docx`
- `pdf2image`
- `reportlab`
- `pdfplumber`
- `artifact_tool`
- `openpyxl`
- `pandas`

## Python packages (optional)
- `pypdf`
- `PyMuPDF`

## Internal / proprietary SDKs
- `oaiproto.coworker.*`

## Purpose summary
- `soffice`: convert DOCX to PDF for rendering and inspection.
- `pdftoppm`: convert PDFs to PNGs for visual QA.
- `granola-cli`: render/serialize spreadsheet artifacts via the Granola engine.
- `python-docx`: create and edit DOCX files.
- `pdf2image`: rasterize PDFs to images in the DOCX rendering script.
- `reportlab`: generate PDFs programmatically.
- `pdfplumber`: read/extract PDF content when needed.
- `artifact_tool`: create/edit/recalculate/render spreadsheets.
- `openpyxl`: spreadsheet editing alternative with formatting support.
- `pandas`: spreadsheet data manipulation alternative.
- `pypdf`: optional PDF manipulation utilities.
- `PyMuPDF`: optional PDF manipulation and rendering utilities.
- `oaiproto.coworker.*`: internal proto types used by spreadsheet examples.