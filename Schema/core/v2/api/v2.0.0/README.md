# ION Protocol Specification — Core API Layer

This directory contains exactly two files. That is intentional.

```
beckn.yaml   ← Beckn Protocol v2.0.0 (upstream, do not edit)
ion.yaml     ← ION network extension (this is what you implement)
```

## How they relate

`beckn.yaml` is the upstream Beckn Protocol specification. It defines:
- 30 API endpoints (`/discover`, `/select`, `/init`, `/confirm`, `/status`, etc.)
- All core data model schemas (`Contract`, `Resource`, `Offer`, `Commitment`, `Consideration`, `Performance`, `Settlement`, `Provider`, `Participant`, `Tracking`, `Attributes`, …)

`ion.yaml` extends `beckn.yaml` without copying it. It defines:
- **L2** — ION network profile: Indonesian regulatory rules, signing requirements, allowed payment rails, conformance matrix (`x-ion-profile` block)
- **L3** — 8 additional ION endpoints: `/raise` family (6) + `/reconcile` + `/on_reconcile`. Every path `$ref`s Beckn types from `beckn.yaml`
- **L4** — 11 cross-sector attribute packs that mount on Beckn's `*Attributes` slots
- **L5** — Trade (8) and Logistics (10) sector attribute packs, same mounting pattern
- **Cross-schema rules** — 5 conditional requirements that ONIX enforces at runtime (`x-ion-conditional-rules` block)

## To validate

Run both files together through any OpenAPI 3.1.1 validator:
```
spectral lint ion.yaml --ruleset beckn-ruleset.yaml
```

If your toolchain requires a single merged file:
```
```

## Upstream tracking

When Beckn releases a new version, only `beckn.yaml` needs updating. All 42 `$ref: beckn.yaml#/...` references in `ion.yaml` automatically resolve to the new version. No other files need changing for a Beckn version bump.

Current Beckn target: **v2.0.0** (`core-v2.0.0-rc1`)
