# RestaurantPerformance — ION Hospitality Restaurant Ordering Extension

**Pack:** `hospitality/performance/v1`  
**Class:** `ion:RestaurantPerformance`  
**Attaches to:** `Performance.performanceAttributes`  
**Sector:** Hospitality — Restaurant & Food Ordering (HSC-restaurant-ordering)  
**Version:** v1  

## Purpose

RestaurantPerformance provides kitchen state, rider tracking, live GPS, delivery proof for online food ordering platforms
(Grab Food, Gojek GoFood, Zomato, Swiggy style applications).

## Attachment

This pack extends the `Performance.performanceAttributes` slot in the Beckn v2.0.0 object model.
Every message carrying this pack MUST set:

```json
{
  "@context": "https://schema.ion.id/hospitality/performance/v1/context.jsonld",
  "@type": "ion:RestaurantPerformance"
}
```

## Conditional requirements

Fields marked `x-ion-condition:` are conditionally required.
ONIX enforces these at the API boundary via Rego policy.
See `docs/ION_ONIX_Conditional_Policy.md` in the repo root.

## Related packs

All HSC-restaurant-ordering packs work together:

| Pack | Beckn slot | Purpose |
|---|---|---|
| `delivery/v1` | Resource | Menu item definition, halal, allergens, customisations |
| `provider/v1` | Provider | Restaurant profile, hours, licensing, delivery zones |
| `offer/v1` | Offer | Pricing, delivery fees, promos, scheduling |
| `commitment/v1` | Commitment | Order line with customisation selections |
| `consideration/v1` | Consideration | Price breakdown, fees, tax, cashback |
| `contract/v1` | Contract | Confirmed order, delivery prefs, group order |
| `performance/v1` | Performance | Kitchen states, rider tracking, ETA, delivery proof |

## Examples

See `examples/` for complete payload examples.
