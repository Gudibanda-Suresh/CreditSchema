# Schema Extensions

This directory contains ION's attribute pack extensions — the field definitions that populate Beckn's `*Attributes` slots.

## The core idea

Beckn v2.0 defines a data model with extension points built in. Every core object has an `*Attributes` slot that accepts a JSON-LD typed object. ION uses these slots to add Indonesia-specific fields without touching Beckn itself.

```
Beckn core object        ION extension pack          What it adds
─────────────────────    ──────────────────────────  ────────────────────────────────
Resource                 trade/resource/v1           Product structure, availability,
  └─ resourceAttributes  ◄── TradeResource  food/fashion/electronics fields
                                                      
Offer                    trade/offer/v1              Return policy, cancellation policy,
  └─ offerAttributes     ◄── TradeOffer     COD availability, proof of delivery

Settlement               core/payment/v1             Payment method details (QRIS,
  └─ settlementAttributes ◄── PaymentDeclaration      VirtualAccount, EWallet, COD...)

Participant              core/participant/v1          Role taxonomy, NPWP/NIB/NIK,
  └─ participantAttributes ◄── IONParticipant  address hierarchy with RT/RW
```

Multiple packs can attach to the same slot simultaneously. A `Settlement` record in a reconciliation step carries both `PaymentDeclaration` (from `core/payment/v1`) and `IONReconciliation` (from `core/reconcile/v1`). They are distinguished by their `@type` field.

## Directory structure

```
extensions/
  core/              ← L4: cross-sector packs — apply to every ION sector
  trade/             ← L5: Trade sector packs (active)
  logistics/         ← L5: Logistics sector packs (active)
  mobility/          ← L5: reserved
  finance/           ← L5: reserved
  tourism/           ← L5: reserved
  healthcare/        ← L5: reserved
```

## What each pack contains

Every versioned pack folder (`v1/`) holds exactly these files:

| File | What it is |
|---|---|
| `attributes.yaml` | OpenAPI 3.1.1 schema — the authoritative definition of every field |
| `schema.json` | JSON Schema mirror of attributes.yaml — for validators that don't parse OAS |
| `context.jsonld` | JSON-LD context — maps field names to `ion:` namespace IRIs |
| `vocab.jsonld` | RDF vocabulary — defines each field as an `rdf:Property` with labels and comments |
| `profile.json` | Attachment metadata — which Beckn object this pack extends, indexing hints |
| `renderer.json` | UI rendering hints — labels, display order, field groupings for BAP UIs |
| `README.md` | Property table — every field, its type, whether required, and a plain-English description |
| `docs/` | Narrative documentation — design rationale, worked examples, field-level guidance |
| `examples/` | Concrete JSON examples — complete valid payload fragments |

The concept folder above `v1/` also has a `README.md` that shows all available versions in a table.

## How a pack declares its attachment point

Every pack's main schema object carries an `x-beckn-attaches-to` annotation:

```yaml
# schema/extensions/trade/resource/v1/attributes.yaml

TradeResource:
  type: object
  x-beckn-attaches-to: Resource.resourceAttributes
  x-jsonld:
    '@context': ./context.jsonld
    '@type': ion:TradeResource
  allOf:
    - $ref: ../../../../core/v2/api/v2.0.0/beckn.yaml#/components/schemas/Attributes
  properties:
    resourceStructure:
      type: string
      enum: [PLAIN, VARIANT, WITH_EXTRAS, COMPOSED, BUNDLE]
    availability:
      # ...
```

The `allOf` with `beckn.yaml#/.../Attributes` means this schema inherits Beckn's requirement that every `*Attributes` object must have `@context` and `@type`. Both fields are automatically required.

## Composition example

Here is how multiple packs layer onto a single Beckn `Resource` in a trade catalog:

```json
{
  "id": "item-nasi-goreng-spesial",
  "descriptor": {
    "name": "Nasi Goreng Spesial"
  },
  "resourceAttributes": {
    "@context": [
      "https://schema.ion.id/core/localization/v1/context.jsonld",
      "https://schema.ion.id/core/product/v1/context.jsonld",
      "https://schema.ion.id/trade/resource/v1/context.jsonld"
    ],
    "@type": "ion:TradeResource",

    "name": { "id": "Nasi Goreng Spesial", "en": "Special Fried Rice" },
    "halalStatus": "HALAL",
    "halalCertNumber": "ID38010000151020",

    "resourceStructure": "COMPOSED",
    "resourceTangibility": "PHYSICAL",
    "availability": { "status": "IN_STOCK" },
    "countryOfOrigin": "IDN",
    "ageRestricted": false,
    "images": ["https://cdn.seller.id/items/nasi-goreng.jpg"],
    "logisticsServiceType": "HYPERLOCAL",
    "food": {
      "classification": "HALAL",
      "spiceLevel": "HIGH",
      "allergens": []
    },
    "preparationTime": "PT15M",
    "customisationGroups": [
      {
        "id": "CG-SPICE",
        "name": { "id": "Level Pedas", "en": "Spice Level" },
        "selectionType": "SINGLE_REQUIRED",
        "options": [
          { "id": "OPT-MILD",  "label": { "id": "Tidak Pedas" } },
          { "id": "OPT-HOT",   "label": { "id": "Pedas" },      "priceDelta": { "type": "FIXED", "value": 0 } },
          { "id": "OPT-XHOT",  "label": { "id": "Extra Pedas" }, "priceDelta": { "type": "FIXED", "value": 2000 } }
        ]
      }
    ]
  }
}
```

The `name` and `halalStatus` fields come from `core/localization` and `core/product` packs. Everything else comes from `trade/resource`. At runtime, all fields coexist in the same JSON object — there is no nesting or merging step needed.

## Adding a new sector

1. Create `extensions/{sector}/README.md` — describe the sector and list which core packs it uses
2. Create pack folders following the same structure as an existing sector
3. Add the sector to `schema/core/v2/api/v2.0.0/ion.yaml` under `x-ion-conformance.schemaPackMatrix`
4. Opening a new sector requires ION Council ratification — see `GOVERNANCE.md`

## Pack versioning

Packs version independently using integer versions (`v1`, `v2`). When a pack needs a breaking change, a new version folder is created alongside the old one. Old versions are never deleted — implementations pinned to `v1` continue to work. The `profile.json` in each version declares its own identifier. The concept-level `README.md` shows the version index.
