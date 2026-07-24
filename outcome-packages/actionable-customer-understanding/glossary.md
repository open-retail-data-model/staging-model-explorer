# Actionable Customer Understanding — Business Glossary

> Status: 🔵 Nearing complete (0.1-beta) · Last reviewed: 2026-07-21

Business terms for the Customer Lifetime Value and Behavioral Segmentation use cases. Vendor-neutral; follows the ORDM [data model principles](../../docs/data-model-standards.md).

## Tables & view

| Object | Grain | Description |
|---|---|---|
| `customer_order_line` (canonical-core) | One customer × order × product line | Customer-attributed purchases (the base for per-customer value + behavioral signals). |
| `gold_customer_ltv` | One customer (by surrogate) | Historical + predicted CLV, RFM segmentation, value tiers. Keys on the surrogate; **no raw PII**. |
| `gold_customer_behavioral_attributes` | One customer (by durable `profile_id`) | Wide, agent-discoverable behavioral trait surface (cadence, affinity, lifecycle stage, promo responsiveness) + value/RFM attributes surfaced from `gold_customer_ltv`. No raw PII. |
| `segment_membership` | One `profile_id` × `audience_id` × `as_of_date` (append-only) | Which customers belong to which segment, as of an evaluation date. Targeting plane only. |
| `marketing.audience` (canonical-core) | One segment definition (SCD2) | The shared segment/audience **definition** catalog membership references (ADR 0022). |

## Terms

| Term | Definition |
|---|---|
| **CLV — historical** (`historical_clv`) | Total **realized gross margin** to date: `SUM(net_amount − units × unit_cost)`. Can be negative (a real loss). |
| **CLV — predicted** (`predicted_clv`) | A transparent forward estimate, **NOT an ML model**: `avg_order_margin × frequency_per_period × expected_active_periods`, clamped to ≥ 0. `expected_active_periods = GREATEST(0, LEAST(tenure_periods, HORIZON_CAP=12) − recency_periods)` — loyalty (tenure) extends the horizon, recency shrinks it. A **heuristic starter metric**; if/when an MLflow model exists it can replace this (follow-up, not pulled into the view). |
| **Recency** (`recency_periods`) | Fiscal periods (NRF 4-5-4) since the customer's last order. |
| **Frequency** (`frequency`) | Distinct purchase occasions (`order_id`) in the observation window. |
| **Monetary** (`total_spend`, `avg_order_value`) | Total net spend and spend per order. |
| **RFM** (`r_score`, `f_score`, `m_score`, `rfm_score`) | Recency, Frequency, Monetary each scored 1–5 from **stable fixed thresholds** (not population `NTILE`), so a customer's score is **reproducible across refreshes** and changes only when their own behavior crosses a cutoff. **5 = best** (most recent / frequent / highest spend). `rfm_score = 100·R + 10·F + M` (classic 111–555 cell). Raw recency/frequency/spend stay exposed so a consumer can rebucket. |
| **Value tier** (`value_tier`) | `historical_clv` bucketed on **stable fixed thresholds** (PLATINUM ≥ 2500 / GOLD ≥ 1000 / SILVER ≥ 250 / BRONZE else), not an `NTILE(4)` quartile — reproducible across refreshes for a frozen campaign audience. Thresholds (and the monetary M cutoffs) are in the deploy's **base/reporting currency**, whole units — illustrative mid-market retail defaults adopters retune to their own currency/basket size. |
| **Churn-risk proxy** (`churn_risk_proxy`) | Heuristic flag: recency > 2.0 × the customer's own average cadence (`tenure_periods / frequency`). Clearly a proxy, not a model. |

### Behavioral Segmentation

| Term | Definition |
|---|---|
| **Behavioral segmentation** | Grouping customers by **observed actions** (purchase cadence, category/brand affinity, promo responsiveness, lifecycle stage) — distinct from value/RFM (worth) and demographic (identity). Reuses the RFM/value scores as attributes rather than recomputing them. |
| **Trait surface** (`gold_customer_behavioral_attributes`) | The wide, one-row-per-customer, agent-discoverable set of behavioral + value traits a CDP composes custom audiences from. Keyed on the durable `profile_id`; no raw PII. |
| **Lifecycle stage** (`lifecycle_stage`) | Behaviorally-derived relationship phase: `new` / `active` / `lapsing` / `churned`, from recency vs the customer's own purchase cadence. |
| **Category / brand affinity** (`top_category`, `top_brand`, `distinct_categories`, `distinct_brands`) | The category/brand a customer spends the most net revenue in, plus the breadth of categories/brands purchased. |
| **Promo purchase share** (`promo_purchase_share_ratio`) | Share of net spend on discounted lines (net < gross) — a promo-responsiveness signal, 0–1. |
| **Opt-in status** (`opt_in_status`) | Marketing consent surfaced as a **discoverable attribute** (`opted_in` / `opted_out` / `unknown`). NOT the legal source of truth (that is canonical-core `consent`) and never the activation gate. |
| **Segment definition** (`marketing.audience`) | The named, versioned rule/criteria for a segment (shared silver dimension, ADR 0022). Definition only — no member lists, no PII. |
| **Segment membership** (`segment_membership`) | Which customers belong to which segment, as of an evaluation date (append-only snapshots). `membership_status`: realized / existing / exited. Targeting plane only — carries no consent flags. |
| **Activatable audience** (`gold_activatable_audience_current`, in UCV) | Segment membership intersected with the consent+preference+reachability gate — the reverse-ETL-ready surface of who may actually be contacted per purpose × channel. The one place the three planes combine. |

## PII handling

The view keys on the customer **surrogate** (`profile_sk`) plus the pseudonymous business key `profile_id`. Raw PII — `loyalty_id`, `household_id`, names, date of birth — is **never** carried here; it stays in the governed `profile` dimension (tagged `dbx_pii_*`). A DQ check (`ltv_no_raw_pii`) and a test assert PII absence on the gold schema.
