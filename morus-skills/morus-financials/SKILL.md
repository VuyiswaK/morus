---
name: morus-financials
description: >
  Use this skill whenever a user uploads or pastes company financial data (annual report,
  10-K, earnings release, investor presentation, or raw financials) and asks for a financial
  model, FCFF analysis, three-statement model, or any structured financial output.
  Trigger on phrases like "build a model", "run the numbers", "simple FCFF", "full model",
  "financial model for X", or any time company financials are provided with the intent to
  analyse or model them. Always use this skill — even for quick or informal requests — 
  when financial data is the input and a structured Excel output is expected.
---

# Morus Financial Modelling Skill

**Morus** — Independent equity research, reasoning and a concentrated portfolio.

---

## Overview

This skill produces structured Excel financial models from company financial data.
Two output types are supported:

| Type | Template | Use when |
|---|---|---|
| **Simple FCFF** | `assets/simple-fcff-temp.xlsx` | Quick FCF-to-firm snapshot, historic + estimated |
| **Full Model** | `assets/fin-temp.xlsx` | Three-statement model (P&L, Balance Sheet, Cash Flow) + DCF + Comps |

The user will typically say "simple" or "full" — if ambiguous, default to **Simple FCFF** and offer to build the full model after.

---

## Step-by-Step Workflow

### 1. Identify inputs
- Source type: annual report / 10-K / earnings release / pasted text / prior Excel
- Most recent fiscal year end and reporting currency
- Number of historical periods available

### 2. Choose output type
- **Simple FCFF**: user says "quick", "simple", "just the FCF" — or fewer than 3 years of data
- **Full Model**: user says "full", "three-statement", "capital markets model", "comps", "DCF" — or 5+ years of data

### 3. Extract data
Pull the following from the source. If a line item is not disclosed, leave the cell blank and apply a red fill — **never estimate or fill with zero**.

**Income Statement (required for both)**
- Revenue (total)
- Cost of goods sold / Cost of sales
- Gross profit
- SG&A / Operating expenses
- R&D (if disclosed)
- D&A
- EBIT / Operating profit
- Interest expense
- Earnings before tax
- Tax
- Net income / Net earnings

**Balance Sheet (required for Full Model; Working Capital items for Simple)**
- Cash & equivalents
- Trade & other receivables
- Inventories
- Other current assets
- Total current assets
- PPE (net)
- Goodwill & intangibles
- Other non-current assets
- Total assets
- Short-term borrowings
- Accounts payable / Trade payables
- Accrued liabilities / Other current liabilities
- Total current liabilities
- Long-term debt
- Other non-current liabilities
- Total liabilities
- Shareholders' equity (total)

**Cash Flow Statement (required for Full Model)**
- Net cash from operating activities
- Capex
- Net cash from investing activities
- Dividends paid
- Share repurchases
- Net cash from financing activities
- Net change in cash

### 4. Populate the correct template

Read the relevant template from `assets/` using openpyxl (`load_workbook`).
Preserve all existing formatting, formulas, and structure — **do not reformat**.
Only write into data input cells (blue-text cells in the template convention).

**For Simple FCFF** (`assets/simple-fcff-temp.xlsx`):
- Sheet `Input Data`: populate Income Statement, Balance Sheet, Cash Flow rows for each historical year
- Sheet `Historic FCFF`: verify FCF = Net Income + D&A − ΔWorking Capital − Capex flows correctly from Input Data
- Sheet `Estimated FCFF`: populate WACC and terminal growth rate assumptions; revenue growth and FCF margin assumptions in blue cells
- Sheet `Cover Page`: write company name; embed company logo if provided (base64 `<img>`), otherwise styled company name text

**For Full Model** (`assets/fin-temp.xlsx`):
- Sheet `Input Data`: populate all three statements for historical years
- Sheet `P&L`: verify income statement drivers (margins, growth rates) are populated in assumption rows
- Sheet `Balance Sheet`: verify balance sheet drivers (receivable days, inventory days, payable days, capex %) are populated
- Sheet `Cash Flow`: flows from P&L and Balance Sheet — verify NOPAT and working capital changes are correct
- Sheet `DCF Model`: populate WACC, terminal growth rate, shares outstanding, net debt
- Sheet `Assumptions`: document key modelling assumptions
- Sheet `Cover Page`: company name + logo handling as above

### 5. Write output file

Save as:
- Simple: `simple-fcff-{company}.xlsx`
- Full: `fin-model-{company}.xlsx`

Then run formula recalculation:
```bash
python scripts/recalc.py <output_file>.xlsx 30
```

Fix any errors returned before presenting the file.

### 6. Cover Page handling

- **Company logo**: if the user provides an image, embed it as base64 in the Cover Page sheet using openpyxl's image insertion (`from openpyxl.drawing.image import Image`). If not provided, write the company name as styled text (bold, large font) in the same cell region.
- **Morus logo**: always add "morus" as text in a small cell top-right of the Cover Page, styled: black circle background, white text. Use openpyxl cell fill (`PatternFill`) and font to approximate this — a dark cell with white bold "morus" text is sufficient at this stage.
- **Date**: write today's date below the company name.
- **Currency and units**: write clearly (e.g. "USD' millions").

---

## Colour Coding (preserve template conventions)

| Colour | Meaning |
|---|---|
| Blue text | Hardcoded inputs — user-editable assumptions |
| Black text | Formulas and calculations |
| Green text | Cross-sheet links |
| Red text | External links |
| Red fill | Missing / undisclosed data |

---

## Number Formatting

- Thousands separator: commas
- Percentages: one decimal (e.g. 14.1%)
- Multiples: one decimal + x (e.g. 12.3x)
- Negatives: parentheses, not minus sign
- Zeros: display as "−"
- Years: text strings, not numbers

---

## Output Files

```
results/
├── simple-fcff-{company}.xlsx
└── fin-model-{company}.xlsx
```

---

## Notes

- This skill is for modelling only. No buy/sell signals, no price targets.
- Always append "Not investment advice · morus" to the Cover Page footer.
- If the user asks for written analysis after the model is built, that is a separate response — keep the file clean.
- If you cannot find a required line item in the source data, mark it missing (red fill) and note it to the user after presenting the file.
