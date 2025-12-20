# oai-skills dependencies

## CLI / system tools

- LibreOffice `soffice` (public)
- Poppler `pdftoppm` (public)
- Granola CLI (`granola-cli`) (internal/unknown)

## Python packages (required)

- `python-docx` (public)
- `pdf2image` (public)
- `reportlab` (public)
- `pdfplumber` (public)
- `artifact_tool` (internal/unknown)
- `openpyxl` (public)
- `pandas` (public)

## Python packages (optional)

- `pypdf` (public)
- `PyMuPDF` (public)

## Internal / proprietary SDKs

- `oaiproto.coworker.*` (internal/unknown)

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
