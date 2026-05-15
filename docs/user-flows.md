# User Flows - EPM Application

## 👤 Flow 1: Nouvel Utilisateur - Sign Up & Setup

```
┌─────────────────────────────────────────────────────────────┐
│ 1. LANDING PAGE                                             │
│ - Marketing & features                                      │
│ - "Start Free" CTA                                          │
└────────────────┬────────────────────────────────────────────┘
                 │ Click "Start Free"
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. SIGN UP FORM                                             │
│ Fields:                                                     │
│ - Email                                                     │
│ - Password                                                  │
│ - First Name                                                │
│ - Last Name                                                 │
│ - Organization Name                                         │
│ - Country                                                   │
│ - Accounting Standard (PCG/IFRS/GAAP)                      │
└────────────────┬────────────────────────────────────────────┘
                 │ Submit
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. EMAIL VERIFICATION                                       │
│ - Send verification email                                   │
│ - User clicks link                                          │
│ - Email confirmed                                           │
└────────────────┬────────────────────────────────────────────┘
                 │ Email verified
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. ONBOARDING TUTORIAL                                      │
│ - Welcome screen                                            │
│ - Feature overview (2-3 slides)                             │
│ - Skip option                                               │
└────────────────┬────────────────────────────────────────────┘
                 │ Complete or skip
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. DASHBOARD (Empty State)                                  │
│ - Welcome message                                           │
│ - "Upload your first CSV" CTA (prominent)                   │
│ - Getting started guide                                     │
│ - Help resources                                            │
└──────────────────────────────────────────────────────────────┘
```

## 📤 Flow 2: Import CSV Balance Comptable

```
┌─────────────────────────────────────────────────────────────┐
│ 1. IMPORT PAGE                                              │
│ - Upload zone (drag & drop)                                 │
│ - Click to browse files                                     │
│ - Supported formats info                                    │
│ - Sample CSV download link                                  │
└────────────────┬────────────────────────────────────────────┘
                 │ User selects CSV file
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. FILE PARSING                                             │
│ Backend checks:                                             │
│ - File size                                                 │
│ - Format (CSV/XLSX)                                         │
│ - Charset detection                                         │
│ - Separator detection (,/;)                                 │
│ - Decimal detection (./,)                                   │
└────────────────┬────────────────────────────────────────────┘
                 │ Parsing complete
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. PREVIEW & VALIDATION                                     │
│ Show:                                                       │
│ - First 10 rows of data                                     │
│ - Detected separator & decimal                              │
│ - Column mapping (Compte, Debit, Credit)                    │
│ - Total records: 1,250                                      │
│ - Validation status:                                        │
│   ✓ All columns present                                     │
│   ✓ Debit/Credit format valid                               │
│   ✓ All 1,250 rows valid                                    │
│ - Edit separator/decimal if needed                          │
└────────────────┬────────────────────────────────────────────┘
                 │ Validate
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. IMPORT DETAILS                                           │
│ Form:                                                       │
│ - Import Name: "Balance comptable Janvier 2026"             │
│ - Description (optional)                                    │
│ - Period Start: 2026-01-01                                  │
│ - Period End: 2026-01-31                                    │
│ - Processing options:                                       │
│   □ Auto-map accounts                                       │
│   □ Generate financial statements                           │
│   □ Calculate KPIs                                          │
└────────────────┬────────────────────────────────────────────┘
                 │ Confirm import
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. PROCESSING                                               │
│ Progress bar:                                               │
│ - Parsing CSV: ████████░░░░░░░░░░░ 40%                      │
│ - Mapping accounts: ████████████░░░░░░░░░░ 60%              │
│ - Validating data: ████████████████░░░░░░░░ 80%             │
│ - Generating statements: ███████████████████░░ 95%          │
│ - Calculating KPIs: ████████████████████░░░░░ 100%          │
│                                                             │
│ Estimated time: 2-3 seconds                                │
│ Cancel option available                                     │
└────────────────┬────────────────────────────────────────────┘
                 │ Processing complete
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 6. SUCCESS & SUMMARY                                        │
│ ✓ Import successful                                         │
│                                                             │
│ Summary:                                                    │
│ - Period: Janvier 2026                                      │
│ - Rows imported: 1,250                                      │
│ - Total debit: €500,000                                     │
│ - Total credit: €500,000                                    │
│ - Status: Balanced ✓                                        │
│                                                             │
│ Buttons:                                                    │
│ - View Period Data → Dashboard                              │
│ - Review Mappings → Mapping Config                          │
│ - Upload Another → Back to Import                           │
└──────────────────────────────────────────────────────────────┘
```

## 🎯 Flow 3: Review & Configure Account Mapping

