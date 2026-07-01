# Morus Financial Display Skill
---
name: morus-financials
description: When given company financial data (uploaded document, image, or pasted text), extract the most recent balance sheet and income statement values and render them as a styled HTML artifact consistent with Morus brand standards.

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

When financial data is provided, extract:

### Income Statement
| Line Item | Notes |
|---|---|
| Revenue | Top-line / net revenue |
| Cost of Revenue / COGS | If disclosed |
| Gross Profit | Or derive from above |
| Operating Expenses | Total or broken out if meaningful |
| EBIT / Operating Income | |
| Interest Expense / Income | |
| Pre-tax Income | |
| Tax | |
| Net Income | Bottom line |
| EPS (Basic & Diluted) | If disclosed |
| Shares Outstanding | Weighted avg |
| EBITDA | If disclosed or derivable |

### Balance Sheet
| Line Item | Notes |
|---|---|
| Cash & Equivalents | |
| Short-term Investments | If disclosed |
| Total Current Assets | |
| PP&E (net) | |
| Intangibles / Goodwill | If material |
| Total Assets | |
| Total Current Liabilities | |
| Long-term Debt | |
| Total Liabilities | |
| Total Equity | |
| Book Value per Share | If derivable |

### Key Derived Ratios (auto-calculate if data allows)
- Gross Margin %
- Net Margin %
- Current Ratio
- Debt-to-Equity
- Return on Equity (if prior year equity available)

### Period Handling
- Always label the period clearly (e.g. "FY2024", "Q3 2025", "H1 2025")
- If multiple periods are present, show the **two most recent** side-by-side with a YoY % change column
- Currency: display as extracted (USD, ZAR, GBP, etc.) — label clearly

---

## HTML Output Template

Render as a single self-contained HTML artifact. Structure:

```
┌─────────────────────────────────────────────────────┐
│  [Company Logo + Name]          [morus circle logo]  │
│  Ticker · Exchange · Sector                          │
│  Period: FY2024  |  Currency: USD (millions)         │
├──────────────────────┬──────────────────────────────┤
│  INCOME STATEMENT    │  BALANCE SHEET               │
│  ─────────────────   │  ───────────────────         │
│  Revenue       1,234 │  Cash              456       │
│  Gross Profit    890 │  Current Assets    789       │
│  EBIT            234 │  Total Assets    2,345       │
│  Net Income      180 │  Current Liab.     321       │
│  EPS            0.45 │  LT Debt           500       │
│                      │  Total Equity    1,200       │
├──────────────────────┴──────────────────────────────┤
│  KEY METRICS                                         │
│  Gross Margin 72%  |  Net Margin 14.6%              │
│  Current Ratio 2.4x  |  D/E 0.42x                  │
└─────────────────────────────────────────────────────┘
│  Not investment advice · morus.research              │
└─────────────────────────────────────────────────────┘
```

### HTML Spec

