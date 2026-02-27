# multi-asset-portfolio-tracker — Project Instructions

**Version:** 2.4.1
**App Version:** 2.4.1
**Last Updated:** 2026-02-27

> Bump both version numbers on every change. Use semantic versioning — major.minor.patch.
> - **Major** — new asset type, new core feature
> - **Minor** — new field, new UI section, new export feature
> - **Patch** — wording fix, sample data update, small tweak

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 2.4.1 | 2026-02-27 | Excel Live Prices sheet falls back to last successfully fetched price (not export-time price) on API failure |
| 2.4.0 | 2026-02-27 | Excel Live Prices sheet uses WEBSERVICE()+FILTERXML() formulas for auto-refreshable live prices |
| 2.3.0 | 2026-02-27 | Excel fetches latest prices at export time via API for Equity, MF, NPS, SGB, Commodity |
| 2.2.0 | 2026-02-27 | README.md & CHANGELOG.md maintained in source, About tab in app, Changelog sheet in Excel export |
| 2.1.0 | 2026-02-27 | Current Price Per Unit column in app summary & each asset sheet; Excel current value via named cell approach |
| 2.0.0 | 2026-02-27 | Live price API integration (MF, NPS, Equity, SGB), current value in summary, added SGB asset type |
| 1.6.0 | 2026-02-27 | Added yearly investment bar chart (stacked by asset type) to Dashboard |
| 1.5.0 | 2026-02-27 | Added Commodity & Property asset types, weight field, rental income tracking |
| 1.4.0 | 2026-02-27 | Added sample data for 3 persons across all asset types |
| 1.3.0 | 2026-02-27 | Added Excel export with SheetJS, per-asset sheets, summary sheet, formulas |
| 1.2.0 | 2026-02-27 | Excel-like sheet UI, inline editing, blank row pinned at top |
| 1.1.0 | 2026-02-27 | Multi-person support, PersonManager, usePersons hook |
| 1.0.0 | 2026-02-27 | Initial project instructions |

---

## Project Overview

Build a React single-page application to record and track personal investment transactions across multiple persons and the following asset classes:

- **Equity** (stocks, shares)
- **Mutual Fund (MF)**
- **NPS** (National Pension System)
- **PPF** (Public Provident Fund)
- **EPF** (Employee Provident Fund)
- **FD** (Fixed Deposit)
- **Commodity** (gold, silver, etc.)
- **SGB** (Sovereign Gold Bond)
- **Property** (real estate)

---

## Person Management

- The app supports **multiple persons** (e.g. family members, partners)
- Each person has only two fields:
  - **Name** — full name (string, required)
  - **Label** — short identifier tag (string, e.g. "self", "spouse", "child", required)
- Persons are managed via a simple list (add / edit / delete)
- Every transaction is linked to a person via their ID
- The transaction list and dashboard can be filtered by person

---

## Tech Stack

- React (with hooks: useState, useEffect, useContext)
- localStorage for persistence (no backend required)
- CSS Modules or Tailwind CSS for styling
- Optional: Recharts or Chart.js for portfolio summary charts

---

## Core Features

### 1. Transaction Entry Form
Each transaction must capture:

| Field | Type | Notes |
|---|---|---|
| Date | date | Default to today |
| Asset Type | dropdown | Equity, MF, NPS, PPF, EPF, FD, Commodity, SGB, Property |
| Transaction Type | dropdown | Varies by asset (see below) |
| Description | text | Stock name, fund name, bank name, etc. |
| Amount (₹) | number | Total transaction value |
| Units | number | Only for Equity and MF |
| Price per Unit | number | Only for Equity and MF |
| Interest Rate (%) | number | Only for FD |
| Weight (grams) | number | Only for Commodity |
| Notes | textarea | Optional free-text |

**Transaction types per asset class:**

