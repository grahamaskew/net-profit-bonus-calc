# VITAL Net Profit Bonus Calculator — Project Notes

> **For Claude Code:** Read this file at the start of any session involving `index.html`. It documents the full context, design decisions, structure, and revision history for this project. Do not make changes to `index.html` without first reviewing this file.

---

## 1. Project Overview

**What it is:** A single-file, self-contained HTML calculator tool built for VITAL.

**Purpose:** Allows users to model net operating profit (NOP) targets against actual performance, and optionally calculate a bonus pool based on NOP gain. The tool is designed to be used interactively in a browser and exported to PDF for reporting.

**Intended audience:** Internal business users — likely finance, operations, or leadership — who are working with revenue targets, NOP benchmarks, and bonus allocation on a per-period basis.

**Source:** Built from a source XLSX file (`Net_Profit__bonus_calculator.xlsx`) containing the original calculation logic, and styled to match a pre-existing VITAL design system established in a separate `index.html` bid pricing worksheet.

---

## 2. Page Structure

The page is a single `index.html` file with no external scripts. It is divided into four numbered card sections, plus a persistent header:

### Header
- VITAL logo (embedded base64 WebP)
- Subtitle: "Net Profit Bonus Calculator"
- Three action buttons: **Reset** (clears all fields and resets defaults), **Clear** (clears input fields only), **Export PDF** (triggers `window.print()`)

### Section ① — Target Revenue vs Actual
- Two-column comparison layout
- Left: Annual Revenue Target (editable currency input)
- Right: Actual Annual Revenue (editable currency input)
- Legend explaining editable vs calculated fields

### Section ② — Net Operating Profit (NOP)
- Two-column comparison layout, rows aligned across both columns using a hidden spacer row on the right
- **Left column (NOP Targets):**
  - Baseline NOP % (editable) → Baseline NOP $ (calculated)
  - NOP Target % (editable) → NOP Target $ (calculated)
  - NOP Gain / Loss (Target) (calculated)
- **Right column (Actual NOP):**
  - [Spacer row — aligns with Baseline NOP % input]
  - Actual NOP at Baseline % = Actual Revenue × Baseline NOP % (calculated)
  - Actual NOP % (editable) → Actual NOP $ (calculated)
  - NOP Gain / Loss (Actual) = Actual NOP $ − Actual NOP at Baseline $ (calculated)

### Section ③ — Bonus Pool
- Toggle (show/hide) labelled "Allocate $ to bonus pool"
- When ON, reveals a two-column bonus section:
  - Left: % NOP gain → bonus (editable rate), Applied to selector (Actual or Target NOP Gain)
  - Right: NOP gain used (calculated), % NOP gain applied (calculated), Total Bonus Pool $ (calculated)
- Bonus is only calculated when NOP gain is positive; zero or negative gain yields no bonus

### Section ④ — Performance Summary
- 2×2 grid of colour-coded metric cards:
  - **Revenue Attainment** — Actual / Target revenue
  - **NOP % Attainment** — Actual NOP % / Target NOP %
  - **Actual NOP Gain** — dollar gain above baseline
  - **Total Bonus Pool** — bonus $ if allocated
- Cards turn green (good), amber (warn), or red (bad) based on thresholds

---

## 3. Design Decisions

### Design System
This tool was built to match an existing VITAL design system established in a separate bid pricing worksheet (`index.html`). All visual decisions below were carried over from or are consistent with that system.

### Colour Palette
All colours are defined as CSS custom properties on `:root`:

| Variable | Value | Usage |
|---|---|---|
| `--vital-green` | `#6687e1` | Primary accent, card headers, buttons, computed totals |
| `--vital-green-dark` | `#4a6bc4` | Button hover state |
| `--vital-green-light` | `#eef1fc` | Highlighted card backgrounds |
| `--bg` | `#f5f6fd` | Page background |
| `--surface` | `#ffffff` | Card surface |
| `--surface-alt` | `#f4f6fd` | Right comparison column, computed field backgrounds |
| `--input-bg` | `#cfe2f3` | Editable input background (blue tint) |
| `--input-border` | `#cc0000` | Editable input border (red) — deliberate visual signal |
| `--amber` | `#b45309` | Warning state |
| `--red` | `#b91c1c` | Error / bad state, negative values |

