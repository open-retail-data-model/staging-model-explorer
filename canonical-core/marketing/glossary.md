# Marketing Domain — Business Glossary

> Status: 🟡 In progress (v1_mvm)

| Term | Definition |
|---|---|
| **Promotion** (`promotion`) | The conformed trade-promotion dimension (SCD2) — mechanics, funding, planned lift/spend, and the dates/fiscal weeks it ran. A dimension of the POS `sales` fact (one version per promotion). Includes the reserved `NO_PROMO` member so non-promoted sales attribute to a real surrogate. Promotion **performance** (ROI, lift, baseline) is computed in the Promote with Purpose outcome package, which consumes this dimension. |

_TODO: expand as additional marketing tables (campaign, offer, coupon) ship._