```
┌─────────────────────────────────────────────────────────────┐
│ 1. MAPPING OVERVIEW                                         │
│ Table:                                                      │
│ ┌──────────┬────────────┬─────────────────┬────────────┐   │
│ │ Account  │ Label      │ Auto-Category   │ Status     │   │
│ ├──────────┼────────────┼─────────────────┼────────────┤   │
│ │ 601000   │ Achats MP  │ COGS            │ ✓ Correct  │   │
│ │ 602000   │ Services   │ COGS            │ ! Review   │   │
│ │ 702000   │ Ventes     │ Revenue         │ ✓ Correct  │   │
│ │ 401000   │ Clients    │ Accounts Rec.   │ ✓ Correct  │   │
│ └──────────┴────────────┴─────────────────┴────────────┘   │
│                                                             │
│ Filters:                                                    │
│ - Show all / Needs review / Correct                         │
│ - Search by account code                                    │
│ - Filter by category                                        │
└────────────┬─────────────────────────────────────────────────┘
             │ Click on row to edit
             ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. EDIT MAPPING MODAL                                       │
│                                                             │
│ Account Code: 602000                                        │
│ Account Label: Services                                     │
│                                                             │
│ Category Selection (dropdown):                              │
│ ☑ Cost of Goods Sold (auto-detected)                        │
│ ☐ Operating Expenses                                        │
│ ☐ Marketing & Sales                                         │
│ ☐ Custom category                                           │
│                                                             │
│ Confidence: 85%                                             │
│ Notes: "Services are outsourced - consider as OpEx"         │
│                                                             │
│ Buttons: Save | Cancel | Reset to Auto                     │
└────────────────┬────────────────────────────────────────────┘
                 │ Save changes
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. BULK ACTIONS                                             │
│ - Select multiple accounts                                  │
│ - Bulk assign category                                      │
│ - Bulk confirm mappings                                     │
│ - Undo changes                                              │
│                                                             │
│ Footer: "3 of 250 mappings need review"                     │
│         "All mappings reviewed" ✓                           │
└──────────────────────────────────────────────────────────────┘
```

## 📊 Flow 4: View Dashboard & Financial Analysis

```
┌─────────────────────────────────────────────────────────────┐
│ 1. DASHBOARD HEADER                                         │
│ ┌────────────────────────────────────────────────────────┐ │
│ │ Financial Overview - Janvier 2026                     │ │
│ │                                                        │ │
│ │ Period selector: [Janvier 2026 ▼]                      │ │
│ │ Comparison: [vs. Décembre 2025 ▼]                     │ │
│ │ View: [Monthly ▼] [Dashboard | Detailed | Export]    │ │
│ └────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ 2. KPI CARDS (Top Section)                                  │
│ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐         │
│ │ Revenue      │ │ EBITDA %     │ │ Gross Margin │         │
│ │ €125,000 ↑   │ │ 44.0% ↑      │ │ 64.0% ↑      │         │
│ │ +13.6% MoM   │ │ +4 pts MoM   │ │ +4 pts MoM   │         │
│ └──────────────┘ └──────────────┘ └──────────────┘         │
│                                                             │
│ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐         │
│ │ Cash Position│ │ Burn Rate    │ │ Current Ratio│         │
│ │ €150,000     │ │ €5,000/month │ │ 1.87x        │         │
│ │ ✓ Stable     │ │ ▼ Improving  │ │ ✓ Healthy    │         │
│ └──────────────┘ └──────────────┘ └──────────────┘         │
└──────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ 3. FINANCIAL STATEMENT CARDS                                │
│                                                             │
│ ┌──────────────────────────────────┐                       │
│ │ INCOME STATEMENT - Jan 2026      │                       │
│ ├──────────────────────────────────┤                       │
│ │ Revenues          €125,000 100%  │                       │
│ │  Product Sales    €95,000  76%   │                       │
│ │  Service Rev.     €30,000  24%   │                       │
│ │                                  │                       │
│ │ Cost of Goods      -€45,000 -36% │                       │
│ │                                  │                       │
│ │ Gross Profit       €80,000  64%  │                       │
│ │                                  │                       │
│ │ Operating Exp.     -€25,000 -20% │                       │
│ │                                  │                       │
│ │ EBITDA             €55,000  44%  │◄─ Drill-down option │
│ └──────────────────────────────────┘                       │
│                                                             │
│ ┌──────────────────────────────────┐                       │
│ │ BALANCE SHEET - Jan 2026         │                       │
│ ├──────────────────────────────────┤                       │
│ │ ASSETS            €400,000       │                       │
│ │  Current Assets   €150,000  38%  │                       │
│ │  Fixed Assets     €250,000  62%  │                       │
│ │                                  │                       │
│ │ LIABILITIES       €200,000       │                       │
│ │  Current Liab.    €80,000   40%  │                       │
│ │  Long-term Liab.  €120,000  60%  │                       │
│ │                                  │                       │
│ │ EQUITY            €200,000  50%  │                       │
│ └──────────────────────────────────┘                       │
└──────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ 4. CHARTS SECTION                                           │
│                                                             │
│ ┌──────────────────────┐  ┌──────────────────────┐         │
│ │ Revenue Trend (6M)   │  │ Expense Breakdown    │         │
│ │                      │  │                      │         │
│ │ €125k │                │  │ COGS      45%       │         │
│ │ €120k │ /              │  │ OpEx      20%       │         │
│ │ €110k │/ ╲ ╱ ╱╲      │  │ Other     35%       │         │
│ │ €100k ├──╱──╲        │  │                      │         │
│ │       └─────────      │  │                      │         │
│ └──────────────────────┘  └──────────────────────┘         │
│                                                             │
│ ┌──────────────────────┐  ┌──────────────────────┐         │
│ │ EBITDA % Evolution   │  │ Cash Flow Analysis   │         │
│ │                      │  │                      │         │
│ │ 44% ┐                │  │ Jan: €12k (positive)│         │
│ │ 42% │ ╱─             │  │ Feb: €8k  (positive)│         │
│ │ 40% │╱               │  │ Mar: €5k  (positive)│         │
│ │ 38% └                │  │ Runway: 30 months   │         │
│ │ 36%                  │  │                      │         │
│ └──────────────────────┘  └──────────────────────┘         │
└──────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ 5. INSIGHTS & ALERTS                                        │
│                                                             │
│ 📈 Revenue growing strong                                   │
│    Total revenue increased from €110k (Dec) to €125k (Jan), │
│    a growth of +13.6%. Product sales remain the main driver.│
│                                                             │
│ ⚠️  Operating expenses increased                             │
│    OpEx jumped to €25k, up from €20k last month. This is   │
│    primarily due to increased marketing spend (€3.5k vs €2k)│
│    [View details]                                           │
│                                                             │
│ ✓ Profitability improved                                    │
│    EBITDA margin reached 44%, up from 41% in December.     │
│                                                             │
│ 💡 Recommendation                                           │
│    Consider reviewing marketing ROI given the spending      │
│    increase. Expected payback: Q2-Q3 2026.                 │
└──────────────────────────────────────────────────────────────┘
```

