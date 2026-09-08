# Twelve-Company Financial Review (2009–2022)

A financial statement analysis project covering 12 companies — AAPL, MSFT, GOOG, AMZN, NVDA, INTC, MCD, PYPL, BCS, AIG, PCG, and SHLDQ — built in three layers from the same source data: a Python/SQL notebook, an interactive Power BI dashboard, and a standalone web app.

**Source data:** [Financial Statements of Major Companies 2009–2023](https://www.kaggle.com/datasets/rish59/financial-statements-of-major-companies2009-2023) (Kaggle)

```
notebook/      Jupyter notebook — SQL (via SQLite) + pandas + matplotlib
data/exports/  CSVs exported from the notebook, feeding both downstream layers
powerbi/       Power BI Desktop file (.pbix) — 5-page interactive dashboard
webapp/        Standalone HTML/JS app — no build step, no dependencies
```

---

## 1. Overview

The project asks ten questions of the same dataset, each building on the last:

1. Year-over-year revenue growth — who's growing fastest/slowest
2. Size vs. profitability — are the biggest companies also the most profitable
3. Returns (ROE / ROA / ROI) — combined rank, by company and year
4. ROE vs. ROA rank gap — flags leverage-driven return inflation
5. Debt/Equity trend — leverage over time
6. Earnings quality — operating cash flow vs. net income
7. EBITDA margin — revenue-to-EBITDA conversion efficiency
8. Valuation proxy — Market Cap ÷ Net Income, ranked rather than raw
9. Real vs. nominal growth — revenue growth after stripping out U.S. inflation
10. Composite quality score — combined ROE/ROA/cash-quality rank, with a win rate normalized by each company's years in the dataset

Each of the three layers (notebook, Power BI, web app) answers all ten questions independently, using the same underlying logic, so the numbers should match across all three — any place they don't is flagged below.

## 2. Cleaning & Processing

Work done in the notebook before any analysis:

- **Column names stripped** of leading/trailing whitespace
- **Null and duplicate rows checked** explicitly before proceeding
- **Loaded into an in-memory SQLite database**, with every metric built as a SQL query (window functions — `RANK()`, `LAG()` — rather than pandas, so the ranking logic is explicit and auditable)
- **Derived columns added in pandas** after each SQL pull: inverted ranks for the quadrant chart, three-way average rank for ROE/ROA/ROI, average nominal/real growth, and normalized win rate for the composite score

**Known data issue found and handled:** BCS (Barclays) reports a flat placeholder value of `1` in the source data's EBITDA column for every single year (2009–2022) — not a real figure. Left as-is, this produces a fabricated near-zero EBITDA margin rather than a missing value. BCS is excluded from the EBITDA margin view specifically, in both the Power BI report and the web app, with a visible note — it remains in every other chart, where its underlying data is valid.

**Fixes made after the first pass**, once specific charts turned out to need it:
- Several early charts used a single year (2022) as a snapshot; these were converted to multi-year averages (by company) or company × year heatmaps, since a single year understated how rankings shift over time
- The Debt/Equity and EBITDA margin views moved from an unreadable multi-line chart to a company × year heatmap
- Market Cap ÷ Net Income was switched from a raw ratio to a year-by-year rank, since the raw ratio spikes into the hundreds whenever a company's net income is small in a given year
- The composite score's "years ranked #1" was changed from a raw count to a win rate (wins ÷ years the company actually appears in the dataset), since a raw count rewarded companies with more years of data rather than genuinely consistent performers

## 3. Power BI

A 5-page interactive dashboard built from the exported CSVs, with Company and Year slicers synced across every page:

- **Overview** — size-vs-profitability quadrant scatter, composite win-rate bar, and four KPI cards (companies in view, year range, top performer, % of company-years that beat inflation)
- **Growth** — average YoY revenue growth, and nominal vs. real growth side by side
- **Returns & Efficiency** — ROE/ROA/ROI combined rank heatmap, and the ROE-vs-ROA rank gap chart
- **Leverage & Earnings Quality** — Debt/Equity heatmap (diverging color scale centered at zero), and cash flow vs. net income shown as distance from 1.0x parity
- **Margins, Valuations & Composite** — EBITDA margin heatmap (BCS excluded), valuation rank bar, and a supporting table with the raw wins/eligible-years/valuation-spread numbers behind the headline charts

Model setup notes: two dimension tables (`Dim_Company`, `Dim_Year`) relate to every fact table so the slicers filter consistently across pages; Power BI's automatic relationship detection created a few unwanted fact-to-fact relationships during setup, which were removed in favor of routing everything through the two dimension tables.

## 4. Bespoke Apps

**Ledger & Line** (`webapp/index.html`) — a standalone, single-file web app rebuilding all five Power BI pages as a self-contained HTML/JS app. No install, no server, no Power BI license required — open the file directly in any browser.

Differences from the Power BI version:
- Company and Year filters are fully live: since the app embeds row-level data rather than pre-aggregated numbers, every chart, heatmap, and ranking recalculates on the fly as filters change, rather than just hiding pre-computed figures
- A few fields missing from the exported CSVs at build time (`roa_rank`, `roi_rank`, `valuation_rank`) are recomputed client-side using the same ranking logic as the notebook

## 5. Conclusion (Findings)

- **Apple, Microsoft, Nvidia, and Intel** account for effectively all of the composite score's #1 rankings across 2009–2022 — no other company in the dataset wins even once on the combined ROE/ROA/cash-quality measure.
- **Size and profitability track together at the top, but not below it.** MSFT, AAPL, GOOG, and INTC are both larger and more profitable than the group median; MCD and NVDA show the opposite pattern — smaller, but more efficient than their size would suggest.
- **BCS carries the most leverage in the dataset** (debt roughly 5–9x equity most years) but still clears the cash-flow-quality bar — though as a bank, its cash flow is driven by trading and financing activity rather than normal operations, so this ratio is a weaker signal for BCS than for the other companies.
- **MCD and SHLDQ both show negative shareholder equity in several years, for opposite reasons.** Sears' (SHLDQ) negative equity reflects years of real losses on its way to bankruptcy; McDonald's reflects debt-funded share buybacks, not financial distress — the same raw metric, two very different underlying stories.
- **Growth and inflation-adjusted growth diverge most for the fastest-growing companies.** AMZN, GOOG, AAPL, and NVDA show the widest gap between nominal and real revenue growth — a meaningful share of their headline growth reflects economy-wide inflation rather than the business itself expanding.
- **Leverage inflates reported returns for several companies.** SHLDQ, AIG, AMZN, and BCS show ROE ranks noticeably better than their ROA ranks — a signal that debt, not operating strength, is doing more of the work behind their return figures.

---

## Running each piece

- **Notebook:** requires `pandas`, `numpy`, `matplotlib`, `sqlalchemy`, `kagglehub`. Running top to bottom re-downloads the source dataset and regenerates every CSV in `data/exports/`.
- **Power BI:** open `powerbi/Mock_Financial_Project.pbix` in Power BI Desktop (free). Data source is `data/exports/*.csv` — reconnect under Transform Data if that folder moves.
- **Web app:** open `webapp/index.html` directly in any browser.

## License / use

Mock analytical project for learning purposes — not investment advice.
