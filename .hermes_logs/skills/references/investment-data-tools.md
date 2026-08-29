# Investment Research Data Acquisition — Per-Role Tool Integration

> Updated: 2026-07-24 — ai-berkshire 89-commit sync review
> Reference: `~/GitHub/ai-berkshire/tools/`

## Overview

Ai-Berkshire zero-dependency Python tools (stdlib + curl only) integrate
via Hermes's `terminal` toolset:

| Tool | Function | Data Source |
|------|----------|-------------|
| `ashare_data.py` | Real-time quotes, financials, valuation, search | 腾讯行情 + 东方财富 |
| `financial_rigor.py` | Exact decimal: market cap, valuation, cross-validate, 3-scenario, Benford, calc | Pure computation (Decimal p=28) |
| `report_audit.py` | Data audit: 15% extract, validate, pass/fail verdict | Pure computation |
| `twstock_data.py` | Taiwan stock: quote, financials, valuation, monthly revenue, dividends | FinMind API |

## Per-Role Tool Allocation

| Role | Tools | Rationale |
|------|-------|-----------|
| 巴菲特 | terminal (full suite), web, file | Data-driven core: PE/PB/ROE, three-scenario, cross-validation |
| 芒格 | terminal (financials + calc), web, file | Competitor comparison, growth rate verification |
| 李录 | terminal (financials + calc), web, file | Risk signal quantification (receivables, inventory, goodwill) |
| 段永平 | web, file only | "毛估估" philosophy — data tools break character |
| 主持人 | terminal (calc + report_audit.py), web, file | Currency unification, data quality gate |

## Currency Conversion Rules

1. Internal analysis: use reporting currency (Tencent→HKD, Moutai→CNY, Apple→USD)
2. Cross-market only: convert to unified currency (CNY/USD) for comparison
3. Annotate: `HK$4,650亿（≈ US$595亿，1 USD=7.82 HKD，2026-07-01）`
4. Use `financial_rigor.py calc` — never floating-point
5. HKD pegged to USD (7.75-7.85 band)
6. Historical rates for historical data — not current rates

## Exact Decimal Requirement

**NEVER mental math.** All calculations via `financial_rigor.py`'s exact
decimal engine. Python float introduces cumulative drift.

## Cross-Validation

Two independent sources per metric. ≤1% ✅ / 1-5% ⚠️ / >5% ❌.
GAAP vs Non-GAAP MUST be noted.

## invest_skills/ File Structure

```
~/invest_skills/
├── duan-yongping.md      (221 lines) — 段永平, 生意本质
├── warren-buffett.md     (319 lines) — 巴菲特, 护城河/财务/估值
├── charlie-munger.md     (235 lines) — 芒格, 逆向/竞争
├── li-lu.md              (240 lines) — 李录, 文明趋势/长期风险
└── team-lead.md          (258 lines) — 主持人, 调度/汇总/共识
```

Sections: 身份 → 核心框架 → 人格特征 → 群聊互动规则 → 数据获取能力 → 原始来源

## ai-berkshire Updates (2026-07-24 — 89 commits ahead)

### ashare_data.py 52-Week Bug Fix 🚨

Fields 47/48 were 涨停/跌停, NOT 52-week extremes. Fixed in d92286c —
now fetches from 东方财富 f174/f175. Must `git pull` before deploying bots.

### financial-data.md 复权 Rules

- 不复权: current snapshot only
- 前复权: historical comparison (use this)
- 后复权: total return / CAGR
- Never mix adjusted/unadjusted in same analysis

### New Tools
- `twstock_data.py`: Taiwan stock (FinMind API)
- `star_history_chart.py`: Star chart

### New Skills
- `income-investment.md`: Dividend/income analysis
- `thesis-drift.md`: Thesis drift detection (Improved/Unchanged/Weakened)
