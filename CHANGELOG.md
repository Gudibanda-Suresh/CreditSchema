# ION Network Specification — Changelog

## v0.6.1 — May 2026

**Logistics patch — multi-stop P2P variant.**

This is a non-breaking additive patch to the logistics sector. No existing payloads, patterns, or schema packs are modified in a breaking way. All changes are additions.

### What changed

#### New variant: `flows/logistics/variants/multi-stop/v1/`

A new variant covering single-booking, single-rider (or single-AWB) deliveries that visit one pickup stop and two or more drop stops in declared sequence. Applies to `parcel` (with `routingTopology: P2P_MULTI_STOP`) and `hyperlocal`. Key fields introduced: `performanceAttributes.sequence`, `performanceAttributes.type` (PICKUP | DROP), `performanceAttributes.packageIds[]`, `performanceAttributes.verification`, `performanceAttributes.codAmount` per stop, `performanceAttributes.agentDetails` (pre-assigned rider), `performanceAttributes.executionLogs[]` (order log), consideration breakup type `MULTI_STOP_FEE`, and `breakup[].stopRef` for per-stop COD fees.

#### Modified: `flows/logistics/patterns/parcel/v1/pattern.yaml`

- `P2P_MULTI_STOP` added as a named `routingTopology` value in the phase 3 state machine description alongside the existing `P2P`, `P2H2P`, `P2H2H2P`.
- `multi-stop` added to `variantWindows` as the earliest-to-latest activation window (`init` through `on_status[DELIVERED]`).
- `MULTI_STOP_FEE` and `COD_FEE.stopRef` added to the `on_select` `conditionalFields` list.

#### Modified: `flows/logistics/patterns/hyperlocal/v1/pattern.yaml`

- `multi-stop` added to `variantWindows`.

#### New examples: `flows/logistics/patterns/parcel/v1/examples/`

- `init-multi-stop.json` — conformant `init` request for a three-stop P2P booking (Mumbai → Pune → Bengaluru) using ION schema URLs and `ion:` type prefixes.
- `on-confirm-multi-stop.json` — conformant `on_confirm` callback with AWB, pre-assigned agent, per-stop verification credentials (OTP/barcode), tracking URL, execution logs, and full tariff breakup including `MULTI_STOP_FEE` and per-stop `COD_FEE` lines.

#### Modified: `profile.json` in parcel and hyperlocal patterns

- `specVersion` bumped from `0.5.1-draft` to `0.6.1`.

---

## v0.6.0 — May 2026

**Initial published release.**

This is the first version of the ION Network Specification made available externally. It covers two active sectors — Trade and Logistics — and provides everything needed to build a conformant network participant.

---

### What is in this release

#### Protocol specification

The specification lives in two files at `schema/core/v2/api/v2.0.0/`:

- **`beckn.yaml`** — Beckn Protocol v2.0.0, vendored and pinned to `core-v2.0.0-rc1`. Defines 30 API endpoints and the core data model. Annotated with upgrade instructions. Do not edit.
- **`ion.yaml`** — The complete ION extension specification in a single OpenAPI 3.1.1 file. Contains:
  - **L2** — ION network profile: Ed25519 signing rules, Indonesian regulatory requirements (NPWP, NIB, halal, BPOM), allowed payment rails, conformance matrix, 90-day upgrade policy
  - **L3** — 8 ION-only endpoints: `/raise`, `/on_raise`, `/raise_status`, `/on_raise_status`, `/raise_details`, `/on_raise_details`, `/reconcile`, `/on_reconcile`
  - **L4** — 11 cross-sector attribute pack schemas (50 schemas total, 42 `$ref` links to `beckn.yaml`)
  - **L5** — 18 sector attribute pack schemas (trade: 7 packs, logistics: 10 packs, plus 1 logistics participant addendum)
  - **`x-ion-conditional-rules`** — 5 cross-schema conditional requirements enforced by ONIX at runtime

All `$ref` links in `ion.yaml` point to the local `beckn.yaml`. No external URLs.

#### Schema extension packs

27 attribute packs across three layers, each with `attributes.yaml`, `schema.json`, `context.jsonld`, `vocab.jsonld`, `profile.json`, `renderer.json`, `README.md`, `docs/`, and `examples/`.