- Equity → Buy, Sell, Dividend
- Mutual Fund → Buy, Sell, SIP, Dividend
- NPS → Invest, Withdraw
- PPF → Deposit, Withdraw, Interest Credit
- EPF → Deposit, Withdraw, Interest Credit
- FD → Deposit, Maturity, Premature Withdrawal, Interest Credit
- Commodity → Buy, Sell
- SGB → Buy, Sell, Interest Credit
- Property → Buy, Sell, Rental Income

### 2. Transaction List View
- Display all transactions in a table/card list, sorted by date (newest first)
- Columns: Date, Asset Type, Transaction Type, Description, Amount, Units/Price (if applicable), **Current Price/Unit** (live, read-only — shown for Equity, MF, NPS, SGB, Commodity rows)
- Allow Edit and Delete on each row
- Filter by Asset Type (All / Equity / MF / NPS / PPF / EPF / FD / Commodity / SGB / Property)
- Search by description keyword

### 3. Portfolio Summary Dashboard
- Total invested amount across all asset classes
- Total withdrawn/redeemed amount
- Net invested (invested − withdrawn)
- Per-asset breakdown: invested, withdrawn, net, current value, unrealised gain/loss, number of transactions
- **Current Price Per Unit** column shown in the summary table for each asset that supports live pricing (Equity, MF, NPS, SGB, Commodity) — fetched from the live price API and displayed alongside current value
- Commodity: track weight (grams) alongside value
- SGB: track weight (grams) and live gold price
- Property: track rental income separately from capital gains
- **Current Value** column in summary fetched via free APIs (see Live Price API section below)
- Pie/bar chart of allocation by asset type
- **Yearly Investment Bar Chart** — stacked bar chart showing total invested per year, broken down by asset type:
  - X-axis: years (derived from transaction dates)
  - Y-axis: total amount invested (₹)
  - Each bar stacked by asset type, colored using ASSET_COLORS
  - Togglable between Invested only, Withdrawn only, and Net views
  - Filter by person (inherits the active person selector)
  - Built using Recharts `BarChart` with `ResponsiveContainer`

### 4. Live Price API Integration

Fetch current market prices to calculate **Current Value** and **Unrealised Gain/Loss** in the summary dashboard. Use free public APIs — no API key required where possible.

| Asset Type | API | Endpoint | Notes |
|---|---|---|---|
| Equity (NSE/BSE) | `https://query1.finance.yahoo.com/v8/finance/chart/{symbol}.NS` | Yahoo Finance (no key) | Append `.NS` for NSE, `.BO` for BSE |
| Mutual Fund | `https://api.mfapi.in/mf/{schemeCode}` | mfapi.in (free, no key) | Returns NAV history; use latest entry |
| NPS | `https://npscra.nsdl.co.in` | NSDL NPS NAV (scrape or use mfapi proxy) | Use scheme code stored in description |
| SGB | `https://query1.finance.yahoo.com/v8/finance/chart/SGBFEB32.NS` | Yahoo Finance | Use relevant SGB series symbol |
| Commodity (Gold/Silver) | `https://www.goldapi.io/api/XAU/INR` | GoldAPI (free tier) or fallback to Yahoo Finance `GC=F` | Convert troy oz to grams (1 troy oz = 31.1 g) |

**Implementation rules:**
- Store the ticker/scheme code in the transaction `description` field or a new optional `ticker` field on the transaction object
- Fetch prices on Dashboard load; cache results in component state for the session (do not persist to localStorage)
- Show a loading spinner per asset card while price is being fetched
- If fetch fails, show last known invested value with a warning badge ("Live price unavailable")
- Current Value = units held × latest price
- Unrealised Gain/Loss = Current Value − Net Invested; show in green (gain) or red (loss)
- Assets without live price support (PPF, EPF, FD, Property) show "N/A" for Current Value

**New field on transaction object (optional):**
```js
ticker: "HDFCBANK.NS"   // Optional — NSE/BSE symbol, MF scheme code, or SGB series
```

