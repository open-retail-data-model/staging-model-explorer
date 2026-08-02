# Connected Store Signals — Business Glossary

> Status: 🟡 In progress (0.1-beta) · Last reviewed: 2026-07-28

Business terms for the Connected Store Signals outcome package. Vendor-neutral; follows the
ORDM [data model standards](../../docs/data-model-standards.md).

Two things about this domain that the term list alone will not convey. First, **footfall
vendors do not agree on what footfall means** — the same field name carries at least four
meanings depending on device configuration, and there is no standard to appeal to (unlike
RFID, which has GS1 EPCIS, or building telemetry, which has Brick and BACnet). Every
convention column below exists because of that absence. Second, **no measure here is safe to
sum without checking `aggregation_level`**.

## Tables & views

| Object | Grain | Description |
|---|---|---|
| `store` (canonical-core) | One version per store (SCD2) | Conformed store master. |
| `fiscal_calendar` (canonical-core) | One row per date | NRF 4-5-4 fiscal calendar. Joined logically on `trading_date = date_key`, not by FK. |
| `equipment` (canonical-core) | One version per asset (SCD2) | The maintained asset a temperature probe monitors. |
| `store_zone` | One version per zone (SCD2) | Recursive spatial hierarchy: site → area → aisle → bay → shelf position. Carries area, capacity and countability. |
| `sensing_deployment` | One version per deployment (SCD2) | The governance record for a sensing installation: method, purpose, lawful basis, DPIA, signage, retention, staff-exclusion policy. |
| `sensing_disclosure_policy` | One effective version per disclosure policy | Audience-specific k-threshold, zero suppression, same-cell dependent-measure suppression, and minimum publishable window. |
| `sensor_device` | One version per device (SCD2) | Physical device registry, anchored on Brick Schema 1.4.4 + QUDT, with BACnet / OPC UA / Sparkplug identity and calibration metrology. |
| `sensor_report_interval` | One report type × subject namespace × canonical interval × level × device | Narrow EPCIS-shaped interval envelope. Every modeled series is non-overlapping; alternate vendor rollups stay outside the additive fact. |
| `foot_traffic_interval_detail` | One row per `gs1:Count` envelope | One-to-one traffic, counting-convention, staff, occupancy, and dwell extension. |
| `gold_store_traffic_daily_current` | One store × trading date | Store-level daily traffic, dwell and occupancy with NRF prior-year comparison. |
| `gold_zone_traffic_current` | One store × zone × trading date | Zone-level traffic with density and engagement. |
| `gold_store_traffic_daily_shared` | One store × trading date | Disclosure-controlled shared store traffic. |
| `gold_zone_traffic_shared` | One store × zone × trading date | Disclosure-controlled shared zone traffic. |
| `gold_traffic_sales_productivity_current` | One store × trading date | Sales and units per entry and per footfall. It exposes no conversion surface. |
| `gold_css_device_health` | One device version × report type × aggregation level × trading date | Whole-estate fleet health: uptime, coverage, freshness, session integrity, calibration, governance gaps. Report type and level are in the grain because one device can report more than one thing at more than one level. |
| `mv_store_foot_traffic` | store × date | Store totals. Separate from the zone view because a metric view cannot require a filter, so one combined view let `SELECT footfall` return the store total plus every zone total. |
| `mv_zone_foot_traffic` | store × zone × date | Zone detail, for comparing and ranking zones and for density. Zone footfall does NOT sum to a store total. |
| `mv_traffic_sales_productivity` | store × date | Metric view over traffic-sales productivity. |
| `mv_device_health` | store × device type × date | Metric view over the fleet-health gold view. |

## Telemetry integrity terms

| Term | Meaning |
|---|---|
| Sequence gap | Source messages inferred missing within a source session. Raw sequence values stay in Bronze; the interval envelope carries only the interval-level count. |
| Duplicate delivery | Source messages delivered more than once. A different fault from a gap and counted separately as an interval summary. |
| Session restart | More than one `source_session_id` in a device-day. |

## Counting terms

