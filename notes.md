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

A single `index.html`: a sign-in screen, then a navy top bar and six numbered cards.

### Sign-in screen
- Email + password (Firebase Auth), with a toggle between "Sign in" and "Create an account".
- Each user sees only their own calculator (Firestore `calcs/{uid}`).

### Top bar
- Title "Net Profit Bonus Calculator" / "NOP Performance & Bonus Pool". No logo.
- Save status (Loading… / Saving… / Saved to your account / error) and the signed-in email.
- **Reset**: clears everything, including employee rows and weightings, after a confirm.
- **Clear**: clears the six top input fields only.
- **Export PDF**: `window.print()`.
- **Sign out**: flushes or awaits any pending save first.

### ① Target Revenue — Target vs Actual
- Annual Revenue Target and Actual Annual Revenue (editable, comma-formatted whole numbers).

### ② Net Operating Profit (NOP)
- Left: Baseline NOP % → Baseline NOP; NOP Target % → NOP Target; NOP Gain / Loss (Target).
- Right: [spacer row]; Actual NOP at Baseline % (= actual revenue × baseline %); Actual NOP % → Actual NOP; NOP Gain / Loss (Actual) = Actual NOP − Actual NOP at Baseline %.

### ③ Bonus Pool
- Toggle "Allocate to bonus pool". When on: % NOP gain → bonus rate, and a basis selector (actual or target gain).
- Result: NOP gain used, % applied, Total Bonus Pool (2 dp). Zero or negative gain → 0.00.

### ④ Employee Bonuses (collapsed on load)
- Columns: Name, Tenure (years, 2 dp), Position level (1–5, 1 dp), Owner elective (0–5, 1 dp; empty counts as 0), % share (1 dp), Bonus amount (2 dp).
- With no rows: a start panel offering Upload XLSX / CSV, or Skip upload — enter manually.
- Share and amount maths: see revision note 2026-09-30.

### ⑤ Weighting (collapsed on load)
- Tenure / Position level / Owner elective %. Each must be ≥ 0.5%, and together they must total exactly 100.0%.

### ⑥ Performance Summary
- 2×2 colour-coded cards: Revenue Attainment, NOP % Attainment, Actual NOP Gain, Total Bonus Pool.
- Green / amber / red thresholds: ≥ 100% / ≥ 85% / below.

---

## 3. Design Decisions

### Design System
This tool was built to match an existing VITAL design system established in a separate bid pricing worksheet (`index.html`). All visual decisions below were carried over from or are consistent with that system.

### Colour Palette
"Executive SaaS" palette. All colours are CSS custom properties on `:root`:

| Variable | Value | Usage |
|---|---|---|
| `--navy` | `#102A43` | Top bar, dark card headers, login hero |
| `--accent` | `#2F80ED` | Accent card headers, primary buttons, computed totals |
| `--teal` | `#2DD4BF` | Login hero accent, "saved" dot |
| `--bg-page` / `--bg-card` / `--bg-subtle` | `#F6F8FB` / `#FFFFFF` / `#F1F5F9` | Page, card, computed-field backgrounds |
| `--input-bg` | `#EBF4FE` | Editable input background |
| `--input-border` | `#DC2626` | Editable input border (red) — deliberate visual signal |
| `--amber` / `--red` | `#B45309` / `#B91C1C` | Warning / bad state, negative values |

**Note:** The red input border is intentional. It distinguishes editable fields from calculated outputs at a glance.

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
- The whole top bar (actions, save status, sign-out) and the add/upload/delete controls are hidden on print
- Collapsed cards print collapsed
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
| **Geist** | Font | Google Fonts `<link>` | Falls back to system sans-serif |
| **Courier New** | Font | System font | All numeric/computed values |
| **Firebase JS SDK 10.14.1** (app, auth, firestore) | ES modules | `https://www.gstatic.com/firebasejs/…` | Sign-in and per-user save |
| **SheetJS 0.18.5** | Script | cdnjs, lazy-loaded on first upload only | Reads XLSX / CSV |

No build step or bundler. Hosted on **GitHub Pages** (`grahamaskew.github.io/net-profit-bonus-calc`), and every push to `main` deploys.

**Firebase project:** `nop-bonus-calc-gmvz3` (Firestore `nam5`).
- Rules and the email/password provider are deployed from `firebase.json`: `firebase deploy --only firestore:rules,auth`.
- Authorized domains: localhost, the project's firebaseapp.com / web.app domains, and grahamaskew.github.io.

---

## 6. Known Issues / To Do

- **Firefox PDF headers:** `@page` suppresses print headers in Chrome, Edge and Safari. Firefox may still add them unless "Print headers and footers" is unchecked.
- **Edits made during "Loading…" are lost:** the saved copy replaces the screen when it arrives.
- **Load failure = read-only session:** if the saved copy can't be loaded, autosave stays off (so a blank calculator can't overwrite it) until reload.
- **No password reset / email verification:** sign-in is email + password only.
- **Input validation is limited in ①–③:** percentages above 100% or baseline > target are accepted. Card ④ and ⑤ inputs are validated.

---

## 7. Revision Notes

**2026-09-28 — Firebase sign-in + per-user save.** Mirrors `../revenue planner`. Email/password auth; each user's inputs autosave to Firestore `calcs/{uid}` (JSON in `stateJson`), locked by `firestore.rules`. No admin dashboard. Setup steps in `README.md`. Firebase config in `index.html` still holds `REPLACE_WITH_...` placeholders until the project is created.

---

**2026-09-30 — Employee Bonuses + Weighting cards; no currency symbol.**
- Cards ④ Employee Bonuses and ⑤ Weighting (Performance Summary is now ⑥). Both start collapsed on every page load.
- Weighting: three %s (Tenure, Position level, Owner elective). Each must be ≥ 0.5%, and together they must total exactly 100.0%. The check is done in integer tenths, so no float drift.
- Score = w_t·(tenure ÷ longest tenure) + w_p·(level ÷ 5) + w_o·(owner ÷ 5). Tenure is scaled so years don't swamp the 1–5 scales.
- % share = score ÷ sum of scores, so the shares always total 100%. Bonus amount = Total Bonus Pool × share, split in whole cents with largest remainder so the rows sum exactly to the pool.
- No shares appear until every non-blank row is valid (name, tenure ≥ 0, level 1–5, owner 0–5 or empty) and the weightings are valid. Fully blank rows are ignored.
- With no rows, the card shows a start panel: Upload XLSX / CSV, or Skip upload — enter manually (adds a first row). Deleting the last row brings the panel back.
- Upload, multi-tab workbooks: a tab picker opens with the last tab preselected.
- Header synonyms from the historical "Yearly Bonus Schedule" workbook: Years = Tenure, Extra Point(s) = Owner elective. With no Name header, the column left of Years is read as names.
- Import reads only the unbroken block under the header row, stopping at the first blank name or a Total row, so the summary blocks below are ignored.
- Out-of-range values (e.g. Position 0) import and are flagged.
- Upload (XLSX/CSV, SheetJS lazy-loaded from cdnjs) replaces all rows. It uses the first sheet and matches columns by header name. Only Name is required; unmatched columns are ignored.
- Rows and weightings autosave in the Firestore state (`employees`, `weights`). Reset also clears them (it asks first); Clear does not.
- Formats: %, level, owner → 1 dp; tenure, bonus amount and Total Bonus Pool (card ③ and summary) → 2 dp with commas. Other amounts stay whole numbers. The "$" symbol was removed from every value, input prefix and label.
