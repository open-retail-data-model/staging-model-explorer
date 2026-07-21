# Balancing Margin and Volume - Glossary

| Term | Definition |
|---|---|
| **Price Gap** | The difference in price between our product and a competitor's identical or equivalent product. |
| **Price Index** | A ratio comparing our price to a competitor's price (e.g., 1.05 means we are 5% more expensive). |
| **Substitute** | A product that can be purchased instead of another (e.g., different brand of milk). Used in cannibalization modeling. |
| **Complement** | A product that is often bought together with another (e.g., hot dogs and hot dog buns). Used in basket affinity analysis. |
| **Affinity Coefficient** | A score between 0 and 1 indicating how strongly two products are related (substitutes or complements). |
| **Weeks of Cover** | Inventory health indicator categorizing stock life (e.g., '<1 Week', 'Overstock'). |
| **Markdown Depth Pct** | The percentage depth of the markdown relative to base price. |
| **Stockout Adjusted Demand** | De-seasonalized and de-promoted normal demand adjusted for out-of-stock periods. |
| **Price Barrier Proximity** | Measures proximity to psychological boundaries like .99. |
| **Margin Health Bucket** | Categorical indicator of gross margin performance (e.g., 'High', 'Healthy', 'Bleeding'). |
| **Competitive Pressure Bucket** | Categorical indicator of competitor price aggression (e.g., 'Severe', 'Moderate', 'Low'). |
| **Discount Depth Bucket** | Categorical depth of the current markdown (e.g., 'Clearance >50%', 'Shallow <10%'). |
| **Cross Elasticity Proxy** | The substitution effect drawn from cannibalized items. |
| **Point Own Price Elasticity** | The percentage change in volume for a 1% change in price (PED). |
| **Planogram** | A visual representation of a stores products or services, used to maximize sales and space. |
| **Allocated Facings** | The number of physical items of a specific product visible at the front of a retail shelf. |
| **Allocated Sqft** | The total square footage assigned to a specific product on a retail shelf. |
| **Space Elasticity** | The sensitivity of sales volume to changes in the amount of shelf space allocated to a product. |
| **Substitute Demand Transfer Pct** | The probability that demand transfers to a substitute product when an item is unavailable or dropped. |
| **Rolling Sales Velocity 14d** | The 14-day rolling average of daily units sold. |
| **Baseline Velocity** | The expected volume sold in the absence of promotions. |
| **Margin Contribution Pct** | An item's 14-day realized gross-margin rate: rolling gross-margin dollars divided by rolling net revenue on days that carry a gross-margin rate, over the same 14-day window (ratio-of-sums). Days lacking a margin rate are excluded from the denominator so the rate is not biased downward. |
| **Assortment Recommendation** | Rule-based recommendation for an item's status in the assortment (KEEP, DROP, EXPAND), derived deterministically from sales velocity, margin health, and days-on-hand. |
| **Projected Volume Lift** | The expected change in volume from a recommended assortment action. |
| **Projected Margin Lift** | The expected change in margin from a recommended assortment action. |
| **Cannibalization Drag** | The expected loss in sales on other items due to demand transfer or cannibalization. |
| **Settled Sale Amount** | Allocated magnitude of sale payments in a settled or captured state, spread to product × store × day. The denominator base for payment-side leakage rates. |
| **Refund Amount** | Allocated magnitude of merchant-initiated refunds (contra-revenue), from the payment domain. Distinct from a chargeback. |
| **Chargeback Amount** | Allocated magnitude of bank-initiated payment reversals (disputes). Kept distinct from refunds: involuntary, typically irrecoverable, and never touches the POS sales fact. |
| **Adjustment Amount** | Allocated magnitude of post-sale payment adjustments (goodwill / corrections); treated as contra-revenue. |
| **Revenue Leakage** | Booked revenue not ultimately realized, surfaced via the payment side: refunds, chargebacks, and adjustments that reduce realized margin below the sales-fact figure. |
| **Realized Margin** | Gross margin after subtracting payment-side leakage (refunds, chargebacks, adjustments); the money actually kept, distinct from booked (net-of-discount) margin. |