### 5. Data Persistence
- Save all transactions to localStorage under the key `portfolio_transactions`
- Save all persons to localStorage under the key `portfolio_persons`
- Load both from localStorage on app start
- Provide an Export to JSON button (downloads a .json file)
- Provide an Import from JSON button (reads a .json file and merges/replaces transactions)

### 6. Excel Export
- Provide an **Export to Excel (.xlsx)** button in the Toolbar
- Use the **SheetJS (xlsx)** library to generate the workbook client-side
- The exported workbook must mirror the look and feel of the React app:
  - One sheet per asset type (Equity, MF, NPS, PPF, EPF, FD, Commodity, SGB, Property) plus a **Summary** sheet, a **Live Prices** sheet, and a **Changelog** sheet — matching the app's tab structure
  - Each asset sheet has the same columns as the React table for that asset type
  - Header row styled with the asset's color (matching ASSET_COLORS), bold white text
  - Alternating row fill colors to match the app's zebra striping
  - Column widths set to match the app's proportions
- Each asset sheet must include **Excel formulas** (not hardcoded values) for:
  - A **Total Invested** row at the bottom: `SUMIF` on transaction type column matching invested types (buy, invest, deposit, SIP)
  - A **Total Withdrawn** row: `SUMIF` matching withdrawn types (sell, withdraw, maturity, premature withdrawal)
  - A **Net** row: Total Invested − Total Withdrawn
  - A **Current Price/Unit** column (read-only, pre-filled at export time with the latest fetched price — labelled "Price at export")
  - A **Current Value** column using an Excel formula: `=units_held × current_price_cell` — referencing the Live Prices sheet so the user can manually update prices to recalculate
- The **Live Prices sheet**:
  - A dedicated sheet with columns: **Ticker / Scheme Code**, **API URL**, **Raw Response**, **Current Price (₹)**, **Last Known Price (₹)** (hidden), **Last Fetched At**
  - One row per unique ticker found in transactions
  - **Uses Excel's built-in `WEBSERVICE()` + `FILTERXML()` formulas to fetch and parse live prices directly inside Excel** (Windows Excel only; Mac/web fall back to export-time baked prices)
  - The React app writes the ticker and pre-built API URL into each row at export time; Excel evaluates the formulas when the file is opened or recalculated (Ctrl+Alt+F9)
  - **Formula approach per asset type:**
    - **Mutual Fund (mfapi.in returns JSON-like plain text):**
      - API URL cell: `https://api.mfapi.in/mf/{schemeCode}/latest`
      - Raw response via: `=WEBSERVICE(B2)` (mfapi returns plain text NAV value)
      - Price cell: `=VALUE(TRIM(C2))` — parse the returned NAV number
    - **Equity / SGB (Yahoo Finance v8 returns JSON; use the older v7 CSV endpoint instead):**
      - API URL cell: `https://query1.finance.yahoo.com/v7/finance/download/{symbol}.NS?period1=0&period2=9999999999&interval=1d&events=history`
      - Price cell: `=VALUE(FILTERXML(WEBSERVICE("https://query1.finance.yahoo.com/v8/finance/chart/"&A2&"?interval=1d"), "//regularMarketPrice"))` — note: Yahoo Finance may block WEBSERVICE calls due to CORS; in that case fall back to export-time baked price with a note
    - **Commodity / Gold:**
      - Use Yahoo Finance symbol `GC=F` (Gold Futures, USD) via same WEBSERVICE approach, then convert: `=price_in_USD * USD_INR_rate / 31.1035` (troy oz to grams)
      - USD/INR rate fetched separately: `=WEBSERVICE("https://query1.finance.yahoo.com/v8/finance/chart/USDINR=X?interval=1d")`
  - **Fallback strategy — last known good price (not export-time price):**
    - The Live Prices sheet has an additional hidden column: **Last Known Price (₹)** — this stores the most recent successfully fetched price
    - The formula in the **Current Price** column is: `=IFERROR(VALUE(fresh_fetched_price), LastKnownPrice)`
    - When a fresh WEBSERVICE fetch succeeds, a macro or formula chain updates the Last Known Price column with the new value
    - When a fetch fails (API down, no internet, Mac/Web), the Last Known Price column retains the previous successful value and is used instead
    - Implementation: use a helper column with `=IF(ISNUMBER(fresh_price), fresh_price, D2)` where D2 is the Last Known Price — Excel will preserve the last good value as long as the file is saved after each successful fetch
    - On first export, Last Known Price is seeded with the export-time price fetched by the React app — so there is always a valid fallback from day one
  - Named ranges defined per ticker (e.g. `HDFCBANK_NS_PRICE`) pointing to the **Current Price** column — all asset sheet and Summary formulas reference these named ranges
  - Label clearly: "Prices auto-refresh via WEBSERVICE() on Windows Excel (Ctrl+Alt+F9). On failure, last successfully fetched price is used. On Mac/Web, save the file after each successful fetch to preserve prices."
  - The React app fetches and seeds the Last Known Price column at export time as the initial fallback value