**Note:** The red input border (`#cc0000`) is intentional — it distinguishes editable fields from calculated outputs at a glance and matches the original design system.

### Typography
- **Body / UI font:** Geist (loaded from Google Fonts), weights 300–700
- **Computed values / numbers:** Courier New (monospace) — provides clear visual separation between labels and numeric outputs, and ensures number alignment

### Layout
- Max content width: 900px, centred
- Two-column comparison grid used for Sections ①, ②, and ③ to allow side-by-side target vs actual reading
- Section ② uses a hidden spacer row (`visibility: hidden; pointer-events: none`) on the right column so that "Actual NOP %" aligns horizontally with "NOP Target %" across the grid
- Metric cards use a 2×2 CSS grid

### Input Philosophy
Only five fields are user-editable:
1. Annual Revenue Target
2. Actual Annual Revenue
3. Baseline NOP %
4. NOP Target %
5. Actual NOP %

Plus two bonus parameters (rate % and basis selector) when the bonus pool is toggled on. All other values are calculated and displayed as read-only computed spans.

### PDF Export
- `window.print()` is used for PDF export
- `@page { margin: 0.5in; size: auto; }` suppresses the browser's native print header (title, URL, date/time stamp)
- The Export PDF and other action buttons are hidden on print via `.header-actions { display: none; }`
- The tool's own `<header>` (logo + subtitle) is retained and visible in the PDF output
- `print-color-adjust: exact` is applied to metric cards to preserve background colours in print

---

## 4. Key Prompts

The following prompts from the build session most significantly shaped the final result:

**Establishing the design system:**
> "Carefully review the attached index.html file and retain ALL of the formatting."

**Defining the source data:**
> "Yes, I'll upload it now" [uploading `Net_Profit__bonus_calculator.xlsx`] + "A different calculator or worksheet"

**Setting input/output rules:**
> "Revenue targets and NOP % are editable; rest calculated"

**Toggle behaviour:**
> "Show/hide a bonus pool breakdown section"

**Actual NOP at Baseline formula:**
> "Actual Revenue × Baseline NOP %"

**Row alignment:**
> "Just make the row heights/positions visually line up across both columns"

**PDF header suppression:**
> "Remove it entirely (suppress title, URL, and date/time)" [referring to the browser's native print header]

---

## 5. Dependencies

| Dependency | Type | How loaded | Notes |
|---|---|---|---|
| **Geist** | Font | Google Fonts CDN (`@import url(...)`) | Weights 300, 400, 500, 600, 700. Requires internet connection to load; falls back to `sans-serif` |
| **Courier New** | Font | System font | Used for all numeric/computed values. No external load required |

**No JavaScript libraries.** All interactivity (calculations, toggle, currency formatting, colour logic) is written in vanilla JS in a single inline `<script>` block at the bottom of `index.html`.

**No build tools, bundlers, or frameworks.** The file is entirely self-contained and can be opened directly in any modern browser.

---

## 6. Known Issues / To Do

- **Firefox PDF headers:** The `@page` CSS rule suppresses browser print headers in Chrome, Edge, and Safari. Firefox respects the margin but may still render its own header/footer depending on user print settings. Users on Firefox may need to manually uncheck "Print headers and footers" in the print dialog.

- **Negative value formatting:** `fmtDollar()` renders negative numbers as `$-20,000` (JavaScript's native `toLocaleString()` behaviour) rather than `-$20,000`. This is cosmetically non-standard but functionally correct. Negative gain values are additionally highlighted in red via the `.negative` CSS class, so the visual treatment is unambiguous. Could be addressed with a sign-check and string manipulation if strict formatting is required.

- **Logo:** The VITAL logo is embedded as a base64 WebP string copied from the original bid pricing worksheet. If the logo changes, the base64 string in the `<img src="...">` tag in the `<header>` will need to be updated.

- **No input validation:** Percentage inputs accept values above 100% or below 0%. There is no user-facing error for nonsensical combinations (e.g. Baseline NOP % > NOP Target %). This is by design for flexibility but could be revisited if guardrails are needed.

- **No persistent state:** The tool does not save state between sessions. All inputs are cleared on page reload. If persistence is needed, `localStorage` or a backend would be required.

---

## 7. Revision Notes

_This section is reserved for future revision entries. Leave blank until revisions are made post-deployment._

---
