# AI-Assisted Store Associates — Business Glossary

> Status: 🟡 In progress (0.1-beta) · Last reviewed: 2026-08-29

Vendor-neutral; follows the ORDM [data model standards](../../docs/data-model-standards.md). This package
is the **Next Best Action (NBA)** engine for the store floor: it normalizes disparate real-time triggers into
a single prioritized task queue routed to the right associate, then closes the loop by capturing the
response and the real-world outcome. It **builds on** the canonical core rather than redefining it — customer,
product, and store context are foreign keys into `customer.profile` / `product.product` / `store.store`, and
the trigger signals map to existing entities (see *Builds on* in the README).

## Tables & views

| Object | Grain | Description |
|---|---|---|
| `associate` | One row per associate per SCD2 version | Store-workforce dimension: role, skills, home store, employment type. Master-tagged extension; no raw employee PII. Live floor status/zone is out of scope (operational state). |
| `suggested_action` | One row per suggested action | The normalized, prioritized NBA queue routed to associates. Adopter-owned current-state table (schema + DQ contract); lifecycle transitions are logged in `action_response_log`. |
| `action_response_log` | One row per associate response event | Append-only log of how the associate acted on a prompt (accept/reject/defer/complete). |
| `outcome_resolution` | One row per resolved action | Append-only real-world outcome (sale closed, item restocked, …) — the ROI signal and labeled training history. |
| `gold_associate_action_queue_current` | One row per open, non-expired action | The live prioritized queue an associate sees, ranked within associate by `priority_score`. |
| `gold_action_outcome_funnel_daily` | One row per store × trading date × action_type | Generated / accepted / rejected / completed / expired counts, acceptance & completion rates, avg response time, realized outcome value. |
| `mv_associate_action_performance` | store × trading date × action_type | UC Metric View over the daily funnel: acceptance, completion, responsiveness, realized value (ratio-of-sums). |
| `profile` / `product` / `store` | — | *(canonical-core)* the customer / product / store context the actions reference. |

## Key concepts

| Term | Definition |
|---|---|
| **Next Best Action (NBA)** | The single highest-value task an associate should do right now, chosen by normalizing every trigger into one queue and ranking by a unified score. |
| **Business Value Score** (`priority_score`) | A dynamic 0–100 value the associate UI sorts on: `base_score` × dynamic multipliers, then time-decayed. Lets fundamentally different tasks (help a VIP vs. restock a shelf) be ranked against each other. |
| **Base score (hierarchy of needs)** | The priority in a vacuum before context: safety_loss_prevention 90, bopis_fulfill 75 (SLA-driven), customer_assist 60, restock_shelf / merchandising 40, maintenance_cleaning 20. |
| **Dynamic multipliers** | Contextual scaling applied to the base: customer VIP ×1.5 (unknown ×1.0); high-margin product ×1.4 (low-margin ×0.8); completely empty shelf ×2.0 (low stock ×1.1). *Example:* an unknown shopper dwelling at high-margin OLED TVs (60 × 1.4 = **84**) outranks an empty shelf of HDMI cables (40 × 0.9 × 2.0 = **72**). |
| **`decay_type` — linear** | Persistent-state tasks (restocking, cleaning): the physical state won't self-resolve, so priority erodes slowly. `priority_score = base_score − (elapsed_minutes × 0.5)`, to a floor. |
| **`decay_type` — exponential** | Transient tasks (customer assistance, queue-busting): the opportunity window closes fast. `priority_score = base_score × (0.8 ^ elapsed_minutes)` — a VIP assist at 85 falls below routine restocking within ~5 minutes. |
| **`decay_type` — immediate_nullification** | A secondary event makes the task instantly irrelevant: the customer leaves the zone, another associate scans the out-of-stock item, or the store closes. `priority_score → 0` and `status → 'expired'`. |
| **`context_payload`** | Schemaless JSON the associate app renders, so a new task type needs no schema migration. Common shapes below. |
| **Acceptance rate / completion rate** | `actions_accepted / actions_generated` and `actions_completed / actions_generated`. A run of `rejected | too_busy` in one zone is a staffing signal, not just a data point. |

### `context_payload` common shapes

Customer assistance:
```json
{"action_type":"CUSTOMER_ASSIST","trigger_source":"BLE_BEACON_DWELL",
 "location":{"zone_name":"High-Value Electronics","aisle":"4B"},
 "customer_context":{"is_known":true,"loyalty_tier":"Platinum","recent_purchase":"PlayStation 5","app_cart_active":["HDMI 2.1 Cable"]},
 "suggested_dialogue":"Hi, finding everything okay? I see you might be looking for accessories for your PS5."}
```
Restocking:
```json
{"action_type":"RESTOCK_SHELF","trigger_source":"POS_INVENTORY_DECREMENT",
 "location":{"zone_name":"Home Goods","planogram_id":"P-104-A"},
 "product_context":{"sku":"1009842","margin_tier":"High","current_floor_qty":0,"backroom_location":"Bin 42-C","replenish_qty":3}}
```
BOPIS expedite:
```json
{"action_type":"BOPIS_FULFILL","trigger_source":"ORDER_MANAGEMENT_SYSTEM",
 "location":{"zone_name":"Fulfillment Desk"},
 "order_context":{"order_id":"ORD-99381","item_count":4,"sla_deadline":"2026-08-29T19:30:00Z","minutes_remaining":67,"urgency_flag":"ELEVATED"}}
```

## Metrics (definitions)

Captured engine-neutrally (definition, grain, expression) and materialized as the UC Metric View
`mv_associate_action_performance`. Rates are **ratio-of-sums** so they re-aggregate at any grain.

| Metric | Definition | Grain / dimensions | Expression (over the gold funnel) |
|---|---|---|---|
| `actions_generated` | Count of NBAs dispatched. | store × date × action_type | `SUM(actions_generated)` |
| `acceptance_rate` | Share of generated actions accepted. | store × date × action_type | `SUM(actions_accepted) / SUM(actions_generated)` |
| `completion_rate` | Share of generated actions completed. | store × date × action_type | `SUM(actions_completed) / SUM(actions_generated)` |
| `avg_response_seconds` | Mean dispatch-to-first-response time. | store × date × action_type | `SUM(response_seconds_total) / SUM(responses_timed)` |
| `sales_closed` | Actions resolved as a closed sale. | store × date × action_type | `SUM(sales_closed)` |
| `realized_outcome_value` | Monetary value attributed to actions. | store × date × action_type | `SUM(realized_outcome_value)` (single reporting currency) |

## Standards used

| Concept | Standard |
|---|---|
| Dates / timestamps | ISO 8601 (UTC) |
| Currency | ISO 4217 (`currency_code`, `outcome_currency_code`) |
| Money typing | DECIMAL(18,2) for `outcome_value_amount` (ORDM §A.6) |

## Deferred (documented, not built)

A normalized `associate_skill` association (skills are a delimited STRING today); a live associate
availability/roster fact (floor status/zone is operational state the app owns); an ML expected-value model
trained on `outcome_resolution` (this package ships the labeled history, not the model); and a UC Governed
Tag overlay for the workforce dimension. Trigger ingestion (sensor / POS / digital) is out of scope — those
signals live in connected-store-signals, `transaction.sales`, and `interaction.touchpoint`.
