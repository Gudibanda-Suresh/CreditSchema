# ION Schema — Open Observations

This document explains the schema issues that have not yet been fixed, why they exist, what the problem actually is, and what fixing them would require. It is written for anyone reading the codebase who wants to understand the current state without needing deep technical background.

There are four open observations. Two are deferred to v0.6.0 because fixing them is a breaking change. Two are partial because the core issue has been addressed but a complete resolution requires a governance decision.

---

## M1 — Schema class names carry a redundant "Attributes" suffix

**Status:** Deferred to v0.6.0 (breaking change)

### What the problem is

Every ION schema extension pack defines a class — a named blueprint for the data it describes. Right now, every single one of those class names ends with the word "Attributes":

```
TradeResource
TradeOffer
LogisticsOffer
LogisticsShipment
RestaurantMenuItem
IONParticipant
... (19 classes total)
```

The Beckn Schema Design Guide says this is an anti-pattern and explicitly prohibits it. The reason: the name of a concept should describe **what the thing is**, not **where it sits in a message**. "Attributes" is a container word — it describes the slot in the message that holds the data, not the data itself.

### A plain-language analogy

Imagine you are filling in a form at a hospital. The form has a section called "Patient Details". You write your name as:

> **Name:** "John Smith Patient Details"

The words "Patient Details" are the name of the section on the form. Your actual name is just "John Smith". Putting the section name inside your name is redundant and confusing — anyone looking at your file now has to mentally strip the section name every time they read your name.

That is exactly what `TradeResource` does. The real concept is a **trade product** (or a **trade resource**). "Attributes" is just the name of the slot in the Beckn message where this data sits. It adds noise without adding meaning.

### What the names should be

The guide gives a concrete rename table. For ION, the equivalent would be:

| Current name | What it should be | Why |
|---|---|---|
| `TradeResource` | `TradeProduct` | It describes a product in the trade sector |
| `TradeOffer` | `TradeOffer` | It describes commercial offer terms |
| `TradeContract` | `TradeContract` | It describes a trade contract |
| `TradePerformance` | `TradePerformance` | It describes fulfilment performance |
| `TradeProvider` | `TradeProvider` | It describes a trade seller/merchant |
| `TradeCommitment` | `TradeCommitment` | It describes a line item commitment |
| `TradeConsideration` | `TradeConsideration` | It describes pricing and breakup |
| `LogisticsShipment` | `Shipment` | It describes a shipment |
| `LogisticsOffer` | `LogisticsOffer` | It describes a logistics service offer |
| `LogisticsContract` | `LogisticsContract` | It describes a logistics contract |
| `LogisticsPerformance` | `LogisticsPerformance` | It describes delivery performance |
| `LogisticsProvider` | `LogisticsProvider` | It describes a logistics provider |
| `LogisticsCommitment` | `LogisticsCommitment` | It describes a cargo commitment |
| `LogisticsConsideration` | `LogisticsConsideration` | It describes freight pricing |
| `LogisticsTracking` | `LogisticsTracking` | It describes shipment tracking |
| `IONParticipant` | `Participant` | It describes a network participant |
| `IONRating` | `Rating` | It describes a rating submission |
| `IONReconciliation` | `Reconciliation` | It describes a reconciliation record |
| `RestaurantMenuItem` | `RestaurantMenuItem` | It describes a food delivery menu item |

### Where this matters in practice

The class name appears on the wire in every single message, inside the `@type` field:

```json
"resourceAttributes": {
  "@type": "ion:TradeResource",
  ...
}
```

An AI agent reading this message sees `ion:TradeResource` and has to decode: "OK, 'Trade' means this is a trade-sector thing, 'Resource' means it's a product, 'Attributes' means... the attributes of that product? Or the attributes slot? Or both?" The word "Attributes" creates ambiguity that adds no information.

With the corrected name:

```json
"resourceAttributes": {
  "@type": "ion:TradeProduct",
  ...
}
```

Now it is unambiguous. "This is a trade product." Done.

### Why it has not been fixed yet

Changing a class name is a **breaking change**. Every buyer app and seller app that currently reads ION messages checks the `@type` field. If `ion:TradeResource` becomes `ion:TradeProduct` overnight, every implementation that was looking for the old name stops working.

The correct process is:

1. Publish the new names in v0.6.0
2. For 90 days (ION's mandatory upgrade notice period), both the old name and the new name are accepted
3. Each old name gets an `owl:deprecated` and `owl:sameAs` annotation in the vocabulary file, pointing to the new name, so that any agent that knows about ontology relationships automatically understands they are the same thing
4. After 90 days, the old names are retired

This is tracked for v0.6.0.

---

## M4 — Some schema packs nest data too deeply

**Status:** Deferred to v0.6.0 (requires field-by-field regulatory analysis)

### What the problem is

The Beckn Schema Design Guide says schemas should be as flat as possible. Deep nesting — data stored inside objects inside objects inside objects — makes it harder for AI agents to understand the meaning of each field, because the field is surrounded by structural wrapper layers rather than sitting in clear context.

The guide sets a practical test: *if an agent read a well-written description and looked at product images, would it still need this field in structured form?* If not, the field probably belongs in plain text (`Descriptor.longDesc`), not in a nested structure.

Right now, the ION trade/resource pack reaches nesting depth 13, and the hospitality food delivery pack reaches the same. For comparison, a simple flat list of fields has depth 1.

### A plain-language analogy

Imagine a recipe card. There are two ways to write it.

**Deep nesting version:**
```
Recipe:
  FoodInfo:
    NutritionalData:
      PerServing:
        Macronutrients:
          Protein:
            Amount:
              Value: 25
              Unit: grams
```

To find out the protein content, you have to open six boxes, one inside another. Each box adds a layer you have to mentally process before reaching the actual number.

**Flat version:**
```
proteinPerServingGrams: 25
```

Same information. One step. An agent reading the flat version immediately knows: "25 grams of protein per serving." The nested version requires the agent to reconstruct that meaning from six layers of structure.

### Where this is happening in ION

**trade/resource/v1 — the `food` sub-object:**

```yaml
food:
  type: object
  properties:
    classification:       # depth 1 inside food
      type: string
    allergens:            # depth 1 inside food
      type: array
      items:
        type: string      # depth 2 — fine
    nutritionalContent:   # depth 1 inside food
      type: object
      properties:
        nutrients:        # depth 2 inside food
          type: array
          items:
            type: object  # depth 3 — each nutrient is itself an object
            properties:
              name:       # depth 4
              amount:     # depth 4
                type: object
                properties:
                  value:  # depth 5 ← the actual number is 5 levels down
                  unit:   # depth 5
              dailyValuePct:  # depth 4
```

To find the protein amount, a machine (or a human) has to navigate: `food → nutritionalContent → nutrients[n] → amount → value`. Five levels deep for a single number.

The guide says: a multi-paragraph `longDesc` field reading "Each serving contains 25g protein, 18g fat, 45g carbohydrate, 450 calories" gives an AI agent more usable information than this entire nested structure — and is readable by a human in a fraction of a second.

**hospitality/delivery — customisation groups:**

The food delivery pack lets restaurants define customisation options (size, spice level, toppings). The menu option labels go 13 levels deep:

```
customisationGroups          ← level 1
  options                    ← level 2
    label                    ← level 3
      id                     ← level 4 (the actual Bahasa text)
```

The actual Indonesian label text — four words like "Extra Pedas" — sits at depth 13 in the full schema tree. An agent trying to display this to a consumer has to navigate 13 levels of structure to find it.

### Why not all nesting is a problem

The guide is explicit: not everything should be flattened. Fields that participate in **arithmetic** (price, weight for shipping cost calculation), **filtering** (halal status, age restriction for consumer safety), or **regulatory compliance** (BPOM registration number for legal traceability) — these must stay structured because machines need to process them precisely.

The fields that should be flattened are purely **descriptive** ones: nutritional labels, preparation notes, colour descriptions, feature lists. These are things a human would write in a product description, and an AI agent reads them best as natural prose anyway.

### Why it has not been fixed yet

Every field in these nested structures needs to be evaluated individually:

- Does this field participate in arithmetic, filtering, or compliance checking? → **Keep it structured.**
- Is it purely descriptive? → **Move to `longDesc` or flatten.**

With 1,487 lines in trade/resource alone, this is a significant analysis task. Doing it correctly requires input from the ION product and legal teams (to confirm which fields are genuinely regulatory) and from the implementation teams (to confirm which fields apps actually query programmatically). Getting it wrong — flattening a field that turns out to be needed for filtering — would be another breaking change.

This is tracked for a dedicated v0.6.0 schema refactoring cycle.

---

## C2 — Some schema packs declare mandatory fields that could break valid implementations

**Status:** Partial — documented, full resolution is a governance task

### What the problem is

In schema design there is an important distinction between two kinds of "required":

- **Network-mandatory:** This field is required by Indonesian law or ION network rules. Every participant on the ION network must provide it. This is a policy decision enforced at the network level.
- **Schema-mandatory:** This field is structurally required in the data format. Any message without it fails schema validation regardless of context.

The Beckn guide says: shared schemas should not declare fields as structurally mandatory by default. The reason is that a schema is meant to describe what *can* exist in a message — not what *must* exist in every possible context. Different networks using the same schema may have different requirements.

The issue in ION is that some packs declare required fields directly in the schema (`required:` block), when those requirements are actually ION-specific network rules, not universal data structure requirements.

### A plain-language analogy

Imagine a standard form template for a business contract. The template says:

> Field marked with * are **required**.  
> **NPWP*** (Indonesian tax number)  
> **NIB*** (Business registration number)

This is fine if the template is only ever used in Indonesia. But if someone in Singapore tries to use this template for a contract with an Indonesian company, they cannot fill in an Indonesian tax number — they do not have one. The template rejects their submission because a field it calls "required" is not applicable to their situation.

The template should say: "NPWP is required when the participant is an Indonesian registered business." That is a network rule, not a structural rule about the form itself.

### What was done so far

The five packs that still have `required:` blocks now have a comment documenting the regulatory basis:

```yaml
# ION network mandatory: Indonesian business registration requirements 
# (UU 11/2020 Cipta Kerja; PP 5/2021)
required:
  - npwp
  - nib
```

This tells anyone reading the file why those fields are required and that the requirement is an ION/Indonesia-specific rule. It does not fix the underlying structural problem — the fields are still schema-mandatory rather than network-policy-mandatory.

### What full resolution requires

The correct approach is to move the mandatory constraint out of the schema's `required:` block and into ION's network policy layer — a separate validation rule that ONIX (the reference implementation) enforces when a message arrives at an ION gateway. The schema would then describe the field as optional, and ONIX would reject messages that are missing it based on the network policy.

This is a governance decision because it requires agreement on which fields are network-mandatory (enforced by ONIX policy) versus which are truly structurally required in all contexts. The ION Council needs to ratify the list before the schema `required:` blocks are changed.

The five packs currently affected: `core/address`, `core/identity`, `core/participant`, `core/payment`, `core/product`.

---

## m2 — Two regulation citations remain inside description text rather than in the dedicated annotation field

**Status:** Partial — acceptable as contextual prose, not a strict violation

### What the problem is

ION uses a dedicated annotation field called `x-ion-regulatory` to record which Indonesian law or regulation makes a particular field required. This keeps regulation citations in a consistent, machine-readable place separate from human-readable descriptions.

For example, the `halalStatus` field in `core/product` is annotated like this:

```yaml
halalStatus:
  type: string
  description: Dietary classification. MANDATORY for all restaurant ordering resources.
  x-ion-regulatory: UU 33/2014 tentang Jaminan Produk Halal
```

The regulation is in `x-ion-regulatory`. The description is in `description`. They are separate.

Two places in the codebase still have regulation numbers mentioned inside multi-line description text:

**core/participant/v1 — NPWP field description:**
```yaml
description: 'Indonesian Tax Registration Number (NPWP) — 16-digit format per PMK 112/2022). 
  Some implementations still accept legacy 15-digit format...'
```

**core/reconcile/v1 — tax withholdings description:**
```yaml
description: 'Indonesian tax withholdings applied to this settlement. Marketplaces in Indonesia 
  withhold PPh 22 on certain transactions per PMK 41/2022 (marketplace tax collector appointment).
  PPh 23 applies on service fees...'
```

### Why these two were not fixed

Both of these are regulation numbers appearing **inside a sentence** that explains context — not as standalone citations. The NPWP description is explaining why 16-digit format exists (because of a 2022 regulation that changed it from 15 digits). The reconcile description is explaining *what* PPh 22 is and *when* it applies, as background knowledge for a developer reading the schema.

These are different from the cases that were fixed, where a regulation number was sitting alone in a description as a bare citation with no surrounding context.

The judgment call: removing "per PMK 112/2022" from the NPWP description would make the description less useful — a developer reading it would no longer understand *why* there are two valid formats (old 15-digit and new 16-digit). The regulation number is earning its place in that sentence.

### What a stricter fix would look like

A stricter approach would add `x-ion-regulatory:` annotations to these fields in addition to the inline mentions:

```yaml
npwp:
  type: string
  x-ion-regulatory: PMK 112/2022 (NPWP 16-digit format transition)
  description: 'Indonesian Tax Registration Number (NPWP) — 16-digit format 
    (migrated from 15-digit in 2022). Some implementations still accept legacy format...'
```

This is low priority. Both fields already have the regulation number visible to a reader; the machine-readable `x-ion-regulatory` annotation is additive, not corrective. This can be addressed in the normal course of pack maintenance.

---

## Summary

| Observation | Status | What it needs |
|---|---|---|
| M1 — `*Attributes` class names | Deferred to v0.6.0 | 19 class renames + `owl:deprecated` migration paths + 90-day notice period |
| M4 — Deep nesting | Deferred to v0.6.0 | Field-by-field regulatory analysis + ION Council + implementation team input |
| C2 — required[] in shared schemas | Partial | ION Council governance decision on which fields move to ONIX network policy layer |
| m2 — Inline regulation citations | Partial | Low priority — citations are contextual prose, not standalone violations |

None of these are bugs that break anything today. M1 and M4 are design improvements that will make the schemas more useful for AI agents and more compliant with the Beckn guide. C2 and m2 are correctness issues that matter more when ION schemas are used outside the ION network context.

---

*ION Network Specification — Open Observations Register*  
*Last updated: v0.5.2*