**Core packs (L4) — 11 packs, apply to every sector:**

| Pack | Beckn attachment | What it covers |
|---|---|---|
| `core/address/v1` | Address | Indonesian address hierarchy: 34 BPS province codes, kabupaten, kelurahan, kecamatan, RT/RW, postal code, address type, landmark |
| `core/identity/v1` | Provider.providerAttributes | NPWP (16-digit), NIB (13-digit), NIK, PKP status, legal entity name, business type (PT/CV/UD/PERORANGAN) |
| `core/localization/v1` | Resource.resourceAttributes | LocalisedText pattern — name/shortDesc/longDesc as language-keyed objects (id, en, ms, zh, ar, ta, ko, ja) |
| `core/participant/v1` | Participant.participantAttributes | 20-role taxonomy, NPWP/NIB/NIK/KTP/passport, authorised signatory, full Indonesian address with RT/RW/patokan/accessibility |
| `core/payment/v1` | Settlement.settlementAttributes | PaymentDeclaration + QRIS, VirtualAccount, EWallet (16 providers), CashOnDelivery, BankTransfer, BISettlement (BI_FAST/RTGS/SKN), BNPL (12 providers), CardPayment, CreditTerms, Refund, InvoiceReference, ProcessingFee |
| `core/product/v1` | Resource.resourceAttributes | halalStatus (MUI/BPJPH), halalCertNumber, BPOM registration, SNI certification, SP-PIRT, ageRestricted, minAge |
| `core/raise/v1` | ION raise channel | ION issue tickets: type (11 values), priority, bilateral attempt log, evidence, thread, SLA, resolution |
| `core/rating/v1` | RatingInput.target.targetAttributes | Rating category (PROVIDER/ITEM/FULFILLMENT/AGENT), value 1–5, photo/video reviews, verified purchase, seller response |
| `core/reconcile/v1` | Settlement.settlementAttributes | reconId, settlement basis, amounts (base/finderFee/withholding), adjustments, tax withholdings (PPh22/PPh23/PPN), COD remittance batch, batch reconcile, clawbacks |
| `core/support/v1` | Support.channels[*] | Consumer complaint: category, complainant info, issue action trail (complainant + respondent), escalation levels (SELLER→BPSK→KOMINFO→ION) |
| `core/tax/v1` | Consideration.considerationAttributes | Tax regime (PPN/PPnBM/PPh22/PPh23/EXEMPT), tax category (BKP/JKP/NON_BKP), rate, amount, taxIncluded, eFakturRef, taxBaseAmount |

**Trade packs (L5) — 7 packs:**

| Pack | Beckn attachment | What it covers |
|---|---|---|
| `trade/commitment/v1` | Commitment.commitmentAttributes | lineId, resourceId, offerId, quantity, locked price, customisation selections, special instructions |
| `trade/consideration/v1` | Consideration.considerationAttributes | PPN/PPnBM rates, discount type/value, 31 breakup line types (ITEM, DELIVERY, TAX, COD_FEE, STREAMER_COMMISSION, PPH_WITHHOLDING, PAYMENT_MILESTONE, and more) |
| `trade/contract/v1` | Contract.contractAttributes | Fulfilment location, buyer instructions, invoice preferences (Faktur Pajak), credit terms, subscription billing, live commerce context, gifting details, payment schedule milestones, refund destination |
| `trade/offer/v1` | Offer.offerAttributes | 10 policy IRI fields, COD eligibility, return evidence requirements, flash sale mechanics, stock reservation, voucher stacking, payment method eligibility matrix, price change schedule |
| `trade/performance/v1` | Performance.performanceAttributes | performanceMode, SLA, agent identity, AWB, tracking URL, delivery OTP, proof of delivery, QC result, age verification at delivery, SLA breach tracking, shipping insurance |
| `trade/provider/v1` | Provider.providerAttributes | storeStatus, operating hours, holiday calendar, providerCategory (17 values), KYC lifecycle (status/level/validUntil/caveats), category licences, marketplace-managed logistics, live commerce |
| `trade/resource/v1` | Resource.resourceAttributes | resourceStructure (PLAIN/VARIANT/WITH_EXTRAS/COMPOSED/BUNDLE), resourceTangibility (PHYSICAL/DIGITAL_*), availability signal, variant matrix, customisation groups, category-specific sub-objects for food/fashion/electronics/beauty/agritech/pharmacy/packaged/regulatory, digital goods, image sets, videos, brand identity, geographic indication |

