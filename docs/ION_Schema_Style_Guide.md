# ION Schema Style Guide

Authoritative conventions for pack authors. All contributions must follow this guide.

## 1. Layer model reminder

| Layer | Location | What goes here |
|---|---|---|
| L1 | `beckn.yaml` (external) | Beckn core — never modified |
| L2 | `schema/core/v2/api/v2.0.0/ion.yaml` | ION network profile overlay |
| L3 | `schema/core/v2/api/v2.0.0/ion.yaml` → `paths:` block | `/raise` family (6 endpoints) + `/reconcile` + `/on_reconcile` |
| L4 | `schema/extensions/core/*/v1/` | Cross-sector attribute packs |
| L5 | `schema/extensions/{trade,logistics}/*/v1/` | Sector-specific attribute packs |

## 2. Required files per pack version

Every `vN/` folder must contain all of these:

```
attributes.yaml    ← Primary source of truth (OpenAPI 3.1.1)
schema.json        ← JSON Schema mirror (generated)
context.jsonld     ← JSON-LD context (@protected:true required)
vocab.jsonld       ← RDF vocabulary
profile.json       ← Attachment metadata
renderer.json      ← UI rendering hints
README.md          ← Property table
```

The parent concept folder (above `vN/`) must also have a `README.md` version-index table.

## 3. attributes.yaml rules

### info block (required)
```yaml
info:
  title: "ION {Domain} {Concept} Extension (v{N})"
  description: "..."
  version: 1.0.0
  license:
    name: Apache-2.0
    url: https://www.apache.org/licenses/LICENSE-2.0
  contact:
    name: ION Network
    url: https://schema.ion.id
```

### Schema object annotation (required)
Every main schema object must carry:
```yaml
x-jsonld:
  '@context': ./context.jsonld
  '@type': ion:SchemaName
x-beckn-attaches-to: BecknObject.attributesSlot
```

### Legal x-beckn-attaches-to values
- `Resource.resourceAttributes`
- `Offer.offerAttributes`
- `Contract.contractAttributes`
- `Commitment.commitmentAttributes`
- `Consideration.considerationAttributes`
- `Performance.performanceAttributes`
- `Settlement.settlementAttributes`
- `Provider.providerAttributes`
- `Participant.participantAttributes`
- `Tracking.trackingAttributes`
- `RatingInput.target.targetAttributes`
- `Support.channels[*]` (array slot — use @type discriminator)

### allOf with Beckn Attributes base (required)
```yaml
allOf:
  - $ref: ../../../../core/v2/api/v2.0.0/beckn.yaml#/components/schemas/Attributes
```

**Never use the external GitHub URL.** Always use the local relative path.

### additionalProperties
Set `additionalProperties: true` on all extension bags (allows NPOS overlay schemas).
Add `x-ion-closed-extensions: false` to document this is intentional.

### Conditional requirements
For conditions within the same schema, use JSON Schema `if/then/else`:
```yaml
- if:
    properties:
      ageRestricted:
        const: true
    required: [ageRestricted]
  then:
    required: [minAge]
```

For cross-schema conditions (trigger field in a different schema bag), add a rule to the `x-ion-conditional-rules:` block in `ion.yaml`. ONIX reads this block and enforces the rule at every API boundary.

## 4. context.jsonld rules

```json
{
  "@context": {
    "@version": 1.1,
    "@protected": true,
    "ion": "https://schema.ion.id/{domain}/{concept}/v{N}#",
    "beckn": "https://spec.beckn.io/vocab/v2.0.0#",
    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "schema": "https://schema.org/",
    "SchemaName": "ion:SchemaName",
    "fieldName": "ion:fieldName"
  }
}
```

`@protected: true` is mandatory. All four standard prefixes (`rdf`, `rdfs`, `xsd`, `schema`) must be present.

## 5. vocab.jsonld rules

Every class and property needs a full `rdfs:comment` (not truncated):

```json
{
  "@graph": [
    {
      "@id": "ion:SchemaName",
      "@type": "rdfs:Class",
      "rdfs:label": "Schema Name",
      "rdfs:comment": "Full description from attributes.yaml"
    },
    {
      "@id": "ion:fieldName",
      "@type": "rdf:Property",
      "rdfs:label": "fieldName",
      "rdfs:comment": "Full field description"
    }
  ]
}
```

## 6. profile.json rules

Required fields:
```json
{
  "id": "ion-{domain}-{concept}-v{N}",
  "version": "1.0.0",
  "domain": "{domain}",
  "type": "AttributePack",
  "name": "ION {Domain} {Concept} Extension (v{N})",
  "packSchema": "./schema.json",
  "jsonldContext": "./context.jsonld",
  "extends": "beckn:{BecknObject}",
  "attachmentPoints": { "slotName": ["SchemaName"] },
  "indexing": {
    "indexable": [{"path": "$.fieldName"}],
    "filterable": [{"path": "$.fieldName"}],
    "sortable": []
  },
  "minimalForDiscovery": ["fieldName"]
}
```

## 7. Naming conventions

| Thing | Convention | Example |
|---|---|---|
| Concept folder name | `kebab-case` | `participant-logistics` |
| Schema object name | `PascalCase` with domain prefix | `TradeResource` |
| JSON-LD term IRI | `ion:PascalCase` or `ion:camelCase` | `ion:TradeResource` |
| Field names | `camelCase` | `resourceStructure` |
| Policy IRIs | `ion://policy/{cat}.{sub}.{spec}` | `ion://policy/cancel.standard.until-dispatched` |

## 8. context.schemaContext usage

Every API call should populate `context.schemaContext` with the ION extension context URIs relevant to the message:

```yaml
context:
  schemaContext:
    - https://schema.ion.id/trade/resource/v1/context.jsonld
    - https://schema.ion.id/trade/offer/v1/context.jsonld
    - https://schema.ion.id/core/payment/v1/context.jsonld
```

This makes every message self-describing and enables deterministic validation at ION Central.
