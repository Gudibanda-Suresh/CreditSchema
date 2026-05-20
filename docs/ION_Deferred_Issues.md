# ION Schema — Deferred Issues (v0.5.2)

Two issues from the Beckn Schema Design Guide review are deferred to v0.6.0. They are not bugs that break anything today — the schemas work correctly as written. They are design pattern violations that matter for long-term interoperability, particularly as AI agents and third-party tools begin consuming ION schemas.

This document explains what each issue is, why it exists, what it means in practice, and what fixing it will require.

---

## M1 — Schema class names carry the `Attributes` suffix

**Status:** Deferred to v0.6.0  
**Severity:** Major (design pattern, no runtime impact today)  
**Affects:** 19 schema classes across all ION extension packs

### What the issue is

Every ION schema class is named after the Beckn container slot it lives in, rather than the real-world concept it describes. The result is names like `TradeResource`, `LogisticsOffer`, and `FnbMenuItem`.

The Beckn Schema Design Guide explicitly prohibits this pattern. It calls it out by name as an anti-pattern and requires that the class name describe **what the thing is**, not **where the thing is stored**.

### The full list of affected names

| Current name (violating) | What it should be called | What it actually describes |
|---|---|---|
| `TradeResource` | `TradeProduct` | A product listed for sale on the ION Trade network |
| `TradeOffer` | `TradeOffer` | The terms of sale for a product (returns, cancellation, warranty) |
| `TradeContract` | `TradeContract` | An agreed purchase order between buyer and seller |
| `TradeCommitment` | `TradeCommitment` | A specific line item within a contract |
| `TradeConsideration` | `TradeConsideration` | The pricing breakdown (taxes, discounts, fees) |
| `TradePerformance` | `TradePerformance` | The fulfilment record (delivery status, agent, proof) |
| `TradeProvider` | `TradeProvider` | A registered merchant or seller on ION Trade |
| `LogisticsShipment` | `LogisticsShipment` | A shipment being transported |
| `LogisticsOffer` | `LogisticsOffer` | Service terms for a logistics provider |
| `LogisticsContract` | `LogisticsContract` | An agreed logistics engagement |
| `LogisticsCommitment` | `LogisticsCommitment` | A specific shipment line within a logistics contract |
| `LogisticsConsideration` | `LogisticsConsideration` | Freight pricing breakdown (base rate, surcharges, fuel) |
| `LogisticsPerformance` | `LogisticsPerformance` | Live shipment tracking and fulfilment events |
| `LogisticsProvider` | `LogisticsProvider` | A registered carrier or logistics operator |
| `LogisticsTracking` | `LogisticsTracking` | Real-time shipment tracking configuration |
| `IONParticipant` | `IONParticipant` | A registered network participant (buyer, seller, LSP) |
| `IONRating` | `IONRating` | A rating submitted for a product, provider, or delivery |
| `IONReconciliation` | `IONReconciliation` | Financial settlement between BAP and BPP |
| `FnbMenuItem` | `FnbMenuItem` | A prepared food or beverage item available for delivery |

### Why it matters — explained without jargon

Think of a class name as a label on a filing cabinet. The label should say what's inside — **"Customer Orders"**, **"Product Listings"**, **"Delivery Records"**. It should not say where the cabinet sits — **"Third-Floor East-Wing Cabinet Attributes"**.

Right now, ION's schema class names are doing the second thing. `TradeResource` tells you that this is "Attributes that go into the Resource slot in the Trade sector." It does not tell you that this is a **product listing** — which is what it actually is.

This matters for two specific reasons:

**1. AI agents cannot identify the concept without reading the schema contents.**

When an AI agent or a tool on another Beckn network encounters `ion:TradeResource`, it reads the name and tries to understand what it is. The word "Attributes" tells it nothing — every extension pack has the word "Attributes" in its name. The agent has no starting point. It must read the entire schema before it can form even a basic understanding.

If the name were `ion:TradeProduct`, the agent knows immediately: this is a product in a trade context. It can connect that to everything it already knows about products — from schema.org's `Product`, from GS1's product vocabulary, from its own training on product catalogs worldwide. It starts with context rather than from zero.

**2. The class name leaks into the permanent public identifier (IRI) for the schema.**

Every ION schema class has a permanent identifier that looks like this:

```
https://schema.ion.id/trade/resource/v1#TradeResource
```

This IRI is what gets written into every payload that uses this schema, on the wire, across the network, between every BAP and BPP in Indonesia. It is baked into the `@type` field of every catalog message:

```json
{
  "resourceAttributes": {
    "@context": "https://schema.ion.id/trade/resource/v1/context.jsonld",
    "@type": "ion:TradeResource",
    ...
  }
}
```

