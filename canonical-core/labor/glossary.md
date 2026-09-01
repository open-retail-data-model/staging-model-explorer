# Labor domain glossary

| Term | Definition |
|---|---|
| **Employee** | Pseudonymous in-store associate master. Identified by durable `employee_id` only — no personal names or contact PII in ORDM. |
| **Primary role** | The associate's default role for scheduling (`cashier`, `sales_associate`, `fulfillment`, `inventory`, `supervisor`). |
| **Skill code** | A qualification that authorizes assignment to a role or specialized task (e.g. `food_handler`). |
| **Availability window** | A dated start/end interval when an associate can (or cannot) be scheduled. Timestamps are UTC. |
| **Store eligibility** | Optional multi-store assignment rights; absent rows mean home-store-only. |
| **Labor schedule** | A planned shift: employee × store × role × start/end. |
| **Labor actuals** | Worked / timecard intervals, optionally linked to a planned shift. |
| **Hourly labor cost** | Fully-loaded unit-grain hourly cost in base currency (`DECIMAL(18,4)`). |
| **Minimum rest hours** | Required gap between consecutive shifts for the same associate. |
| **Max daily / weekly hours** | Hard caps used by schedule optimization; jurisdiction rules must be configured by the adopter. |