```html
<!DOCTYPE html>
<html>
<head>
<style>
  :root {
    --bg: #111111;
    --surface: #1a1a1a;
    --border: #2a2a2a;
    --text-primary: #f0f0f0;
    --text-secondary: #888888;
    --accent: #ffffff;
    --positive: #4ade80;
    --negative: #f87171;
    --neutral: #94a3b8;
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    background: var(--bg);
    color: var(--text-primary);
    font-family: 'Inter', 'Helvetica Neue', Arial, sans-serif;
    font-size: 13px;
    padding: 24px;
    max-width: 860px;
    margin: 0 auto;
  }

  /* Header */
  .header {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    margin-bottom: 20px;
    padding-bottom: 16px;
    border-bottom: 1px solid var(--border);
  }

  .company-info .company-name {
    font-size: 22px;
    font-weight: 700;
    letter-spacing: -0.5px;
    color: var(--text-primary);
  }

  .company-info .company-meta {
    font-size: 11px;
    color: var(--text-secondary);
    margin-top: 4px;
    letter-spacing: 0.5px;
    text-transform: uppercase;
  }

  .company-info .period-label {
    font-size: 11px;
    color: var(--text-secondary);
    margin-top: 8px;
  }

  /* Morus logo — rendered in CSS, no image needed */
  .morus-logo {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 4px;
  }

  .morus-circle {
    width: 48px;
    height: 48px;
    background: #000000;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    border: 1px solid #333;
  }

  .morus-circle span {
    color: #ffffff;
    font-size: 10px;
    font-weight: 700;
    letter-spacing: 0.5px;
  }

  /* Two-column financial grid */
  .financials-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 16px;
    margin-bottom: 16px;
  }

  .panel {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 4px;
    padding: 16px;
  }

  .panel-title {
    font-size: 9px;
    font-weight: 700;
    letter-spacing: 1.5px;
    text-transform: uppercase;
    color: var(--text-secondary);
    margin-bottom: 12px;
    padding-bottom: 8px;
    border-bottom: 1px solid var(--border);
  }

  /* Financial table */
  .fin-table {
    width: 100%;
    border-collapse: collapse;
  }

  .fin-table tr { border: none; }

  .fin-table td {
    padding: 5px 0;
    font-size: 12px;
    vertical-align: baseline;
  }

  .fin-table td:first-child {
    color: var(--text-secondary);
    font-size: 11px;
    padding-right: 12px;
  }

  .fin-table td:last-child {
    text-align: right;
    font-variant-numeric: tabular-nums;
    color: var(--text-primary);
    font-weight: 500;
  }

  /* Two-period columns: label | period1 | period2 | change */
  .fin-table td.col-p1,
  .fin-table td.col-p2 {
    text-align: right;
    font-variant-numeric: tabular-nums;
    color: var(--text-primary);
    font-weight: 500;
    min-width: 60px;
  }

  .fin-table td.col-change {
    text-align: right;
    font-size: 10px;
    min-width: 44px;
    padding-left: 8px;
  }

  .positive { color: var(--positive); }
  .negative { color: var(--negative); }
  .neutral  { color: var(--neutral); }

  .fin-table tr.divider td {
    border-top: 1px solid var(--border);
    padding-top: 8px;
    margin-top: 4px;
  }

  .fin-table tr.highlight td {
    color: var(--text-primary);
    font-weight: 700;
  }

  .fin-table tr.highlight td:first-child {
    color: var(--text-primary);
  }

  /* Period header row inside table */
  .fin-table tr.period-header td {
    font-size: 9px;
    color: var(--text-secondary);
    text-transform: uppercase;
    letter-spacing: 0.5px;
    padding-bottom: 6px;
    border-bottom: 1px solid var(--border);
  }

  /* Key metrics bar */
  .metrics-bar {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 4px;
    padding: 12px 16px;
    display: flex;
    gap: 32px;
    flex-wrap: wrap;
    margin-bottom: 16px;
  }

  .metric-item { display: flex; flex-direction: column; gap: 2px; }

  .metric-label {
    font-size: 9px;
    text-transform: uppercase;
    letter-spacing: 1px;
    color: var(--text-secondary);
  }

  .metric-value {
    font-size: 16px;
    font-weight: 700;
    font-variant-numeric: tabular-nums;
    color: var(--text-primary);
  }

  /* Footer */
  .footer {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-top: 12px;
    border-top: 1px solid var(--border);
    font-size: 10px;
    color: var(--text-secondary);
  }
</style>
</head>
<body>

  <!-- HEADER -->
  <div class="header">
    <div class="company-info">
      <!-- Replace with company logo img tag if available, else use text -->
      <div class="company-name">COMPANY NAME</div>
      <div class="company-meta">TICKER · EXCHANGE · SECTOR</div>
      <div class="period-label">Period: FY2024 &nbsp;|&nbsp; Currency: USD (millions unless stated)</div>
    </div>
    <div class="morus-logo">
      <div class="morus-circle"><span>morus</span></div>
    </div>
  </div>

  <!-- KEY METRICS BAR -->
  <div class="metrics-bar">
    <div class="metric-item">
      <span class="metric-label">Gross Margin</span>
      <span class="metric-value">—%</span>
    </div>
    <div class="metric-item">
      <span class="metric-label">Net Margin</span>
      <span class="metric-value">—%</span>
    </div>
    <div class="metric-item">
      <span class="metric-label">Current Ratio</span>
      <span class="metric-value">—x</span>
    </div>
    <div class="metric-item">
      <span class="metric-label">Debt / Equity</span>
      <span class="metric-value">—x</span>
    </div>
    <div class="metric-item">
      <span class="metric-label">EPS (Diluted)</span>
      <span class="metric-value">—</span>
    </div>
    <div class="metric-item">
      <span class="metric-label">EBITDA</span>
      <span class="metric-value">—</span>
    </div>
  </div>

  <!-- TWO-COLUMN FINANCIALS -->
  <div class="financials-grid">

    <!-- INCOME STATEMENT -->
    <div class="panel">
      <div class="panel-title">Income Statement</div>
      <table class="fin-table">
        <!-- If two periods: uncomment period-header row and use col-p1/col-p2/col-change -->
        <tr><td>Revenue</td><td>—</td></tr>
        <tr><td>Cost of Revenue</td><td>—</td></tr>
        <tr class="divider highlight"><td>Gross Profit</td><td>—</td></tr>
        <tr><td>Operating Expenses</td><td>—</td></tr>
        <tr class="divider highlight"><td>EBIT</td><td>—</td></tr>
        <tr><td>Interest Expense</td><td>—</td></tr>
        <tr><td>Pre-tax Income</td><td>—</td></tr>
        <tr><td>Tax</td><td>—</td></tr>
        <tr class="divider highlight"><td>Net Income</td><td>—</td></tr>
        <tr class="divider"><td>EPS Basic</td><td>—</td></tr>
        <tr><td>EPS Diluted</td><td>—</td></tr>
        <tr><td>Shares Out. (wtd)</td><td>—</td></tr>
      </table>
    </div>

    <!-- BALANCE SHEET -->
    <div class="panel">
      <div class="panel-title">Balance Sheet</div>
      <table class="fin-table">
        <tr><td>Cash & Equivalents</td><td>—</td></tr>
        <tr><td>Short-term Investments</td><td>—</td></tr>
        <tr class="highlight"><td>Total Current Assets</td><td>—</td></tr>
        <tr><td>PP&E (net)</td><td>—</td></tr>
        <tr><td>Goodwill / Intangibles</td><td>—</td></tr>
        <tr class="divider highlight"><td>Total Assets</td><td>—</td></tr>
        <tr class="divider"><td>Total Current Liabilities</td><td>—</td></tr>
        <tr><td>Long-term Debt</td><td>—</td></tr>
        <tr class="highlight"><td>Total Liabilities</td><td>—</td></tr>
        <tr class="divider highlight"><td>Total Equity</td><td>—</td></tr>
        <tr class="divider"><td>Book Value / Share</td><td>—</td></tr>
      </table>
    </div>

  </div>

  <!-- FOOTER -->
  <div class="footer">
    <span>Source: Company filings. Figures may be rounded.</span>
    <span>Not investment advice &nbsp;·&nbsp; morus</span>
  </div>

</body>
</html>
```

