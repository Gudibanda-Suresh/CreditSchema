# ION — Indonesia Open Network

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE.md)
[![Spec Version](https://img.shields.io/badge/Spec-v0.6.0--draft-orange.svg)](CHANGELOG.md)
[![Beckn](https://img.shields.io/badge/Built%20on-Beckn%20v2.0.0-green.svg)](https://beckn.io)

ION is Indonesia's open digital commerce network. Any buyer app and any seller app on ION can transact with each other — without bilateral integration agreements.

A seller who registers on ION is reachable by every buyer app on the network, across food, grocery, fashion, electronics, beauty, agritech, and the logistics network that moves goods between them.

> **Status.** v0.6.0-draft. Not yet in production. Production launch schedule is set by ION Council.

---

## What this repository is

This repository is the **ION Network Specification** — the single source of truth for how ION works. It contains:

- The schema definitions for every field in every ION message
- The flow specifications for every transaction pattern
- The policy registry that governs commercial terms
- The error registry that defines what ION rejects and why

It is **not** an SDK, a client library, or a running service. It is the spec that all of those are built from.

---

## How ION relates to Beckn

ION is built on top of **Beckn Protocol v2.0.0** — an open protocol for interoperable commerce networks. ION does not fork or modify Beckn. It extends it.

Think of it this way:

```
Beckn Protocol v2.0.0
  └── defines: 30 API endpoints, core data model
       (Contract, Resource, Offer, Commitment, Consideration,
        Performance, Settlement, Provider, Participant...)
       Each core object has an *Attributes slot for extensions.

ION Network (this repo)
  └── extends Beckn with:
       ├── 8 additional endpoints  (/raise family, /reconcile)
       ├── Indonesian network rules  (signing, NPWP, data residency)
       ├── 11 cross-sector attribute packs  (address, identity, payment, tax...)
       └── 18 sector attribute packs  (trade: 8, logistics: 10)
```

The extension mechanism is simple: every ION schema pack declares which Beckn object it extends, using the `x-beckn-attaches-to` annotation. For example:

```yaml
# In schema/extensions/trade/resource/v1/attributes.yaml
TradeResource:
  x-beckn-attaches-to: Resource.resourceAttributes
  # ...fields for product type, availability, category-specific attributes
```

This means a Beckn `Resource` object — which has an `id`, a `descriptor`, and a `resourceAttributes` slot — gets ION's trade fields packed into that slot. Beckn stays unchanged. ION adds fields on top.

---

## The two spec files

The entire ION specification lives in two files, side by side:

```
schema/core/v2/api/v2.0.0/
  beckn.yaml   ← Beckn Protocol v2.0.0 (vendored, do not edit)
  ion.yaml     ← ION extension specification (everything ION adds)
```

`beckn.yaml` is a vendored copy of Beckn's upstream spec, pinned to a specific release. It is kept locally so that validators and tooling work offline. The header inside it explains how to upgrade it when Beckn releases a new version.

`ion.yaml` contains all ION extensions. Every reference to a Beckn type uses `$ref: beckn.yaml#/...` — pointing to the local copy, never to an external URL. If you change `beckn.yaml` to a newer Beckn version, `ion.yaml` automatically picks up the changes.

---

## Repository structure

```
ion-specs/
│
├── schema/
│   ├── core/v2/api/v2.0.0/
│   │   ├── beckn.yaml          ← Beckn Protocol v2.0.0 (vendored)
│   │   └── ion.yaml            ← ION extension spec (L2 + L3 + L4 + L5)
│   │
│   └── extensions/             ← Individual attribute packs (source of truth for fields)
│       ├── core/               ← 11 cross-sector packs (address, identity, payment, tax...)
│       ├── trade/              ← 8 trade sector packs
│       └── logistics/          ← 10 logistics sector packs
│
├── flows/                      ← Transaction flow specifications
│   ├── trade/                  ← 12 commerce patterns (storefront, made-to-order, subscription, live-commerce, digital-goods, business-procurement, marketplace-inhouse, marketplace-listed, forward-auction, reverse-auction, cross-border, government)
│   └── logistics/              ← 6 logistics archetypes (parcel, freight...)
│
├── policies/                   ← Machine-readable policy terms (return, cancellation, SLA...)
├── errors/                     ← Error registry (ION-1xxx through ION-9xxx)
├── docs/                       ← Reference documents, worked examples, and contributor guides
│   ├── ION_Bu_Sari_Wizard.md          ← Plain-language seller journey (start here if you're new to ION)
│   ├── ION_Bu_Sari_Complete_Journey.md ← Technical companion: full API calls and JSON payloads
│   ├── ION_Entity_Hierarchy_Model.md  ← Onboarding classification model (sectors, categories, CRCs, patterns)
│   ├── ION_Resource_Categories.md     ← Complete CRC reference table (all 46 categories across 6 sectors)
│   └── ION_Schema_Style_Guide.md      ← Authoritative guide for schema pack authors
```

**How the parts connect.** A flow spec in `flows/` references field paths like `message.catalog.resources[].resourceAttributes.food.classification`. That field is defined in `schema/extensions/trade/resource/v1/attributes.yaml`. Its commercial terms (what happens if it is wrong) are in `policies/`. The error code if ION rejects it is in `errors/`.

---

## The five-layer model

ION composes with Beckn in five distinct layers. Knowing which layer something belongs to is the most important piece of orientation for anyone reading this repo.

| Layer | What it is | Where it lives |
|---|---|---|
| **L1 — Beckn core** | The upstream protocol: 30 endpoints, core data model, `*Attributes` extension slots. ION never modifies this. | `schema/core/v2/api/v2.0.0/beckn.yaml` |
| **L2 — ION network profile** | Network-wide rules: Ed25519 signing, NPWP/NIB mandatory fields, allowed payment rails, data residency Indonesia, 90-day upgrade policy. | `ion.yaml → x-ion-profile block` |
| **L3 — ION endpoint extensions** | 8 endpoints that ION adds to Beckn's 30: the `/raise` family for dispute escalation (6 endpoints) and `/reconcile` + `/on_reconcile` for settlement. | `ion.yaml → paths: block` |
| **L4 — Cross-sector attribute packs** | Fields that apply across every ION sector: Indonesian address format, business identity (NPWP, NIB), payment methods (QRIS, COD, BNPL...), tax (PPN, PPnBM), participant roles, product certifications (halal, BPOM). | `schema/extensions/core/` |
| **L5 — Sector attribute packs** | Fields specific to one sector. Trade packs cover product structure, offers, performance, contracts. Logistics packs cover shipments, rate logic, customs, tracking. | `schema/extensions/trade/` and `schema/extensions/logistics/` |

A concrete example: when a BPP publishes a nasi goreng listing, the catalog message carries:

```
Beckn Resource (L1)
  └── resourceAttributes:
       ├── IONCatalogLocalization  (L4 core/localization)  — name.id: "Nasi Goreng Spesial"
       ├── IONProductCertifications (L4 core/product)     — halalStatus: HALAL
       └── TradeResource  (L5 trade/resource)   — food.classification: HALAL
                                                           — food.spiceLevel: HIGH
                                                           — preparationTime: PT15M
                                                           — availability.status: IN_STOCK
```

---

## What you implement

### If you are a BPP (seller app, merchant platform, LSP)

You implement Beckn's 30 endpoints **plus** ION's 8 extensions. You publish catalogs using the ION schema packs to populate the `*Attributes` fields. You declare policy IRIs from `policies/` on every offer.

The mandatory fields that ION Central will reject if missing:

| Field | Where | Why mandatory |
|---|---|---|
| `npwp` | `participantAttributes` | Indonesian tax law (PMK 112/2022) |
| `nib` | `participantAttributes` | Business registration (PP 5/2021) |
| `halalStatus` | `resourceAttributes` | Halal product law (UU 33/2014) — food and beverage |
| `countryOfOrigin` | `resourceAttributes` | Consumer protection (PerBPOM 31/2018) |
| `ageRestricted` | `resourceAttributes` | All products — determines if age verification needed |
| `name.id` | `resourceAttributes` | Bahasa Indonesia product name — all products |
| `contactDetailsConsumerCare` | `offerAttributes` | Consumer protection (UU 8/1999 Pasal 7) |

### If you are a BAP (buyer app, marketplace, aggregator)

You implement Beckn's 30 endpoints plus the ION extensions that apply to your role. You subscribe to catalogs. You construct `Contract` messages using ION field paths. You render policy terms to consumers.

### If you are building ONIX (the reference implementation)

ONIX handles the transport layer — HTTP signatures, ACK/callback routing, registry integration, `/publish_catalog`, `/subscribe`, schema validation, error formatting. You build on top of it.

---

## Transaction lifecycle

A standard B2C purchase on ION:

```
Phase 1 — Catalog (async, BPP pushes, BAP subscribes)
  BPP publishes catalog → ION Catalogue Service
  BAP subscribes → receives /on_discover callbacks

Phase 2 — Transaction (direct, BAP ↔ BPP)
  BAP → /select   → BPP → /on_select    (pick items, get quote)
  BAP → /init     → BPP → /on_init      (delivery address, payment method)
  BAP → /confirm  → BPP → /on_confirm   (binding order, contract created)
  BPP → /on_status (×N, unsolicited)    (PACKED → DISPATCHED → DELIVERED)

Phase 3 — Post-fulfilment
  BAP → /rate     → BPP → /on_rate      (ratings)
  BAP → /reconcile → BPP → /on_reconcile  (ION extension: settle financials)
  BAP or BPP → /raise → ION             (ION extension: escalate if needed)
```

See `flows/trade/patterns/storefront/v1/pattern.yaml` for the full field-by-field breakdown of every step.

---

## Start here by role

**I'm new to ION and need to understand where my business fits:**
1. Read `docs/ION_Bu_Sari_Wizard.md` — a plain-language walkthrough of a pharmacy joining ION. All hierarchy layers in practice.
2. Read `docs/ION_Entity_Hierarchy_Model.md` — the formal model: sectors, categories, CRCs, patterns
3. Read `docs/ION_Resource_Categories.md` — find the right category code for your product or service
4. Read this README fully
5. Read `schema/core/v2/api/v2.0.0/README.md` — the two-file spec model explained

**I'm a developer building a BPP or BAP:**
1. Read this README fully
2. Read `schema/core/v2/api/v2.0.0/README.md` — the two-file model explained
3. Read `schema/extensions/README.md` — how attribute packs work
4. Pick your commerce pattern: `flows/trade/README.md` or `flows/logistics/README.md`
5. Open the pattern for your pattern and read it step by step

**I already know Beckn:**
1. Read `schema/core/v2/api/v2.0.0/README.md`
2. Read `schema/extensions/core/README.md` — what L4 adds to Beckn
3. Pick your sector: `schema/extensions/trade/README.md` or `schema/extensions/logistics/README.md`

**I want to contribute a new schema field or product example:**
1. Read `docs/ION_Schema_Style_Guide.md`
2. Read `docs/ION_Bu_Sari_Complete_Journey.md` to understand what real payloads look like
3. Find the right pack in `schema/extensions/`
4. Follow the contribution workflow in `CONTRIBUTING.md`

**I want to understand a specific error:**
1. Look up the error code in `errors/registry.json`
2. The entry references the schema field and the flow step where it applies

---

## Active sectors

| Sector | Covers | Schema | Flows |
|---|---|---|---|
| **Trade** | B2C, B2B, marketplace, subscription, auction, cross-border, government procurement | `schema/extensions/trade/` | `flows/trade/` |
| **Logistics** | Hyperlocal, parcel, freight, Ro-Ro, cross-border, warehousing | `schema/extensions/logistics/` | `flows/logistics/` |
| **Hospitality** | Food delivery and online ordering (`HSC-delivery`) — partial activation | `schema/extensions/hospitality/` | `flows/hospitality/` |

Mobility, finance, tourism, and healthcare are reserved. Other Hospitality CRCs (accommodation, restaurant table, events, wellness) are reserved until their working groups ratify.

**Food classification rule:** Restaurant meals and food delivery apps (GoFood, GrabFood, any prepared-to-order food) → **Hospitality** (`HSC-delivery`, `delivery-order` pattern). Packaged food products sold on e-commerce (Indomie, bottled water, protein powder) → **Trade** (`TRC-food-bev`, `storefront` pattern). The test: is the item made fresh after the order? If yes → Hospitality. Their folder stubs exist; content will be added when their working groups ratify.

---

## Policy registry

Sellers declare policy intent using IRIs — compact identifiers that resolve to ratified terms documents. ION Central validates and enforces them.

```yaml
# Example: a trade offer's policy declarations
offerAttributes:
  returnPolicy:       ion://policy/return.standard.7d-sellerpays
  cancellationPolicy: ion://policy/cancel.prepacked.free
  warrantyPolicy:     ion://policy/warranty.manufacturer.1y-distance-service
  disputePolicy:      ion://policy/dispute.consumer.bpsk
  grievanceSlaPolicy: ion://policy/grievance-sla.consumer.standard
  paymentTermsPolicy: ion://policy/payment-terms.upfront.full
```

Each IRI resolves to a YAML document in `policies/` that defines the window, fee structure, and enforcement behaviour. ION Central rejects catalogs with unknown IRIs and rejects runtime actions that violate declared terms. See `policies/README.md`.

---

## Versioning

ION versions follow `MAJOR.MINOR.PATCH-STAGE`. Implementations receive **90 days' notice** before any mandatory upgrade. See `GOVERNANCE.md`.

Current: **v0.6.0** — targeting Beckn `core-v2.0.0-rc1`. See [CHANGELOG.md](CHANGELOG.md) for what is in this release.

---

*ION Network Specification — Indonesia Open Network*
