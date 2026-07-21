# In-Store Equipment Maintenance — Business Glossary

> Status: 🟡 In progress (v1_mvm) · Last reviewed: 2026-07-15

Business terms for the In-Store Equipment Maintenance use case of the Predictive Operations outcome package. Vendor-neutral; follows the ORDM [data model standards](../../docs/data-model-standards.md).

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

## Key terms

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