- The **Summary sheet** must:
  - List all 9 asset types as rows with columns: Asset, Invested, Withdrawn, Net, Units Held, Current Price/Unit, Current Value, Unrealised G/L, No. of Transactions
  - **Current Price/Unit** and **Current Value** columns reference the Live Prices sheet via named ranges — auto-update when user edits Live Prices sheet
  - Unrealised G/L = Current Value − Net Invested, formatted in green (positive) or red (negative) using conditional formatting
  - Use `SUMIF` / `COUNTA` formulas referencing the individual asset sheets — so editing data in any sheet auto-updates the Summary
  - Include a grand total row at the bottom summing all assets
  - Match the app's dashboard card style: asset color fills in the Asset column cells
- All currency cells formatted as `₹#,##0.00` (Indian locale)
- Date cells formatted as `DD-MMM-YYYY`
- Price cells formatted as `₹#,##0.0000` (4 decimal places for NAV/unit prices)
- The **Changelog sheet** must:
  - Mirror the contents of `CHANGELOG.md` — one row per version entry
  - Columns: Version, Date, Description
  - Header row styled with a neutral dark background and white bold text
  - Rows listed newest-first (descending version order)
  - Read-only informational sheet — no formulas needed
  - Sheet tab labeled "Changelog"

---

## Data Model

```js
// Person object
{
  id: 1710000000000,           // Date.now() as unique ID
  name: "Rahul Sharma",        // Full name
  label: "self"                // Short tag: self, spouse, child, etc.
}

// Single transaction object
{
  id: 1710000000001,           // Date.now() as unique ID
  personId: 1710000000000,     // Reference to person.id
  date: "2024-03-10",          // ISO date string YYYY-MM-DD
  assetType: "Equity",         // One of the 9 asset types
  transactionType: "buy",      // Type specific to asset class
  description: "HDFC Bank",    // Free text
  amount: 15000,               // In INR
  units: 10,                   // Optional, for Equity/MF
  price: 1500,                 // Optional, for Equity/MF
  interestRate: null,          // Optional, for FD
  weight: null,               // Optional, grams — for Commodity/SGB
  ticker: null,               // Optional, symbol for live price lookup
  notes: ""                    // Optional free text
}
```

---

## Component Structure

```
App
├── Header
├── PersonManager            # Add / Edit / Delete persons (simple list UI)
├── Toolbar                  # Person selector dropdown + Export button
├── AssetSheetTabs           # Tab bar — All / Equity / MF / NPS / PPF / EPF / FD / Commodity / SGB / Property
├── SpreadsheetTable         # The active asset sheet
│   ├── TableHeader          # Sticky column headers
│   ├── NewTransactionRow    # Blank editable row pinned at top
│   └── TransactionRow (xN)  # Each saved row, inline-editable, with delete
├── Dashboard                # Summary cards + charts (separate view/tab)
│   ├── SummaryCard (x9)     # One per asset type, shows current value + unrealised G/L
│   ├── AllocationChart      # Pie chart — allocation by asset type
│   ├── YearlyInvestmentChart  # Stacked bar chart — invested per year by asset type
│   └── LivePriceBadge       # Per-card badge showing live price fetch status
├── ImportExport             # JSON import/export + Excel export buttons
└── AboutPage                # Renders README content + full changelog inside the app
```

