# Predictive Operations — Business Glossary

> Status: 🟡 In progress (0.1-beta) · Last reviewed: 2026-07-27

Business terms for the Predictive Operations outcome package (In-Store Equipment Maintenance and Store Performance use cases). Vendor-neutral; follows the ORDM [data model standards](../../docs/data-model-standards.md).

## Tables & views

| Object | Grain | Description |
|---|---|---|
| `equipment` (canonical-core) | One version per equipment asset (SCD2) | In-store equipment master: refrigeration, HVAC, POS, ovens. |
| `equipment_reading` (canonical-core) | One sensor reading (append-only) | Telemetry data: temperature, power, vibration, error codes. |
| `work_order` | One work order | Maintenance task header with lifecycle: open to completed. |
| `work_order_line` | One line per work order | Parts, labor, and service charges for a work order. |
| `maintenance_schedule` | One version per schedule entry (SCD2) | Recurring preventive maintenance plan for an equipment asset. Versioned so schedule changes (frequency, cost, active/inactive) are historically auditable. |
| `gold_equipment_health` | One equipment asset (current) | Health dashboard: latest readings, anomaly stats, health score. |
| `gold_maintenance_effectiveness` | One equipment asset (all time) | Maintenance program effectiveness: PM ratio, MTBF, cost, compliance. |
| `gold_downtime_impact` | One downtime event | Downtime impact analysis: links outages to inventory/spoilage risk. |
| `gold_store_operational_risk` | One store (current as_of_date) | Store Operational Risk Score: composite of four domain sub-scores + diagnosis. |
| `store_risk_score` | One store × as_of_date × score_method (append-only) | Durable snapshot history of the risk score a scoring pipeline writes. |

## Key terms — Store Performance

| Term | Definition |
|---|---|
| **Store Operational Risk Score** | A store-grain index (0-100, higher = riskier) estimating near-term operational/financial deterioration over a declared forward horizon. A transparent, expert-weighted **heuristic index, not a calibrated probability** — ORDM ships no trained model. Composed from four domain sub-scores, blended by visible weights, with non-compensatory escalators. |
| **Domain sub-score** | A 0-100 risk score for one signal domain (Inventory, Supply-Chain, Sales/Margin, Equipment; Labor and CX deferred). Each raw signal is assigned to exactly one domain (its proximate cause) so the availability→conversion pathway is not double-counted. A missing sub-score is `NULL` and excluded from the composite denominator (missing = unknown, not safe). Note: `sales_margin_risk`'s discount-depth half is normalized against a **cross-store** p95, so it is a *relative* (fleet-comparative) signal, not purely store-local — see design spec §4. |
| **Non-compensatory scoring** | The composite is not a plain weighted average: a worst-domain floor (any present sub-score ≥ 75 forces ≥ HIGH) and a steep-sales-decline force prevent a strong domain from averaging away a critical one — the blind spot a "102%-of-target but sliding" store exposes. |
| **Risk tier** | Categorical triage: `UNKNOWN`, `LOW`, `MEDIUM`, `HIGH`, `CRITICAL`. `UNKNOWN` means no domain sub-score was present (`domains_scored = 0`) so the store is unscored, not low-risk (missing = unknown, not safe); the rest come from thresholds on the score plus the two escalators. |
| **Top risk domains** | The up-to-3 domains contributing most to the score (the Diagnose step) — the explainability that makes a risk score trustworthy. |
| **Horizon** | The declared forward assessment horizon (`horizon_days`, 7 in v1) the score is interpreted against — uniform across a snapshot so scores stay comparable. It is a label, not a claim that every input is windowed to 7 days: sub-score observation windows differ by domain (current-state for inventory/supply-chain/equipment, trailing 14-day-vs-prior for sales/margin — see design spec §4.7). Distinct horizons would be separate columns, never mixed in one. |
| **Score method** | `heuristic` (the shipped deterministic index) or `model` (rows a fitted, model-backed pipeline writes later). Distinguishes what the data model produces from what an adopter's model adds. |

## Key terms — In-Store Equipment Maintenance

| Term | Definition |
|---|---|
| **Work order** | A maintenance task for a specific equipment asset. Types: `preventive` (scheduled), `corrective` (reactive fix), `emergency` (urgent), `inspection` (assessment). |
| **Work order line** | An individual cost item within a work order: a part used, a labor entry, or a service charge. |
| **Maintenance schedule** | A recurring preventive maintenance plan specifying what task to perform, how often, and on which equipment. |
| **Health score** | 0-100 score measuring equipment health. Combines anomaly frequency, error counts, and corrective work order history. 100 = fully healthy, 0 = out of service. Note: `latest_temperature` and other "latest reading" columns have no recency bound — check `last_reading_timestamp` to distinguish a genuinely quiet asset from one that has stopped reporting. |
| **Health tier** | Categorical assessment: `healthy`, `at_risk` (elevated anomalies or corrective work), `degraded` (equipment status is degraded), `failed` (out of service). |
| **MTBF** | Mean Time Between Failures — average calendar days between completed corrective/emergency work orders (by `created_date`) for an equipment asset. Only completed work orders are included; open/in-progress failures do not enter the interval series. Higher = more reliable. |
| **Preventive-to-corrective ratio** | Ratio of preventive to corrective work orders. Higher values indicate a more proactive maintenance program. Industry target is typically > 3:1. |
| **Schedule compliance** | Fraction of active maintenance schedules that are on-schedule (last completed within the defined interval). |
| **Downtime** | Hours an equipment asset was non-operational due to a maintenance event. |
| **Spoilage risk** | Risk of perishable product loss when refrigeration equipment fails. Inventory exposure is scoped to perishable product categories and aligned to the latest inventory snapshot on or before the event date. Tiered by downtime duration: `high` (>4h), `medium` (2-4h), `low` (<2h). The exposure columns are per-event store-level snapshots (non-additive across work orders — use MAX per store/date, not SUM). Adopters should refine the perishable filter to match their product taxonomy. |
| **Failure code** | Standardized reason code for why a work order was created (e.g. compressor_failure, sensor_malfunction, wear_and_tear). |

## Builds on (canonical core)

- `equipment` (canonical-core/equipment) — in-store equipment master (SCD2)
- `equipment_reading` (canonical-core/equipment) — sensor telemetry (append-only)
- `store` (canonical-core/store) — conformed store dimension
- `product` (canonical-core/product) — conformed product dimension (perishable category filter)
- `inventory_position` (canonical-core/inventory) — daily inventory for spoilage risk assessment
- `purchase_order_line` (canonical-core/procurement) — PO-line ship-to location, maps supplier risk to store grain (Store Performance)
- `fiscal_calendar` (canonical-core/calendar) — the `as_of_date` grain conforms to `date_key` (Store Performance)
