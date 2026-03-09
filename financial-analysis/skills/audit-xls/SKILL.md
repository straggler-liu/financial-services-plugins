---
name: audit-xls
description: Audit a spreadsheet for formula accuracy, errors, and common mistakes. Scopes to a single sheet, the whole workbook, or a selected range. Use when checking for broken formulas, hardcodes, inconsistent formula patterns, off-by-one ranges, circular refs, or unit mismatches. Triggers on "audit this sheet", "check my formulas", "find formula errors", "QA this spreadsheet", "sanity check this", "look for mistakes".
---

# Audit Spreadsheet

Audit formulas and data for accuracy and mistakes.

## Step 1: Determine scope

If a scope is given, use it. Otherwise ask the user which they want:

- **sheet** — the current active sheet only
- **model** — the whole workbook (all sheets)
- **selection** — only the currently selected range

## Step 2: Run the audit

Check for:

- **Formula errors**: `#REF!`, `#VALUE!`, `#N/A`, `#DIV/0!`, `#NAME?`
- **Hardcoded values inside formulas** (e.g. `=A1*1.05` — the `1.05` should be a cell reference)
- **Inconsistent formulas** across a row or column (a formula that breaks the pattern of its neighbors)
- **Off-by-one range errors** (SUM/AVERAGE that misses the first or last row)
- **Cells that look like formulas but are hardcoded** (value pasted over a formula)
- **Circular references** (intentional or not)
- **Broken cross-sheet links** (references to cells that moved or were deleted)
- **Unit / scale mismatches** (thousands mixed with millions, % stored as whole numbers)

If scope is **model**, also load the `check-model` skill for the full model-integrity checks (balance sheet ties, cash flow reconciliation, logic checks).

## Step 3: Report

Output a table of findings:

| Sheet | Cell | Severity | Issue | Suggested Fix |
|---|---|---|---|---|

Severity: **Critical** (wrong output) / **Warning** (risky) / **Info** (style)

Don't change anything without asking — report first, fix on request.