---

## Business Logic

- When `transactionType` changes based on `assetType`, reset `transactionType` to the first valid option for that asset.
- For Equity/MF: if both `units` and `price` are entered, auto-calculate `amount = units × price` (or allow manual override).
- For summary calculations:
  - **Invested** = sum of amounts where transactionType is in [buy, invest, deposit, SIP]
  - **Withdrawn** = sum of amounts where transactionType is in [sell, withdraw, redeem, maturity, premature withdrawal]
  - **Interest/Dividend/Rental Income** credited = tracked separately, not added to invested total
  - **Commodity**: also track total weight (grams) held = weight bought − weight sold
  - **SGB**: track weight (grams); fetch live gold price to compute current value
  - **Property**: track rental income separately; capital gain = sell amount − buy amount
  - **Current Value** = units/weight held × latest live price (fetched via API)
  - **Unrealised Gain/Loss** = Current Value − Net Invested

---

## UI/UX Guidelines

### Excel-like Sheet Interface
- The main view is a **tabbed spreadsheet** — one tab/sheet per asset type (Equity, MF, NPS, PPF, EPF, FD, Commodity, SGB, Property), plus an "All" tab
- Each tab renders a full-width **editable table** resembling a spreadsheet:
  - Rows are **directly editable inline** — clicking any cell activates an input/select in place
  - A **blank new row is always pinned at the top** of the table; new transactions are entered there and saved in reverse chronological order (newest first)
  - Each row has a **row number column** (like Excel) and a **delete button** on the far right
  - Columns are fixed per asset type sheet, sized appropriately (narrow for date/type, wider for description/amount)
- Sheet tabs styled like spreadsheet tabs, with the active tab highlighted in the asset's color
- A **Person selector** dropdown sits above the sheet tabs to switch between persons; "All Persons" is the default

### Editing Behavior
- Clicking a cell puts it in edit mode (input or select rendered in place)
- Pressing **Enter** or **Tab** confirms the edit and moves focus to the next cell
- Pressing **Escape** cancels the edit and reverts the cell
- Editing the blank top row and pressing Enter saves it as a new transaction; a fresh blank row appears at the top automatically
- Changes are auto-saved to localStorage on each cell blur/confirm
- Show a confirmation on row delete (e.g. highlight row red, click delete icon again to confirm)

### Visual Style
- Dense, data-heavy layout — minimal padding, small font (12–13px), compact row height
- Alternating row background colors for readability
- Sticky header row with column labels
- Sticky row-number column when scrolling horizontally
- Color-coded asset sheet tab indicators matching ASSET_COLORS constants
- No modals or separate form pages — all editing is inline in the table

---

## File Structure

```
README.md                        # Project overview, setup instructions, feature list
CHANGELOG.md                     # Full version history with dated entries
src/
├── components/
│   ├── Header.jsx
│   ├── PersonManager.jsx        # Add / Edit / Delete persons
│   ├── Toolbar.jsx              # Person selector + Export button
│   ├── AssetSheetTabs.jsx       # Sheet tab bar
│   ├── SpreadsheetTable.jsx     # Renders the active asset sheet table
│   ├── NewTransactionRow.jsx    # Blank row pinned at top for new entries
│   ├── TransactionRow.jsx       # Editable saved transaction row
│   ├── Dashboard.jsx
│   ├── SummaryCard.jsx
│   ├── AboutPage.jsx            # Renders README + CHANGELOG inside the app
│   └── ImportExport.jsx         # JSON import/export + Excel export (SheetJS)
├── constants/
│   └── assetTypes.js        # Asset type names, colors, transaction type options
├── hooks/
│   ├── useTransactions.js   # localStorage CRUD for transactions
│   ├── usePersons.js        # localStorage CRUD for persons
│   └── useLivePrices.js     # Fetches live prices from free APIs per asset type
├── utils/
│   └── calculations.js      # Summary/totals calculation functions
├── App.jsx
└── main.jsx
```

