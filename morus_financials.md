---
name: morus-financials
description: When given company financial data (uploaded document, image, or pasted text), extract the most recent balance sheet and income statement values and render them as a styled HTML artifact consistent with Morus brand standards.
---

## Input

Pdf files, images or pasted text: With balance sheet and income statement and other key financial information.

---

## Brand Identity

**Morus** — Independent equity research, reasoning and a concentrated portfolio.

### Visual Token System

| Token | Value | Usage |
|---|---|---|
| `--bg` | `#111111` | Page background |
| `--surface` | `#1a1a1a` | Card/panel background |
| `--border` | `#2a2a2a` | Dividers, table borders |
| `--text-primary` | `#f0f0f0` | Headlines, key figures |
| `--text-secondary` | `#888888` | Labels, units, subtitles |
| `--accent` | `#ffffff` | Morus logo circle fill, emphasis |
| `--positive` | `#4ade80` | Positive growth indicators |
| `--negative` | `#f87171` | Negative / loss indicators |
| `--neutral` | `#94a3b8` | Neutral / flat indicators |

**Typography:** Use system font stack: `'Inter', 'Helvetica Neue', Arial, sans-serif`. Numbers in tabular figures (`font-variant-numeric: tabular-nums`).

**Morus logo:** Rendered in-code as a black circle (`#000`) with white bold wordmark "morus" — no image dependency needed. Place top-right always.

---

## Extraction Rules

When financial data is provided, extract: the values as per the templates in the `assets`

### Period Handling
- Always label the period clearly (e.g. "FY2024", "Q3 2025", "H1 2025")
- If multiple periods are present, show the **two most recent** side-by-side with a YoY % change column
- Currency: display as extracted (USD, ZAR, GBP, etc.) — label clearly

---

## Workflow Instructions for Claude

When a user uploads or pastes company financials, follow these steps:

1. **Identify the source** — annual report, 10-K, earnings release, investor presentation, etc.
2. **Identify the most recent period** — confirm fiscal year end and reporting currency.
3. **Extract values** per the templates above. If a line item is not disclosed, mark it `—`. And fill in a transparent red. Do not estimate or fill with zeros.
4. **Check for two comparable periods** — if prior year data is present, show both and calculate YoY % change. Colour-code: green for improvement, red for deterioration, grey for flat/immaterial (<1%).
5. **Derive ratios** from extracted data. Do not pull ratios from the document — calculate them yourself from extracted figures so they are consistent.
6. **Handle company logo** — if the user provides a company logo image, embed it as a base64 `<img>` tag in the cover page tab of the excel template. If not provided, use the company name as styled text. Always render the Morus logo in CSS (black circle, white "morus" wordmark) top-right — no image file needed.
7. **Format numbers** consistently: use commas as thousands separators, round to the precision used in the source (e.g. if source uses 1 decimal place, keep that). State the unit (millions, billions, thousands) in the period label.
9. **Output only the ouput files** — no prose summary unless the user asks for analysis.

---

## Output Format Guidelines

**For specific excel ouputs format**: 
For simple and full finanncial models use the following templates (in `assets/` folder):
- simple-fcff-model.xlsx - simple free cashflow to the firm  model with formatting and calculation structure
- fin-model.xlsx - three statement financial model with formatting and calculation structure

---

## Output Files

Replace {company name} with the company name being analyzed

```
results/
├── simple-fcff-{company name}.xlsx      # Simple financial model template populated with company values
├── fin-model-{comapany name}.xlsx       # Full financial model template populated with comapny values
└── summary-{comapny name}.pdf           # Summary of key financial from models
```


## Notes
- This skill is for display only. No valuation, no buy/sell signals.
- Always append "Not investment advice · morus" to the footer.
- If the user asks for analysis after the display, that's a separate response — keep the artifact clean.

