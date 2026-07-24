# Unified Customer View — Business Glossary

> Status: 🟡 In progress (0.1-beta) · Last reviewed: 2026-07-10

Vendor-neutral; follows the ORDM [data model standards](../../docs/data-model-standards.md). Built entirely
on the thin customer core — no product/transaction/campaign data.

## Tables & views

| Object | Grain | Description |
|---|---|---|
| `customer_profile_snapshot` | One row per profile per snapshot_date | Append-only point-in-time snapshot of identity/consent/preference/household roll-ups. Extension table; no raw PII. |
| `gold_unified_customer_360_current` | One current row per customer | Conformed 360 summary keyed on the customer surrogate; identity, consent, preference, household roll-ups. No raw PII. |
| `gold_customer_activation_eligibility_current` | One row per customer × purpose × channel | Net-permission gate: `eligible_flag` = consent granted AND preference allows AND a valid identifier exists, with `blocking_reason_code`. |
| `gold_activatable_audience_current` | One row per audience × profile × purpose × channel × jurisdiction | The activation surface: behavioral-segment membership (ACU `segment_membership`, latest as-of) joined to the net eligibility gate, keyed on the durable `profile_id`. Fail-closed — a customer appears only where an eligible gate row exists and the segment is addressable. No raw PII. |
| `profile` / `identity_link` / `consent` / `channel_preference` / `household` / `contact` / `address` | — | *(canonical-core)* the customer core this package rolls up. |

## Key concepts

| Term | Definition |
|---|---|
| **360 summary vs. eligibility gate** | The 360 view's consent flags are a customer-level *summary* ("granted anywhere"). The authoritative per-purpose × channel permission — resolved as-of `jurisdiction_code` — is `gold_customer_activation_eligibility_current`. |
| **Activation eligibility** | `eligible_flag = consent_granted AND preference_allows AND has_identifier`. Consent (legal), preference (soft opt-out), and reachability (a valid identifier) are three distinct governed states and are never collapsed (principle #5). |
| **`blocking_reason_code`** | First failing rule when not eligible: `consent_denied` / `withdrawn` / `preference_opt_out` / `no_identifier`. `suppressed` is **reserved** for the deferred suppression ledger (not produced yet). |
| **`resolved_identity_confidence`** | Summary identity-resolution confidence in [0,1] over a customer's current `active` identity links. |
| **`reachability_score`** | Fraction of channels (email/sms/postal) a customer is reachable on = consent + preference + identifier present. |
| **`profile_completeness_score`** | Fraction of key attributes populated (name, dob, email, address). |
| **`jurisdiction_primary_code`** | The customer's governing privacy region (from primary address); consent precision lives in the eligibility view's `jurisdiction_code`. |

## Metrics (definitions)

These KPIs are captured **engine-neutrally** — definition, grain, and expression — so any consumer can
implement them (dbt metrics, Cube, a BI semantic layer, or Databricks UC Metric Views). A UC Metric View
overlay is a deferred, optional Databricks-native materialization of the same definitions.

| Metric | Definition | Grain / dimensions | Expression (over the gold views) |
|---|---|---|---|
| `active_customer_count` | Count of current unified customers. | customer · dims: status, jurisdiction, household | `COUNT(*)` over `gold_unified_customer_360_current` |
| `avg_identity_confidence` | Mean resolved-identity confidence. | customer | `AVG(resolved_identity_confidence)` |
| `identity_coverage_rate` | Share of customers with ≥1 active resolved identifier. | customer | `AVG(CASE WHEN active_identifier_count > 0 THEN 1.0 ELSE 0 END)` |
| `consent_marketing_rate` | Share of customers with marketing consent granted. | customer · dims: jurisdiction | `AVG(CASE WHEN consent_marketing_flag THEN 1.0 ELSE 0 END)` |
| `contactable_rate` | Share contactable per channel. | customer × channel | `AVG(<channel>_contactable_flag)` |
| `household_coverage` | Share of customers assigned to a household. | customer · dims: household_type | `AVG(CASE WHEN household_size IS NOT NULL THEN 1.0 ELSE 0 END)` |
| `avg_household_size` | Mean household size. | household | `AVG(household_size)` over distinct households |
| `avg_reachability_score` | Mean activation-readiness. | customer | `AVG(reachability_score)` |
| `avg_profile_completeness` | Mean profile completeness. | customer | `AVG(profile_completeness_score)` |
| `consent_granted_rate_by_purpose` | Share of purpose×channel rows with consent granted. | customer × purpose × channel · dims: jurisdiction | `AVG(CASE WHEN consent_granted_flag THEN 1.0 ELSE 0 END)` over the eligibility view |
| `eligible_rate` | Share of purpose×channel rows that are activation-eligible. | customer × purpose × channel · dims: purpose, channel, jurisdiction | `AVG(CASE WHEN eligible_flag THEN 1.0 ELSE 0 END)` |
| `suppression_rate` | *(reserved)* Share blocked by suppression. | customer × purpose × channel | pending the deferred suppression ledger |

## Standards used

| Concept | Standard |
|---|---|
| Dates / timestamps | ISO 8601 (UTC) |
| Jurisdiction / country | ISO 3166 (`jurisdiction_code`, `jurisdiction_primary_code`) |
| Privacy scoping | GDPR / CCPA-CPRA (consent resolved as-of jurisdiction) |

## Deferred (documented, not built)

UC Metric Views (optional Databricks overlay of the metrics above); campaign machinery; the suppression
ledger; and any 360 field needing transaction/product/store data (LTV, purchase recency, affinity,
preferred store). (Segment/audience membership and the activation surface are now BUILT: `audience` is a
silver master, ACU ships `segment_membership`, and `gold_activatable_audience_current` above is the
activation join.) See [`design/customer-identity/`](../../design/customer-identity/).