**Logistics packs (L5) — 10 packs:**

| Pack | Beckn attachment | What it covers |
|---|---|---|
| `logistics/commitment/v1` | Commitment.commitmentAttributes | Line identifiers, package count, weight/volume, container references, SKU manifest for warehouse |
| `logistics/consideration/v1` | Consideration.considerationAttributes | 63 breakup line types covering freight, surcharges, COD fees, customs duties, VAS fees, SLA rebates, tax lines |
| `logistics/contract/v1` | Contract.contractAttributes | FWA reference, invoice preferences (Faktur Pajak/Coretax), PPh23 withholding, PPN exemption, stamp duty, eKYC, cargo insurance (ICC A/B/C), customs declaration, INATRADE permits, LARTAS classification, fumigation/phytosanitary certificates |
| `logistics/offer/v1` | Offer.offerAttributes | Service level, transport mode, rate logic (11 types), zone/weight tiers, cut-off schedule (29 cut-off types), AWB allocation model, label formats, state timing commitments, COD fraud controls (OTP, denominations, daily limits, hold period, variance tolerance) |
| `logistics/participant-logistics/v1` | Participant.participantAttributes | PPJK customs-broker licence, driver SIM number and category (A/B_I/B_II/C/D) |
| `logistics/performance/v1` | Performance.performanceAttributes | AWB/BL/booking reference, pickup model and window, applied cut-offs, label bytes, multi-modal legs (pre-carriage/main/on-carriage), cold chain readings, RTS signal and routing, state timing history, sustainability report (GLEC carbon metrics) |
| `logistics/performance-states/v1` | (state machine definitions) | 10 state machine definitions: hyperlocal, parcel P2P/P2H2P/P2H2H2P, freight, warehouse, Ro-Ro, cross-border, reverse |
| `logistics/provider/v1` | Provider.providerAttributes | Catalog versioning, NIB/SIUP/ALFI/ASPERINDO, provider category (CARRIER_DIRECT/3PL/4PL/FORWARDER/AGGREGATOR/MARKETPLACE_LOGISTICS/WAREHOUSE_OPERATOR), patterns supported, modes, coverage, cold-chain/hazmat/high-value capability |
| `logistics/resource/v1` | Resource.resourceAttributes | Shipment object: weight/dimensions, declared value, hazmat (full IATA/IMDG/ADR dataset — class, UN number, packing group, PSN, NEM, segregation, ShippersDeclaration), lithium battery, temperature requirements, HS codes, commercial invoice, certificate of origin (9 preferential forms), packing list, vehicle-as-resource (Ro-Ro) |
| `logistics/tracking/v1` | Tracking.trackingAttributes | Tracking type (LIVE_GPS/AWB_URL/WEBHOOK/WEBSOCKET/POLLING_API), credentials, webhook events, current location, ETA, route polyline, agent details |

#### Transaction flows

**Trade — 12 patterns:**
storefront (reference), made-to-order (made-to-order), subscription (subscription), live-commerce (live/social commerce), digital-goods (digital goods), business-procurement (prepaid), business-procurement (business-credit variant), marketplace-inhouse, marketplace-listed, forward-auction, reverse-auction, XB (cross-border), B2G (government procurement).

**Trade — 6 variants:**
`during-transaction` (10 sub-variants), `cancellation` (4 sub-variants), `returns` (8 sub-variants), `rto` (4 sub-variants), `mid-transaction-changes` (7 sub-variants), `cross-cutting` (5 sub-variants: track, support, rating, reconcile, raise).

**Logistics — 6 patterns:**
hyperlocal, parcel (reference — all other patterns defined as deltas), freight, roro, cross-border (cross-border), warehouse.

**Logistics — 14 variants:**
`attempts`, `cancellation`, `cod`, `cold-chain`, `cross-cutting`, `customs`, `during-transaction`, `ekyc`, `exception`, `incident`, `reverse`, `rts-handoff`, `value-added`, `weight-dispute`.

