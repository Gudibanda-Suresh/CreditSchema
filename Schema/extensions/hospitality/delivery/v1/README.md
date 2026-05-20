# hospitality/delivery/v1

ION Hospitality sector extension pack for prepared food and beverage delivery.

**CRC:** `HSC-restaurant-ordering`  
**Covers:** Food delivery apps (GoFood, GrabFood, Tokopedia Food), restaurant online ordering, cloud kitchen delivery  
**Beckn attachment:** `Resource.resourceAttributes`  
**Sector:** `ion:hospitality`  
**Pattern:** `delivery-order`, `dine-in-order`

## What this covers

Restaurant meals and beverages ordered for delivery or self-pickup. The resource is a **prepared food item** — made after order placement. This is distinct from Trade food (`TRC-food-bev`) which covers packaged physical goods.

**Classification rule:** If the buyer ends up owning a physical packaged product → Trade (`TRC-food-bev`). If the item is prepared fresh after the order and consumed within hours → Hospitality (`HSC-restaurant-ordering`).

| Examples | CRC |
|---|---|
| Nasi goreng from a restaurant delivery app | `HSC-restaurant-ordering` |
| GoFood/GrabFood meal | `HSC-restaurant-ordering` |
| Packaged Indomie from Tokopedia | `TRC-food-bev` (Trade) |
| AQUA 600ml from an apotek | `TRC-food-bev` (Trade) |

## Required fields

| Field | Type | Notes |
|---|---|---|
| `quantity` | object | `{value, unit}` — unit from: portion, piece, cup, bowl, plate, set |
| `resourceStructure` | string | PLAIN / WITH_EXTRAS / COMPOSED / BUNDLE |
| `resourceTangibility` | string | Always `PHYSICAL` |
| `images` | array | Min 1. First image is primary. |
| `availability.status` | string | IN_STOCK / OUT_OF_STOCK / PREORDER |
| `ageRestricted` | boolean | `true` for alcoholic beverages |
| `countryOfOrigin` | string | ISO alpha-3. Typically `IDN`. |
| `preparationTime` | string | ISO 8601 duration e.g. `PT20M` |
| `classification` | string | HALAL / NON_HALAL / VEG / NON_VEG / HALAL_PENDING / NOT_APPLICABLE |
| `allergens` | array | Required declaration. Empty array = none declared. |

## Conditional fields

| Field | Condition |
|---|---|
| `halalCertNumber` | Required when `classification = HALAL` |
| `customisationGroups` | Required when `resourceStructure IN [COMPOSED, WITH_EXTRAS]` |
| `isAlcoholic: true` | Requires `ageRestricted: true` on the resource |

## Example minimal resource

```json
{
  "@context": "https://schema.ion.id/hospitality/delivery/v1/context.jsonld",
  "@type": "ion:RestaurantMenuItem",
  "quantity": { "value": 1, "unit": "portion" },
  "resourceStructure": "PLAIN",
  "resourceTangibility": "PHYSICAL",
  "logisticsServiceType": "HYPERLOCAL",
  "ageRestricted": false,
  "countryOfOrigin": "IDN",
  "images": ["https://cdn.example.com/nasi-goreng.jpg"],
  "availability": { "status": "IN_STOCK" },
  "preparationTime": "PT15M",
  "classification": "HALAL",
    "allergens": [],
    "halalCertNumber": "MUI-ID-2025-00012345",
    "cuisineType": "Indonesian",
    "spiceLevel": "MEDIUM",
    "isAlcoholic": false,
  "menuCategory": {
    "id": "CAT-RICE",
    "name": { "id": "Nasi", "en": "Rice Dishes" },
    "sequence": 1
  }
}
```

See `docs/` for design notes and `examples/` for full payloads.
