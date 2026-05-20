# delivery-order — Hospitality Pattern v1

Food and beverage ordered for delivery or self-pickup. Item is **prepared fresh after order confirmation**.

**Sector:** `ion:hospitality`  
**CRC:** `HSC-delivery`  
**Pattern:** `delivery-order`

## State sequence

```
ACCEPTED → PREPARING → READY_FOR_PICKUP → OUT_FOR_DELIVERY → DELIVERED
                                        → PICKED_UP  (self-pickup mode)
```

## Applicable variants

- `cross-cutting` — always active (track, support, rating, reconcile, raise)
- `cancellation` — before preparation starts (typically 5 min window)
- `cash-on-delivery` — when `offer.availableOnCod=true`

## Path notation example

```
hospitality / HSP-05 / HSC-delivery / delivery-order / cash-on-delivery
```

## Key differences from trade/storefront

| | `storefront` (Trade) | `delivery-order` (Hospitality) |
|---|---|---|
| Item state at order | In stock, pre-packaged | Does not exist — made to order |
| Returns | Yes (policy-dependent) | No — `returnable: false` always |
| Cancellation | Up to dispatch | Only before PREPARING starts |
| Inventory | Stock-based | Capacity-based |
| Logistics | External LSP | Usually embedded in BPP |

See `pattern.yaml` for the full API sequence.