**IRIs are permanent.** Once a network goes live and thousands of transactions carry `@type: ion:TradeResource`, that string cannot simply be changed. Any system — any BAP, any BPP, any archive, any audit trail — that has stored or processed a payload with that `@type` will break if the name changes without a migration path.

This is why fixing M1 is not just a find-and-replace. It requires:
1. Choosing the new names
2. Declaring the old names as deprecated aliases in every `vocab.jsonld` file using `owl:sameAs` and `owl:deprecated`
3. Announcing a migration window to all implementers (90 days minimum per ION governance policy)
4. Updating all ION-maintained tools (ONIX validator, registry) to accept both names during the transition

### A concrete before-and-after

Here is what the class definition looks like today in `trade/offer/v1/attributes.yaml`:

```yaml
TradeOffer:            # ← The class name
  type: object
  title: Trade Offer Attributes
  x-jsonld:
    '@context': 'https://schema.ion.id/trade/offer/v1/context.jsonld'
    '@type': ion:TradeOffer   # ← This goes on the wire in every payload
  x-beckn-attaches-to: Offer.offerAttributes
```

Here is what it should look like after the fix:

```yaml
TradeOffer:                      # ← Name reflects what the thing IS
  type: object
  title: ION Trade Offer
  x-jsonld:
    '@context': 'https://schema.ion.id/trade/offer/v1/context.jsonld'
    '@type': ion:TradeOffer      # ← Clean, meaningful IRI on the wire
  x-beckn-attaches-to: Offer.offerAttributes
  x-recommended-parent: Offer    # ← Where it lives, without polluting the name
```

And in `vocab.jsonld`, the migration bridge:

```json
{
  "@id": "ion:TradeOffer",
  "@type": "rdfs:Class",
  "rdfs:label": "TradeOffer",
  "rdfs:subClassOf": { "@id": "beckn:Offer" },
  "owl:sameAs": { "@id": "ion:TradeOffer" },
  "skos:note": "Previously named TradeOffer. The old name remains valid as an alias during the v0.5 → v0.6 migration window."
}
```

The `owl:sameAs` line is the safety net. It tells every JSON-LD processor: "if you see `ion:TradeOffer` in an old payload, treat it as identical to `ion:TradeOffer`." Both names work during the transition. Old payloads are not invalidated.

### Why it is deferred and not fixed now

The rename itself takes minutes. The reason it is deferred is the IRI permanence problem described above.

ION is at v0.5.2-draft — no production traffic exists yet. This is actually the **best possible moment** to do this fix, before any live transactions have baked the old names into audit trails. The deferral to v0.6.0 is not because the fix is risky; it is to ensure it is done with the right process (ION Council sign-off on the new names, coordinated communication to pre-production integrators) rather than as a silent change.

---

## M4 — Deeply nested schema structures

**Status:** Deferred to v0.6.0  
**Severity:** Major (agent-readability, no runtime impact today)  
**Affects:** 13 schema packs, worst cases reaching depth 13

### What the issue is

Several ION schemas define data structures that are nested many levels deep — objects inside objects inside objects. The Beckn Schema Design Guide's "Agent-First Design" principle says this fragmentation actively reduces how well AI agents can understand schema content, because it breaks apart semantic meaning that belongs together.

The issue is not that deep structures are forbidden. The issue is that many of the deeply nested fields in ION schemas contain **descriptive information** that an AI agent would understand better as natural-language prose in the `Descriptor.longDesc` field — while a small number of other deeply nested fields contain **regulatory or machine-comparable data** that genuinely need to be structured.

The guide provides a clear test for each field: *"Would a regulator, an auditor, or the network's settlement layer object if this fact were missing?"* If yes — keep it structured. If no — move it to prose.

### What depth means in practice

YAML and JSON schemas nest by indentation. Each level of nesting in the schema adds one level to the data structure in every message that carries it. Here is what depth 13 actually looks like on the wire — a real example from `trade/resource/v1`:

```
message
  └── catalog
        └── resources[]
              └── resourceAttributes          (depth 3 — the extension bag)
                    └── food                  (depth 4 — food sub-object)
                          └── nutrition       (depth 5 — nutrition sub-object)
                                └── vitamins  (depth 6 — vitamins sub-object)
                                      └── []  (depth 7 — array of items)
                                            └── name        (depth 8)
                                                  └── id    (depth 9 — Bahasa Indonesia name)
```

An AI agent reading a product payload for a bag of rice has to reconstruct the sentence *"contains 12% daily value of Vitamin B1"* by combining values from:
- `resourceAttributes.food.nutrition.vitamins[0].name.id` = "Vitamin B1"
- `resourceAttributes.food.nutrition.vitamins[0].dailyValuePct` = 0.12

