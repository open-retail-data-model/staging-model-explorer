# Smarter Demand and Inventory Decisions — Business Glossary

> Status: 🟡 In progress (0.1-beta) · Last reviewed: 2026-07-21

## Tables & views

| Object | Grain | Description |
|---|---|---|
| `planning_unit` | planning unit | Atomic demand-planning entity at configurable grain |
| `planning_unit_segment` | planning unit × fiscal period | ABC/XYZ segmentation and cluster assignment |
| `forecast_model_run` | model run | Model training and scoring execution registry |
| `demand_forecast` | planning unit × period | Statistical demand forecast lines |
| `forecast_component` | forecast × driver | Demand driver decomposition |
| `forecast_adjustment` | planning unit × period | Planner overrides for FVA |
| `order_forecast` | planning unit × order date | Replenishment / purchase / transfer order forecast |
| `forecast_accuracy` | planning unit × period | Lag-based accuracy and bias |
| `service_level_target` | segment version | Target fill rate or days of supply by ABC/XYZ |
| `inventory_policy` | planning unit version | Safety stock, ROP, min/max replenishment policy |
| `inventory_policy_calculation` | policy calculation | Audit of policy derivation inputs |
| `inventory_recommendation` | recommendation | System-generated replenishment recommendation |
| `oos_event` | product × store episode | Out-of-stock episode with cause and severity |
| `lost_sales_estimate` | OOS event × day | Estimated lost demand and revenue |
| `demand_sensing_signal` | signal | Short-horizon demand anomaly |
| `oos_recovery_action` | OOS event × action | Recovery action recommendation or execution |
| `gold_effective_forecast` | planning unit × period | Statistical forecast enriched with planner adjustments |
| `gold_inventory_health` | product × store | Inventory health with DoS and policy compliance |
| `gold_oos_current` | product × store | Active OOS episodes with lost sales to date |

## Demand planning terms

| Term | Definition |
|---|---|
| Planning unit | The atomic entity for forecasting; may be store×SKU or a rolled-up IBP grain |
| FVA (Forecast Value Added) | Improvement of the statistical forecast over a naïve baseline after planner enrichment |
| Forecast bias | Systematic over- or under-forecasting; positive bias implies excess inventory |
| DNA of demand | Decomposition of forecast into base, promo, price, weather, holiday, and residual drivers |

## Inventory terms

| Term | Definition |
|---|---|
| Days of supply (DoS) | On-hand units divided by average daily sales |
| Safety stock | Buffer inventory to absorb demand and lead-time variability |
| Reorder point (ROP) | Inventory level that triggers a replenishment order |
| ABC/XYZ | Volume tier (ABC) × demand variability (XYZ) segmentation matrix |

## Out-of-stocks terms

| Term | Definition |
|---|---|
| OOS rate | Percentage of product×store×day records with a stockout |
| Lost sales | Demand not captured in POS sales due to stockout |
| Demand sensing | Short-horizon anomaly detection bridging the forecast lag blind spot |
| Preventable OOS | Stockout that could have been avoided given policy and supply context |
