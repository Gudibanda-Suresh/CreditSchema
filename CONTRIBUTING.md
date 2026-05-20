# Contributing to ION Network Specification

Thank you for contributing. Please follow these steps.

## Prerequisites

- Familiarity with Beckn Protocol v2.0.0 (`schema/core/v2/api/v2.0.0/beckn.yaml`)
- Read `docs/ION_Schema_Style_Guide.md` before authoring or editing any schema pack
- Read `docs/ION_Entity_Hierarchy_Model.md` to understand ION's sector/category structure
- Read `schema/core/v2/api/v2.0.0/README.md` to understand the two-file spec model and which layer (L2/L3/L4/L5) your change belongs to

## Workflow

1. Open an issue describing the proposed change (for non-trivial changes)
2. Fork the repo and create a feature branch: `git checkout -b feat/my-change`
3. Make your changes following the conventions below
5. Submit a pull request with a clear description and link to the issue

## Conventions

### Schema packs (`schema/extensions/`)

Every pack version folder (`vN/`) MUST contain:

| File | Purpose |
|---|---|
| `attributes.yaml` | OpenAPI 3.1.1 schema — primary source of truth |
| `schema.json` | JSON Schema mirror (generated from attributes.yaml) |
| `context.jsonld` | JSON-LD term mappings — MUST include `@protected: true` |
| `vocab.jsonld` | RDF vocabulary (rdfs:Class + rdf:Property per field) |
| `profile.json` | Attachment metadata — MUST include `type`, `indexing`, `minimalForDiscovery`, `packSchema: "./schema.json"` |
| `renderer.json` | UI rendering hints |
| `README.md` | Property table for this version |

The concept folder (parent of `vN/`) MUST contain an outer `README.md` with the version index table.

### $ref discipline

Always reference `beckn.yaml` via the **local relative path**:

```yaml
$ref: ../../../../core/v2/api/v2.0.0/beckn.yaml#/components/schemas/Attributes
```

**Never** use the external GitHub raw URL. The local copy is the pinned, validated version.

### Naming

- Extension concept folders: `kebab-case` (e.g. `participant-logistics`)
- Schema object names: `PascalCase` with domain prefix (e.g. `TradeResource`)
- JSON-LD term namespace: `ion:` prefix (e.g. `ion:TradeResource`)

### Pattern/variant flows (`flows/`)

Field paths in `pattern.yaml` and `variant.yaml` files MUST use Beckn v2.0 Catalog structure:

```yaml
# Correct (v2.0)
message.catalog.resources[].resourceAttributes.*
message.catalog.offers[].offerAttributes.*
message.catalog.provider.providerAttributes.*

# Wrong (v1.x legacy — do not use)
message.catalog.resources[].resourceAttributes.*
```

### Policies (`policies/`)

New policies MUST follow the IRI grammar: `ion://policy/{category}.{subcategory}.{specifier}[.{qualifier}]`

Run `python3 policies/generate_registry.py` after adding any policy file.

### Cross-schema conditional rules

When a conditional requirement cannot be expressed with JSON Schema `if/then/else` because the trigger field lives in a different schema bag, add a rule to the `x-ion-conditional-rules:` block in `ion.yaml`. ONIX reads this block at startup and enforces all rules at every API boundary. Do not create a separate rules file.

## Validation

```bash
```

All checks must pass before a pull request will be merged.