## 🔍 Flow 5: Drill-Down Analysis

```
User clicks on "EBITDA" in Income Statement
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ EBITDA Breakdown - €55,000                                  │
│                                                             │
│ Structure:                                                  │
│ Revenues                    €125,000                         │
│ ├─ Product Sales            €95,000                          │
│ │  ├─ Compte 702001         €60,000                          │
│ │  ├─ Compte 702002         €25,000                          │
│ │  └─ Compte 702003         €10,000                          │
│ └─ Service Revenue          €30,000                          │
│    ├─ Compte 701001         €20,000                          │
│    └─ Compte 701002         €10,000                          │
│                                                             │
│ Operating Expenses         -€70,000                         │
│ ├─ Cost of Goods           -€45,000                         │
│ │  ├─ Compte 601000        -€25,000                         │
│ │  ├─ Compte 602000        -€15,000                         │
│ │  └─ Compte 603000        -€5,000                          │
│ └─ Operating Costs         -€25,000                         │
│    ├─ Compte 606000        -€10,000                         │
│    ├─ Compte 608000        -€8,000                          │
│    └─ Compte 609000        -€7,000                          │
│                                                             │
│ = EBITDA                    €55,000                          │
│                                                             │
│ [Chart showing breakdown]                                   │
│ [Comparison with previous period]                           │
│ [Export / Download]                                         │
└──────────────────────────────────────────────────────────────┘
```

## 📈 Flow 6: Period Comparison Analysis

```
┌─────────────────────────────────────────────────────────────┐
│ COMPARE: Jan 2026 vs Dec 2025                               │
│                                                             │
│ ┌────────┬──────────┬──────────┬──────────┬────────────┐   │
│ │ Metric │ Jan 2026 │ Dec 2025 │ Change   │ Change %   │   │
│ ├────────┼──────────┼──────────┼──────────┼────────────┤   │
│ │ Revenue│ €125,000 │ €110,000 │ +€15,000 │ +13.6% ↑   │   │
│ │ COGS   │ -€45,000 │ -€42,000 │ -€3,000  │ -7.1% ↑    │   │
│ │ GP Margin│64.0% │ 61.8%   │ +2.2pts  │ +3.6% ↑   │   │
│ │ OpEx   │ -€25,000 │ -€20,000 │ -€5,000  │ -25.0% ↑   │   │
│ │ EBITDA │ €55,000  │ €50,000  │ +€5,000  │ +10.0% ↑   │   │
│ │ EBITDA%│ 44.0%    │ 45.5%    │ -1.5pts  │ -3.3% ↓    │   │
│ └────────┴──────────┴──────────┴──────────┴────────────┘   │
│                                                             │
│ Analysis:                                                   │
│ • Revenue increased 13.6% YoY driven by both products      │
│ • COGS increased but GP margin improved (more efficient)   │
│ • OpEx jumped 25% - investigate drivers                    │
│ • EBITDA up 10% absolute but margin slightly compressed   │
└──────────────────────────────────────────────────────────────┘
```