---

## Workflow Instructions for Claude

When a user uploads or pastes company financials, follow these steps:

1. **Identify the source** — annual report, 10-K, earnings release, investor presentation, etc.
2. **Identify the most recent period** — confirm fiscal year end and reporting currency.
3. **Extract values** per the tables above. If a line item is not disclosed, mark it `—`. Do not estimate or fill with zeros.
4. **Check for two comparable periods** — if prior year data is present, show both and calculate YoY % change. Colour-code: green for improvement, red for deterioration, grey for flat/immaterial (<1%).
5. **Derive ratios** from extracted data. Do not pull ratios from the document — calculate them yourself from extracted figures so they are consistent.
6. **Handle company logo** — if the user provides a company logo image, embed it as a base64 `<img>` tag in the header left. If not provided, use the company name as styled text. Always render the Morus logo in CSS (black circle, white "morus" wordmark) top-right — no image file needed.
7. **Render the artifact** using the HTML template above, substituting all `—` placeholders with real values.
8. **Format numbers** consistently: use commas as thousands separators, round to the precision used in the source (e.g. if source uses 1 decimal place, keep that). State the unit (millions, billions, thousands) in the period label.
9. **Output only the HTML artifact** — no prose summary unless the user asks for analysis.

---

## Two-Period Column Variant

When two periods are available, replace single-value `<td>` columns with this pattern:

```html
<tr class="period-header">
  <td></td>
  <td class="col-p1">FY2023</td>
  <td class="col-p2">FY2024</td>
  <td class="col-change">YoY</td>
</tr>
<tr>
  <td>Revenue</td>
  <td class="col-p1">1,100</td>
  <td class="col-p2">1,234</td>
  <td class="col-change positive">+12%</td>
</tr>
```

Use `.positive`, `.negative`, or `.neutral` class on the change cell based on whether the movement is favourable (context-dependent: e.g. rising debt is negative even though the number is positive).

---

## Notes
- This skill is for display only. No valuation, no buy/sell signals.
- Always append "Not investment advice · morus" to the footer.
- If the user asks for analysis after the display, that's a separate response — keep the artifact clean.