---

## Constants to Define (assetTypes.js)

```js
export const ASSET_TYPES = ["Equity", "Mutual Fund", "NPS", "PPF", "EPF", "FD", "Commodity", "SGB", "Property"];

export const ASSET_COLORS = {
  "Equity": "#FF6B35",
  "Mutual Fund": "#4ECDC4",
  "NPS": "#45B7D1",
  "PPF": "#96CEB4",
  "EPF": "#F7DC6F",
  "FD": "#C39BD3",
  "Commodity": "#F0C040",
  "SGB": "#E8B84B",
  "Property": "#7FB3D3"
};

export const TRANSACTION_TYPES = {
  "Equity": ["buy", "sell", "dividend"],
  "Mutual Fund": ["buy", "sell", "SIP", "dividend"],
  "NPS": ["invest", "withdraw"],
  "PPF": ["deposit", "withdraw", "interest credit"],
  "EPF": ["deposit", "withdraw", "interest credit"],
  "FD": ["deposit", "maturity", "premature withdrawal", "interest credit"],
  "Commodity": ["buy", "sell"],
  "SGB": ["buy", "sell", "interest credit"],
  "Property": ["buy", "sell", "rental income"]
};

export const INVESTED_TYPES = ["buy", "invest", "deposit", "SIP"];
export const WITHDRAWN_TYPES = ["sell", "withdraw", "maturity", "premature withdrawal"];
```

---

## README & CHANGELOG Files

### README.md
Maintain a `README.md` at the project root with:
- **Project name & description** — what the app does, who it's for
- **Features list** — bullet list of all major features (asset types, live prices, Excel export, multi-person, etc.)
- **Tech stack** — React, SheetJS, Recharts, localStorage
- **Getting started** — how to run locally (`npm install`, `npm run dev`)
- **How to use** — brief guide: adding persons, recording transactions, reading the dashboard, exporting
- **API sources** — list of free APIs used for live prices with links
- **Version** — current app version (keep in sync with project instructions version)

### CHANGELOG.md
Maintain a `CHANGELOG.md` at the project root with:
- Format: `## [version] — YYYY-MM-DD` heading per release, followed by bullet points of changes
- Newest version at the top
- Must be updated every time a new feature, fix, or asset type is added
- Example entry format:
```
## [2.2.0] — 2026-02-27
- Added README.md and CHANGELOG.md to source
- Added About tab in app showing README and changelog
- Added Changelog sheet to Excel export
```

### In-app About Page (`AboutPage.jsx`)
- Accessible via an **"About"** tab or link in the app header/toolbar
- Renders two sections:
  - **About this app** — contents of README (feature list, usage guide, API sources)
  - **Version history** — full changelog table (Version | Date | Changes), newest first
- Styled to match the app's visual theme
- App version number shown in the Header at all times (e.g. `v2.2.0` badge)

---

## Getting Started

When asked to build this app, start with:

1. `README.md` and `CHANGELOG.md` at project root
2. `assetTypes.js` constants file
3. `useTransactions.js` custom hook (localStorage CRUD)
4. `usePersons.js` custom hook (localStorage CRUD)
5. `calculations.js` utility functions
6. `SpreadsheetTable.jsx` with inline editing and new row at top
7. `Dashboard.jsx` with summary cards
8. `AboutPage.jsx` rendering README + changelog
9. `ImportExport.jsx` with JSON and Excel export
10. Wire everything in `App.jsx`

---

## Example Prompts to Use in This Project