#### Policy registry

16 categories, 79 ratified policy documents. Sellers declare policy intent as IRIs on offers; ION Central validates and enforces at every API boundary.

**Trade-originated categories:** RETURN (15), CANCELLATION (11), WARRANTY (8), DISPUTE (5), GRIEVANCE_SLA (4), PAYMENT_TERMS (8), PENALTY (19)

**Logistics-originated categories:** EVIDENCE (1), INSURANCE (1), SLA (1), RE_ATTEMPT (1), WEIGHT_DISPUTE (1), LIABILITY (2), INCIDENT (1), RTS_HANDOFF (1), LOGISTICS_FWA (2)

#### Error registry

10 category files, covering ION-1xxx (transport) through ION-9xxx (system). Machine-generated `registry.json` and `logistics-registry.json`. Every error entry is bilingual (id + en) and includes: `affected_field`, `affected_apis`, `schema_ref`, `flow_ref`, `resolution`.

#### Entity hierarchy model and worked examples

`docs/ION_Entity_Hierarchy_Model.md` — the onboarding classification system. Defines:

- **Three planes:** Business/Policy Compliance → Catalogue → Protocol/Technical Compliance
- **6 sectors:** Trade, Hospitality, Logistics, Mobility, Finance, Services
- **38 categories** with KBLI 2025 reference mapping
- **46 Catalogue Resource Categories (CRCs)** with status (ACTIVE/INACTIVE/GOVERNED) and GPC L0 crosswalk for Trade
- **CRC → attribute pack bridge** — each CRC maps to the schema sub-object it enforces, with a concrete before/after example (pharmacy medicine vs mineral water)
- **Patterns and variants** by sector — the transaction blueprint library
- **Path notation:** `sector / category-code / resource-category-code / pattern / variant`

`docs/ION_Resource_Categories.md` — complete CRC reference table. All 46 categories across 6 sectors with description, status, GPC crosswalk, and design notes explaining why categories are merged or kept separate.

`docs/ION_Bu_Sari_Wizard.md` — plain-language walkthrough of a pharmacy seller joining ION. Covers all hierarchy layers (Sector, Segment, CRC, Resource, Pattern, Variant, IGM, Reconciliation) through one seller's journey — the questions she answers, the screens she sees, and what ION derives automatically. Intended for business owners, product managers, regulators, and investors.

`docs/ION_Bu_Sari_Complete_Journey.md` — technical companion to the Wizard. Full API calls and JSON payloads for all 5 product listings, 3 transaction patterns, the IGM complaint flow, and financial reconciliation. Covers both `TRC-health-beauty` (pharmacy sub-object, delivery-time-kyc variant, subscription) and `TRC-food-bev` (food sub-object, allergens, nutrition facts, BPOM ML) in concrete detail.

#### Repository governance

- `beckn.yaml` — vendored with header explaining source, tag, last sync date, and 5-step upgrade process
- `GOVERNANCE.md` — ION Council, versioning policy (MAJOR.MINOR.PATCH-STAGE), 90-day upgrade notice period, extension conventions
- `LICENSE.md` — Apache-2.0
- `CODE_OF_CONDUCT.md` — Contributor Covenant v2.1
- `CONTRIBUTING.md` — contributor workflow, schema pack conventions, `$ref` discipline, naming rules, validation commands
- `docs/ION_Schema_Style_Guide.md` — authoritative pack-authoring guide

---

### Known limitations in this release

- **Production status:** Not live. v0.6.0 is a draft specification. Production launch date is set by ION Council.
- **Beckn upstream:** Targeting `core-v2.0.0-rc1`. ION will re-validate against final Beckn v2.0.0 and publish a corrected release if any wire-level changes are required.
- **Sectors:** Trade and Logistics are active. Hospitality, Mobility, Finance, and Services have reserved folder stubs — content will be published as their working groups ratify.
- **ONIX:** Reference implementation is in development. Available via ION DevLabs when ready.
- **KBLI migration:** KBLI 2020 → 2025 migration deadline is 18 June 2026 per Indonesian regulation. The entity hierarchy model references KBLI 2025 codes.

---

*ION Network — Indonesia Open Network. Built on Beckn Protocol v2.0.0.*