| Term | Definition |
|---|---|
| **Footfall** (`footfall_count`) | Shopper count for the interval, measured **at** `aggregation_level`. Uninterpretable without the four convention columns below. |
| **Entries / exits** (`entry_count`, `exit_count`) | Independently measured crossings. They are **never exactly equal** — a divergence of a percent or so is physics, not a fault. `exit_count` is NULL, never 0, for a non-directional sensor. |
| **Counting unit** (`counting_unit`) | What one count represents: `person`, `group`, `device` or `object`. With group counting enabled, a count is of visitor **groups**. |
| **Group counting** (`group_counting_enabled`) | Whether a family or shopping party is counted as one. Changes `counting_unit` from person to group. |
| **Child counting** (`child_counting_enabled`, `child_height_threshold_cm`) | Whether shoppers below a height threshold are counted. Children are a **subset** of footfall, not an exclusion. The threshold is vendor-specific (130 cm and 120 cm are both in the field), so it travels with the count. |
| **Pass-by** (`passersby_count`, `passby_convention`) | Storefront traffic that did not enter. Vendors define this **two opposite ways** — one includes entrants, one excludes them — so the bare number is ambiguous by a factor of the capture rate. |
| **Capture rate** | `footfall / passersby`. Only meaningful where the pass-by convention is uniform across the rows compared. |
| **Count divisor** (`count_divisor`) | Divisor applied by a beam sensor counting both directions on one door. A device setting, **not** the constant 2. |
| **Unique visitors** (`unique_visitor_count`) | De-duplicated visitors within a window. Not additive across intervals. |

## Staff exclusion terms

| Term | Definition |
|---|---|
| **Staff exclusion method** (`staff_exclusion_method`) | How staff are separated: exclusion line, zone button, patterned or fabric tag, uniform ML, re-identification, fixed subtraction, scaling factor, or none. |
| **Staff exclusion stage** (`staff_exclusion_stage`) | **Where** the exclusion was applied: `at_sensor`, `at_aggregation` or `none`. Without this you cannot tell whether a received number was already corrected, so **double-correction is inevitable**. |
| **Staff exclusion grade** (`staff_exclusion_grade`) | Published accuracy band for the method: high (>90%), medium (70–90%), low (<70%). |
| **Raw vs adjusted** (`visitor_count_raw`, `visitor_count_adjusted`) | Before and after staff adjustment. The identity `adjusted = raw - staff_count` is enforced at error severity. |

## Occupancy terms

| Term | Definition |
|---|---|
| **Occupancy** (`occupancy_snapshot_end`) | People present at interval end. **Semi-additive**: take the last value across intervals, never the sum. |
| **Peak occupancy** (`occupancy_max`) | Highest simultaneous occupancy in the interval. A **store** peak comes from the site row, not `MAX()` over zone rows. |
| **Average occupancy** (`occupancy_avg`) | Mean occupancy, rolled up **duration-weighted** — never an average of averages, which over-weights sparse periods. |
| **Occupancy adjustment** (`occupancy_adjustment`) | A **signed** manual correction. Two unrelated standards independently put this in the schema: a retail vendor exposes it as a queryable metric, and BACnet Access Zone carries `Adjust_Value` as an integer. |
| **Counter reset** (`last_reset_local_ts`) | When cumulative counters were last zeroed. Per ONVIF this is interpreted as **local** time and is unbounded — multiple resets per day are legal, so a model assuming one daily reset is wrong. |
| **Negative occupancy** | Entry/exit drift. It is surfaced as a warning, never clamped: clamping would destroy the only evidence the balance has broken. |

## Dwell terms

| Term | Definition |
|---|---|
| **Dwell total / count** (`dwell_seconds_total`, `dwell_count`) | Numerator and denominator, stored separately so the average re-aggregates at any grain. A pre-computed average cannot. |
| **Average dwell** | `dwell_seconds_total / dwell_count`. |
| **Engaged vs passing** (`engaged_count`, `passing_count`, `engagement_threshold_seconds`) | Dwell observations above and below a configurable threshold. `passing_count` is a dwell bucket and is **not** the same as `passersby_count`. |
| **Engagement rate** | `engaged_count / dwell_count`. |

## Quality and completeness terms