- "Build the SpreadsheetTable component with inline editing and blank row pinned at top"
- "Create the useTransactions and usePersons hooks with localStorage persistence"
- "Build the Dashboard with summary cards for each asset type"
- "Add JSON import/export functionality"
- "Build the Excel export using SheetJS with per-asset sheets, summary sheet, formulas, and color styling matching the app"
- "Add a pie chart showing portfolio allocation by asset type using Recharts"
- "Create README.md and CHANGELOG.md for the project"
- "Build the AboutPage component showing app info and version history"
- "Add the Changelog sheet to the Excel export"
- "Make the app mobile responsive"

---

## Sample Data

Pre-populate the app with the following data on first load (only if localStorage is empty). Use `Date.now()` style IDs incremented by 1.

### Persons

```js
const SAMPLE_PERSONS = [
  { id: 1700000000001, name: "Rahul Sharma",  label: "self"   },
  { id: 1700000000002, name: "Priya Sharma",  label: "spouse" },
  { id: 1700000000003, name: "Aryan Sharma",  label: "child"  }
];
```

### Transactions

```js
const SAMPLE_TRANSACTIONS = [

  // Equity — Rahul (self)
  { id: 1700000001001, personId: 1700000000001, date: "2024-01-10", assetType: "Equity",      transactionType: "buy",      description: "HDFC Bank",        amount: 15000, units: 10,  price: 1500, interestRate: null, notes: "" },
  { id: 1700000001002, personId: 1700000000001, date: "2024-02-15", assetType: "Equity",      transactionType: "buy",      description: "Infosys",          amount: 28000, units: 20,  price: 1400, interestRate: null, notes: "" },
  { id: 1700000001003, personId: 1700000000001, date: "2024-03-20", assetType: "Equity",      transactionType: "sell",     description: "HDFC Bank",        amount: 8000,  units: 5,   price: 1600, interestRate: null, notes: "Partial exit" },

  // Mutual Fund — Priya (spouse)
  { id: 1700000002001, personId: 1700000000002, date: "2024-01-05", assetType: "Mutual Fund", transactionType: "SIP",      description: "Axis Bluechip Fund",    amount: 5000,  units: 120.5, price: 41.49, interestRate: null, notes: "" },
  { id: 1700000002002, personId: 1700000000002, date: "2024-02-05", assetType: "Mutual Fund", transactionType: "SIP",      description: "Axis Bluechip Fund",    amount: 5000,  units: 118.2, price: 42.30, interestRate: null, notes: "" },
  { id: 1700000002003, personId: 1700000000002, date: "2024-03-10", assetType: "Mutual Fund", transactionType: "buy",      description: "Mirae Asset ELSS",      amount: 25000, units: 580.0, price: 43.10, interestRate: null, notes: "Tax saving" },

  // NPS — Rahul (self)
  { id: 1700000003001, personId: 1700000000001, date: "2024-01-31", assetType: "NPS",         transactionType: "invest",   description: "NPS Tier 1",       amount: 10000, units: null, price: null, interestRate: null, notes: "Monthly contribution" },
  { id: 1700000003002, personId: 1700000000001, date: "2024-02-28", assetType: "NPS",         transactionType: "invest",   description: "NPS Tier 1",       amount: 10000, units: null, price: null, interestRate: null, notes: "Monthly contribution" },
  { id: 1700000003003, personId: 1700000000002, date: "2024-03-31", assetType: "NPS",         transactionType: "invest",   description: "NPS Tier 2",       amount: 5000,  units: null, price: null, interestRate: null, notes: "" },

  // PPF — Aryan (child)
  { id: 1700000004001, personId: 1700000000003, date: "2024-01-15", assetType: "PPF",         transactionType: "deposit",  description: "PPF - SBI",        amount: 10000, units: null, price: null, interestRate: null, notes: "Annual deposit" },
  { id: 1700000004002, personId: 1700000000003, date: "2024-02-20", assetType: "PPF",         transactionType: "deposit",  description: "PPF - SBI",        amount: 5000,  units: null, price: null, interestRate: null, notes: "" },
  { id: 1700000004003, personId: 1700000000003, date: "2024-03-31", assetType: "PPF",         transactionType: "interest credit", description: "PPF - SBI", amount: 1243,  units: null, price: null, interestRate: null, notes: "FY2024 interest" },

  // EPF — Rahul (self)
  { id: 1700000005001, personId: 1700000000001, date: "2024-01-31", assetType: "EPF",         transactionType: "deposit",  description: "EPF - EPFO",       amount: 6500,  units: null, price: null, interestRate: null, notes: "Jan salary deduction" },
  { id: 1700000005002, personId: 1700000000001, date: "2024-02-29", assetType: "EPF",         transactionType: "deposit",  description: "EPF - EPFO",       amount: 6500,  units: null, price: null, interestRate: null, notes: "Feb salary deduction" },
  { id: 1700000005003, personId: 1700000000001, date: "2024-03-31", assetType: "EPF",         transactionType: "interest credit", description: "EPF - EPFO", amount: 2340,  units: null, price: null, interestRate: null, notes: "FY2024 interest" },

  // Commodity — Rahul (self)
  { id: 1700000007001, personId: 1700000000001, date: "2024-01-25", assetType: "Commodity", transactionType: "buy",           description: "Gold - SGB",      amount: 56000,  units: null, price: null, interestRate: null, weight: 10,   notes: "Sovereign Gold Bond" },
  { id: 1700000007002, personId: 1700000000001, date: "2024-02-18", assetType: "Commodity", transactionType: "buy",           description: "Silver - Physical",amount: 9500,   units: null, price: null, interestRate: null, weight: 100,  notes: "Physical silver coins" },
  { id: 1700000007003, personId: 1700000000001, date: "2024-03-22", assetType: "Commodity", transactionType: "sell",          description: "Gold - SGB",      amount: 29500,  units: null, price: null, interestRate: null, weight: 5,    notes: "Partial redemption" },

  // Property — Rahul (self)
  { id: 1700000008001, personId: 1700000000001, date: "2024-01-01", assetType: "Property",  transactionType: "buy",           description: "2BHK Pune",       amount: 6500000, units: null, price: null, interestRate: null, weight: null, notes: "Registration done" },
  { id: 1700000008002, personId: 1700000000001, date: "2024-02-01", assetType: "Property",  transactionType: "rental income", description: "2BHK Pune",       amount: 18000,   units: null, price: null, interestRate: null, weight: null, notes: "Monthly rent" },
  { id: 1700000008003, personId: 1700000000001, date: "2024-03-01", assetType: "Property",  transactionType: "rental income", description: "2BHK Pune",       amount: 18000,   units: null, price: null, interestRate: null, weight: null, notes: "Monthly rent" },

  // FD — Priya (spouse)
  { id: 1700000006001, personId: 1700000000002, date: "2024-01-20", assetType: "FD",          transactionType: "deposit",  description: "HDFC Bank FD",     amount: 100000, units: null, price: null, interestRate: 7.25, notes: "1 year tenure" },
  { id: 1700000006002, personId: 1700000000002, date: "2024-02-10", assetType: "FD",          transactionType: "deposit",  description: "SBI FD",           amount: 50000,  units: null, price: null, interestRate: 6.80, notes: "6 month tenure" },
  { id: 1700000006003, personId: 1700000000002, date: "2024-03-15", assetType: "FD",          transactionType: "interest credit", description: "SBI FD",    amount: 1020,   units: null, price: null, interestRate: null, notes: "Quarterly interest" }
];
```

### Loading Logic

```js
// In App.jsx or useTransactions.js / usePersons.js
if (!localStorage.getItem("portfolio_persons")) {
  localStorage.setItem("portfolio_persons", JSON.stringify(SAMPLE_PERSONS));
}
if (!localStorage.getItem("portfolio_transactions")) {
  localStorage.setItem("portfolio_transactions", JSON.stringify(SAMPLE_TRANSACTIONS));
}
```
