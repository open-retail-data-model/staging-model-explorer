# Marketing Domain — Business Glossary

> Status: 🟡 In progress (0.1-beta)

| Term | Definition |
|---|---|
| **Promotion** (`promotion`) | The conformed trade-promotion dimension (SCD2) — mechanics, funding, planned lift/spend, and the dates/fiscal weeks it ran. A dimension of the POS `sales` fact (one version per promotion). Includes the reserved `NO_PROMO` member so non-promoted sales attribute to a real surrogate. Promotion **performance** (ROI, lift, baseline) is computed in the Promote with Purpose outcome package, which consumes this dimension. |
> The former conformed `device` and `audience` dimensions were relocated to the gold
> `commerce-media-networks` package (single-outcome retail media; ADR 0021): `device`
> became the ad-surface `delivery_channel` dim, and `audience` moved as-is (it is
> consumed only by retail-media ad delivery). `marketing` now holds only the conformed
> `promotion` dimension.

_TODO: expand as additional shared marketing tables (offer, coupon) ship._