| Term | Definition |
|---|---|
| **Measurement validity** (`measurement_validity`) | Was the sensor telling the truth: valid, degraded, invalid, imputed. |
| **Arrival completeness** (`arrival_completeness`) | Has the data landed: complete, incomplete, late_arrived, precheck. |
| **Why they are separate** | They are **orthogonal**. A sensor with its batteries pulled for 15 minutes yields an interval that is invalid but complete; a still-open current interval is valid but incomplete. One quality score cannot express both. |
| **Observed zero** (`is_zero_observed`) | A **measured** zero, distinct from an absent row. Most vendors emit no row for either a zero-traffic interval or an offline sensor, which makes the two indistinguishable without this flag. |
| **Uptime / coverage** (`uptime_ratio`, `coverage_ratio`) | Share of the interval the device was online, and share of expected sensors that reported. A low coverage means the count is **partial**, not low. |
| **Capped** (`is_capped`, `capped_count_limit`) | The reading hit the hardware detection ceiling. A capped peak is not an exact peak. |
| **Restatement** (`restatement_version`) | These feeds are **not** append-only. Vendors publish a late-data status that later resolves, and sensors check in with data hours or days old, so a row can be rewritten. |
| **Source observations** (`source_observation_count`, `expected_observation_count`) | Received and expected observations derived from source cadence and delivery completeness, never from shopper count or measured value. |
| **Source message summary** | Interval-level message, gap, duplicate, historical, and bad-quality counts. Per-message sequence and quality fields remain in Bronze. |
| **Session identity** (`source_session_id`) | Source-session identifier effective for the interval. |

## Additivity rules

| Measure | Rule |
|---|---|
| `footfall_count`, `entry_count`, `exit_count` | Additive over **time** within one `aggregation_level`. **Not** additive across zones. |
| `occupancy_snapshot_end` | Semi-additive — last value, never a sum. |
| `occupancy_max` | Max over time at one level. A store peak comes from the site row. |
| `occupancy_avg` | Duration-weighted mean. |
| `avg_dwell_seconds` | Re-derive from `SUM(total) / SUM(count)`. |
| `unique_visitor_count` | Not additive at all. |
| Ratios (`uptime_ratio`, `coverage_ratio`, `confidence_score`) | Averaged, never summed. |

## Privacy and governance terms

| Term | Definition |
|---|---|
| **Aggregate-only schema** | There is no person-level tier in the published model, and the never-publish column list is asserted absent by a schema-level check. |
| **`external_default` k=10 policy** | Reference configuration for explicitly external/shared views, not a universal legal threshold or a base-fact invariant. Operational rows remain complete. |
| **Zero suppression** | A measured zero is suppressed on the same terms as a small count. |
| **Dependent-measure suppression** | Every traffic-derived measure in the same cell is NULL when its primary shared count is suppressed. This is not statistical complementary/secondary-cell suppression. |
| **Disclosure outcome** (`disclosure_suppressed_flag`) | Present only on shared views together with the policy id. Operational facts carry no suppression state. |
| **Sensing deployment** | The first-class governance record for an installation: method, declared purpose, lawful basis, jurisdiction, DPIA (including whether it assessed **employees**), signage, works-council consultation, consent requirement, opt-out mechanism, and retention ceiling. |
| **Sensing is never upstream of pricing** | An architectural invariant, not a usage guideline: no sensing table or view may appear in the lineage of a price-producing object. Enforced by a lineage-walking check, because the offending join is one line of SQL. |

## Traffic-sales productivity terms

| Term | Definition |
|---|---|
| **Conversion rate** | **Not published.** ORDM carries no transaction, basket or receipt count at any grain, so the industry numerator (transactions / visitors) does not exist, and no vendor publishes an accepted proxy. Computing it from `units` would count items rather than baskets and overstate conversion by roughly the average basket size. Recorded as decision D3; closing it needs a canonical-core change. |
| **Sales per entry** | `net_revenue / entry_count`. A productivity measure, not conversion. |
| **Units per entry** | `units / entry_count`. Items per entry, not transactions per visitor. |
| **Sales per footfall** | `net_revenue / footfall_count`. Preserves productivity for non-directional counters; not conversion. |
| **Units per footfall** | `units / footfall_count`. Items per observed footfall, not transactions per visitor. |
