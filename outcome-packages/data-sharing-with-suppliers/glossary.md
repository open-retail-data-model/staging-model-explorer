# Data Sharing with Suppliers — Business Glossary

> Status: 🟡 In progress (0.1-beta) · Last reviewed: 2026-06-09

Business terms for the Category Growth use case. Vendor-neutral; follows the ORDM [data model principles](../../docs/data-model-standards.md).

## Domain Brief — placement decision (Phase 0)

The Category Growth use case sits in the **Data Sharing with Suppliers** column but reads like merchandising/assortment analytics. Two interpretations were considered:

- **A — collaborative** (chosen): category growth analysed **with and attributed to suppliers** (joint business planning), reusing the supplier scorecard and procurement spend. Matches the column.
- **B — merchandising**: pure internal category performance, independent of supplier.

**Decision: Interpretation A.** It matches the column, no pre-existing category/assortment asset implied B, and the supplier scorecard + procurement data exist to support supplier attribution. `gold_category_growth` therefore exposes a `supplier_contribution` (top supplier's share of category procurement spend + scorecard composite) alongside the merchandising metrics. The merchandising metrics (decomposition, share) stand on their own, so the view is still useful if the supplier signal is absent.

## Object

| Object | Grain | Description |
|---|---|---|
| `gold_category_growth` | One category × fiscal period | Category performance, growth decomposition, and the integrated promo / customer-value / supplier signals. |

## Terms

| Term | Definition |
|---|---|
| **Category growth** | Period-over-period change in category revenue (`delta_revenue = category_revenue − prior_period_revenue`), with `pop_growth_pct` and `yoy_growth_pct` comparing **like fiscal periods** (period_index = `fiscal_year*12 + fiscal_period`; PoP = index−1, YoY = index−12). |
| **Growth decomposition** | `delta_revenue` split into four effects that **reconcile exactly**. With D = distribution points (distinct product × store), q = units/point, P = avg price (subscripts 0=prior, 1=current): **distribution** `= (D1−D0)·q0·P0`, **volume** `= D1·(q1−q0)·P0`, **price** `= U1·Σ_sc w1·(p1−p0)`, **mix** `= U1·Σ_sc (w1−w0)·p0` (over sub-categories sc, w = unit share, p = sub-category avg price). distribution + volume + price + mix = `delta_revenue`. |
| **Distribution effect** | Revenue change from broader/narrower availability (more or fewer product × store selling points). |
| **Volume effect** | Revenue change from selling more/fewer units per selling point, holding price and mix. |
| **Price effect** | Like-for-like price change within sub-categories. |
| **Mix effect** | Revenue change from shifting the sub-category mix toward higher/lower-priced sub-categories at constant sub-category prices. |
| **Category share** | `category_revenue / total revenue` for the period; sums to ~1.0 across categories. |
| **Promo contribution** | Incremental margin from `gold_promo_roi` attributable to the category that period (the share of growth that was promo-driven). NULL if the promo view is absent. |
| **Value-tier mix** | `value_share_platinum/gold/silver/bronze` — the split of customer-attributed category revenue across CLV value tiers (`gold_customer_ltv`), showing whether growth comes from high- or low-value customers. NULL if the LTV view is absent. |
| **Supplier contribution** (Interpretation A) | `top_supplier_id`, `supplier_top_share` (share of category procurement spend) and `supplier_top_score` (its scorecard composite). NULL if procurement/scorecard is absent. |

## Media Measurement (supplier media reporting)

The supplier-facing publish + entitlement layer over the Commerce Media Networks fact layer — how a retail-media
network shares measured campaign performance with each supplier/advertiser under access control, with an
immutable audit trail. Routes the media-measurement brief.

| Object | Grain | Description |
|---|---|---|
| `reporting_entitlement` | one version per (principal, advertiser, campaign-scope) | Row-level access anchor: which governance principal may see which advertiser (optionally one campaign) + the minimum disclosure threshold. SCD2. |
| `supplier_media_report_snapshot` | one publish snapshot × brand × campaign × level | Immutable frozen report — what was disclosed to a supplier at a point in time (audit/dispute), sourced from `campaign_day`. |
| `gold_supplier_media_report` | principal × brand × campaign | Entitled supplier report (row-filter source): the CMN `campaign_day` rolled to brand×campaign, joined to `reporting_entitlement`. No person-level data. |

| Term | Definition |
|---|---|
| **Reporting entitlement** | A grant letting a principal (UC group / account / service principal — not a customer) see a supplier's media results, brand-wide or campaign-scoped. |
| **Disclosure threshold** | Minimum aggregation count a row must meet before it is shared with a supplier (privacy/competitive protection). |
| **Published snapshot** | An immutable, versioned freeze of the numbers shared with a supplier, so a later restatement doesn't change the historical disclosure. |

Builds on: `commerce-media-networks` (`campaign_day`) and `commerce-media-networks` (`advertiser`, `campaign`).
Deferred: Lakebase serving projections (`api_supplier_campaign_summary`, `api_user_supplier_map`).

## Joint Demand Planning (CPFR)

Collaborative Planning, Forecasting and Replenishment (CPFR): a retailer and its suppliers jointly
build, submit, reconcile and freeze a **consensus demand forecast** under row-level access control,
with an immutable agreed snapshot for accountability. Each party submits its own versioned plan; the
reconciliation view aligns them, an exception queue surfaces disagreement and gaps, and the published
consensus is frozen so a later restatement never changes what was already agreed. Builds only on the
canonical core (`supplier`, `product`; fiscal weeks as a degenerate `fiscal_week_id`, no calendar FK); NULL-safe enriches with the Smarter Demand package's
statistical baseline (no hard dependency).

| Object | Grain | Description |
|---|---|---|
| `demand_collaboration` | one version per (principal, supplier, product-scope) | Row-level access + scope anchor: which governance principal collaborates on which supplier's plan (optionally one product), the agreed horizon, and the minimum disclosure threshold. SCD2. |
| `demand_plan_submission` | (plan cycle × party × supplier × product × week × version) | Each party's versioned demand-forecast submission (retailer plan and supplier plan land as separate rows; latest version wins). |
| `consensus_forecast_snapshot` | one publish snapshot × supplier × product × week × version | Immutable frozen consensus — the agreed numbers (and each side's number at agreement) at a point in time (audit/dispute). |
| `gold_joint_demand_plan` | principal × supplier × product × week | Entitled joint plan (row-filter source): latest retailer/supplier submissions + latest consensus, NULL-safe statistical baseline. No person-level data. |
| `gold_demand_plan_exceptions` | principal × supplier × product × week | The joint-plan rows needing collaborative attention, ranked by demand value at risk. |

| Term | Definition |
|---|---|
| **Collaborative forecast (CPFR)** | A demand forecast a retailer and supplier build together, submitting their own plans and agreeing a consensus, rather than each planning in isolation. |
| **Plan cycle** | A consensus round (e.g. a monthly CPFR cadence) that groups the submissions and the consensus that reconcile within it (`plan_cycle_id`). |
| **Demand plan submission** | One party's versioned forecast for a supplier × product × fiscal week; a resubmission bumps `submission_version`, and the latest wins in reconciliation. |
| **Consensus demand** | The agreed joint forecast quantity/value for a supplier × product × week, frozen as an immutable versioned snapshot (`consensus_forecast_snapshot`). |
| **Plan variance / disagreement** | `plan_variance_pct = (retailer_qty − supplier_qty) / supplier_qty` — how far the two parties' latest plans diverge. |
| **Consensus lift over baseline** | `consensus_vs_baseline_pct = (consensus_qty − statistical_forecast_qty) / statistical_forecast_qty` — the collaboration's demand signal above the statistical forecast (NULL if the Smarter Demand package is absent). |
| **Exception** | A plan row raised for attention: a missing party plan, a party disagreement beyond tolerance, or a consensus that diverges materially from the statistical baseline. |
| **Disclosure threshold** | Minimum demand volume (units) a plan row must reach before it is shared with a partner — a low-volume/materiality floor, **not** a k-anonymity control (the shared rows are already aggregate at supplier × product × week and carry no underlying entity count). A NULL threshold means no floor. |

Builds on: canonical core `supplier`, `product` (fiscal weeks as a degenerate `fiscal_week_id`, no calendar FK). NULL-safe enrichment:
`smarter-demand-and-inventory-decisions` (`gold_demand_forecast_weekly`) — read-only, no FK.
