# Document Skills Comparative Research

中文版本：[README_zh.md](README_zh.md)

**Status:** Internal retrospective draft. Public release requires license/IP review.

This repository documents and compares document-processing “skills” used by two AI systems (OpenAI ChatGPT agent workflows and Anthropic Claude Code workflows). It aggregates reference materials, scripts, and an analysis report to support research into document automation pipelines, toolchain design, and quality-assurance approaches.

## Purpose

- Preserve a snapshot of real-world document-processing workflows
- Compare closed, integrated toolchains vs. open, composable toolchains
- Provide a research-oriented map of techniques for DOCX/PDF/PPTX/XLSX handling
- Serve as a base for internal retrospectives and future toolchain design

## Scope

Included:

- Skill guides and supporting scripts for document formats (DOCX, PDF, PPTX, XLSX)
- Examples and references used for analysis
- A comparative technical report (see `report.md`)

Not included:

- A runnable, end-to-end “productized” system
- A full public release checklist (see **License & IP Notice** below)

## Repository Layout

- `report.md` — Comparative technical analysis (ChatGPT vs Claude document skills)
- `require.md` / `require.zh.md` — Dependency lists and purpose summaries
- `ChatGPT-document-skills/` — ChatGPT agent-facing skill notes and examples
- `claude-code-document-skills/` — Claude Code skill guides, OOXML references, and tooling
- `share/` — Shared helper scripts (notably slide tooling)
- `OpenAI_private_Resources/` — **Private/internal artifacts (requires review)**

## Key Findings (Summary)

- ChatGPT relies on a proprietary, integrated artifact pipeline with internal rendering and formula engines.
- Claude’s workflows are built on open tools and explicit validation steps, emphasizing transparency and auditability.
- The two systems represent a trade-off between automation speed (closed) and extensibility/inspectability (open).
- PPTX workflows in Claude introduce a novel HTML-to-PPTX path and a reusable template toolchain.

For full details, read `report.md`.

## How to Use (Placeholder)

> TODO: Add operational guidance and runnable examples.
> This section will eventually cover environment setup, validation loops, and end-to-end flows.

## Dependencies (Snapshot)

Details see [require.md](require.md)

**If you plan to open source this repository, you must first review and remove or replace restricted materials.**

Suggested public-release checklist:

1. Remove or redact `OpenAI_private_Resources/`.
2. Review `claude-code-document-skills/**/LICENSE.txt` and exclude any non-redistributable content.
3. Replace proprietary references with public equivalents or documentation-only stubs.
4. Add a clear open-source license at the repository root once cleared.

## Research Value

This repository provides a rare, side-by-side look at two distinct document-automation philosophies, with concrete scripts and workflows that can inform:

- Document QA pipelines
- Spreadsheet formula calculation strategies
- OOXML-based editing systems
- HTML-to-PPTX conversion methods

## Contributing

Not yet defined. Internal retrospective only.

---

If you need a sanitized, public-ready version of this repo, start with the checklist above and update the **How to Use** section once tooling and licensing are clarified.