These two facts are nine levels deep and stored in separate fields. Compare that to writing them once in `Descriptor.longDesc`:

```
"Kandungan gizi per sajian: Energi 360 kkal, Protein 7g, Lemak 1g, 
Karbohidrat 78g. Vitamin B1 12% AKG, Vitamin B3 8% AKG, Besi 6% AKG."
```

An AI agent reading that sentence immediately understands all seven nutritional facts simultaneously, in their natural relationship to each other.

### The four problem areas

**1. `trade/resource/v1` — product detail sub-objects (max depth: 13)**

The `food`, `fashion`, `electronics`, `pharmacy`, and `beauty` sub-objects each contain their own nested sub-objects. Some of these — halal status, BPOM registration number, prescription flag — are genuine regulatory facts that must remain structured. Others — spice level, style descriptions, allergen names — are descriptive content that belongs in `Descriptor.longDesc`.

```yaml
# Current — depth 8, descriptive not regulatory:
food:
  spiceLevel:
    type: string
    enum: [NONE, LOW, MEDIUM, HIGH, EXTRA_HOT]

# Better for agent comprehension:
# In Descriptor.longDesc: "Tingkat kepedasan: sedang. Mengandung cabai rawit."
```

**2. `hospitality/delivery/v1` — customisation group options (max depth: 13)**

The customisation groups for food ordering (size, toppings, spice level choices) reach depth 13 when a `customisationGroup` contains `options[]` which contain `priceDelta` which contains `value`. This depth is structurally necessary — the customisation model is a working interactive feature for restaurant ordering, not descriptive text. This is the least clear-cut case for flattening.

**3. `logistics/contract/v1` — dispute state machine (max depth: 9)**

The contract schema encodes a dispute resolution state machine as a nested property tree: `disputeReason → disputeDetail → claimDetail → claimAssessment → claimResolution`. This is five parallel nested objects, each with their own fields, tracking the lifecycle of a weight dispute or damage claim.

```yaml
# Current — a state machine embedded as nested objects:
disputeReason:
  type: object
  properties:
    policyIri:
      type: string
    description:
      type: string
disputeState:
  type: string
  enum: [DISPUTE_WEIGHT_RAISED, DISPUTE_EVIDENCE_EXCHANGED, ...]
```

The guide's suggestion for this pattern: extract it as a separate `LogisticsDisputeAttributes` pack that can version independently as dispute resolution rules evolve, rather than embedding it as a sub-tree inside the contract schema.

**4. `core/support/v1` — issue action trail (max depth: 11)**

The support schema tracks a full issue ticket thread — complainant actions, respondent actions, escalation events — as deeply nested arrays. This is appropriate for a ticket management system but reaches depth 11 in the schema definition.

### Why it is deferred and not just fixed

Unlike M1, there is no simple rename here. Flattening a nested structure requires deciding, for each nested field, whether it moves to `Descriptor.longDesc` (prose) or stays structured. That decision requires domain judgment:

- Does this field participate in filtering? (a buyer app lets users filter by spice level → needs to stay structured)
- Does this field participate in arithmetic? (price delta in customisation groups → needs to stay structured)
- Does a regulator require this field to be machine-comparable? (halal status → needs to stay structured)
- Is this purely descriptive? (ingredient descriptions, style notes, usage instructions → belongs in `Descriptor.longDesc`)

The `trade/resource` pack alone has over 200 fields across its category sub-objects. Making the wrong judgment on any one field could change what buyer apps can filter on. This is a schema working group task — it requires ION's Trade Sector Working Group to review every field in the affected packs against the three questions above, not a change that should be made by a single author.

The target for v0.6.0 is to produce a reviewed field-by-field classification table and implement the agreed changes in a single coordinated release, with clear documentation of which fields moved where and why.

---

## Summary

| Issue | What breaks today | What breaks at scale | Fix complexity |
|---|---|---|---|
| **M1** — `*Attributes` suffix | Nothing — schemas work correctly | AI agents on counterparty networks cannot identify concepts by name; IRI names are permanently suboptimal | Medium — rename + migration bridges + governance process |
| **M4** — Deep nesting | Nothing — data is correctly transmitted | AI agents must reconstruct semantic meaning from fragmented leaves; descriptive content is harder to process than prose | High — field-by-field regulatory test across 200+ fields in 13 packs |

Both issues are v0.6.0 targets. The ION Network is at v0.5.2-draft with no production traffic, which means the window to fix M1 cleanly (before live IRIs are permanent) is still open. M4 is the longer-term schema maturation work that the working group will drive.

---

*ION Network Specification — v0.5.2-draft*
