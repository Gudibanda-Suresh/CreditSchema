# Hospitality Restaurant Ordering — Design Notes

## Why a separate sector from Trade?

The Hospitality sector covers **time-bounded experiences and prepared services**. Food delivery from a restaurant is Hospitality because:

1. The item does not exist until ordered — it is made fresh
2. The item is perishable within hours — not a stockable good
3. Cancellation policy closes at preparation start — not at dispatch
4. No return window — prepared food cannot be returned
5. The transaction pattern is `delivery-order` (or `dine-in-order`) — not `storefront`

Trade (`TRC-food-bev`) covers packaged food that is stocked, inventoried, and shipped — Indomie, bottled water, canned goods, protein powder, dairy products.

## Sector: ion:hospitality vs ion:trade

| Question | Hospitality Restaurant Ordering | Trade F&B |
|---|---|---|
| Is the item made fresh after order? | Yes | No |
| Is inventory managed? | No (capacity-based) | Yes |
| Can it be returned? | No | Depends on policy |
| Pattern | `delivery-order` | `storefront` |
| CRC | `HSC-restaurant-ordering` | `TRC-food-bev` |

## Customisation model

Restaurant ordering commonly uses COMPOSED resources with mandatory and optional customisation groups:

- **SINGLE_REQUIRED** — buyer must choose exactly one option (e.g. size, spice level)
- **SINGLE_OPTIONAL** — buyer may choose one option or skip (e.g. sauce on the side)
- **MULTIPLE_OPTIONAL** — buyer may choose 0–N options (e.g. toppings)
- **FIXED** — no choice, informational only (e.g. included side dish)

Price deltas are expressed as FIXED amounts in IDR added to the base price.

## Halal and allergen requirements

All restaurant ordering resources MUST declare:
- `classification` — the halal/dietary status
- `allergens` — as an array (empty array is a valid declaration of "none")

These drive mandatory BAP display: buyer apps must surface classification and allergen information on item detail pages before the buyer can add to cart. Failure to declare is a catalog publish error.

## BPOM for hospitality Restaurant Ordering

Most restaurant-prepared food does not require individual BPOM registration. Packaged sauces, condiments, or beverages that the restaurant uses as ingredients may carry BPOM numbers — these are optional to declare via `bpomRegistration`.

Pre-packaged items sold alongside prepared food (e.g. a bottled drink) should be listed as separate Trade (`TRC-food-bev`) resources.

## Alcohol

For alcoholic beverages:
- `ageRestricted: true` is mandatory
- `isAlcoholic: true` is mandatory
- `classification` should be `NON_HALAL`
- Regulatory: Permendag 20/M-DAG/PER/4/2014

## What this pack does NOT cover

- Table reservations → `HSC-restaurant` (not yet active)
- Packaged food for home delivery from supermarkets → Trade `TRC-food-bev`
- Catering/event food → `HSC-events` (not yet active)
