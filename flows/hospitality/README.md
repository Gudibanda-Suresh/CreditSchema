# Hospitality Sector Flows

**Sector:** `ion:hospitality`  
**Active CRCs:** `HSC-restaurant-ordering` (Restaurant & Food Ordering)

---

## Patterns

| Pattern | CRC | Status | Description |
|---|---|---|---|
| [`delivery-order/v1`](patterns/delivery-order/v1/pattern.yaml) | `HSC-restaurant-ordering` | Ready | Restaurant meal or beverage ordered for delivery or self-pickup. The item is prepared after order placement. Covers GoFood, GrabFood, Tokopedia Food, and any ION-connected food ordering platform. |

---

## Variants

| Variant | Applies to | Description |
|---|---|---|
| [`cross-cutting/v1`](variants/cross-cutting/v1/variant.yaml) | All hospitality patterns | Track, support, rate, reconcile — always available once contract is ACTIVE |
| [`cancellation/v1`](variants/cancellation/v1/variant.yaml) | delivery-order | Cancellation within the tight pre-kitchen window (typically 60-90 seconds) |
| [`cash-on-delivery/v1`](variants/cash-on-delivery/v1/variant.yaml) | delivery-order | Consumer pays rider in cash at doorstep |
| [`pre-order/v1`](variants/pre-order/v1/variant.yaml) | delivery-order | Scheduled advance delivery — consumer selects a future delivery slot |
| [`group-order/v1`](variants/group-order/v1/variant.yaml) | delivery-order | Multi-participant group ordering (Grab Group Order / GoFood Pesan Bareng style) |
| [`age-verification/v1`](variants/age-verification/v1/variant.yaml) | delivery-order | Doorstep age verification for orders containing alcoholic beverages |

---

## Schema packs used by this sector

| Beckn slot | Pack | Class |
|---|---|---|
| Resource | `hospitality/delivery/v1` | `RestaurantMenuItem` |
| Offer | `hospitality/offer/v1` | `RestaurantOffer` |
| Provider | `hospitality/provider/v1` | `RestaurantProvider` |
| Commitment | `hospitality/commitment/v1` | `RestaurantCommitment` |
| Consideration | `hospitality/consideration/v1` | `HospitalityConsideration` |
| Contract | `hospitality/contract/v1` | `HospitalityContract` |
| Performance | `hospitality/performance/v1` | `RestaurantPerformance` |

---

## Sector boundary

The key classification test: **if the item is prepared fresh after the order and consumed within hours → Hospitality**. If the buyer ends up owning a physical packaged product → Trade (`TRC-food-bev`).

| Example | Sector |
|---|---|
| Nasi Goreng from GoFood | Hospitality (`HSC-restaurant-ordering`) |
| Aqua mineral water (1.5L) from Tokopedia | Trade (`TRC-food-bev`) |
| Custom birthday cake ordered 3 days ahead | Trade (`TRC-bakery`) |
| Meal kit (cold, cook-at-home) | Trade (`TRC-meal-kit`) |
| Restaurant reservation (dine-in, future) | Hospitality (`HSC-restaurant`) — reserved |
