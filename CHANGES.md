# Credit Schema v2 — Change Log

**Schema:** `credit_schema_v2` → `credit_schema_v2_fixed`
**Validated against:** Schema Design Guide NFH-012 (Beckn Protocol, Networks for Humanity Foundation)
**Date:** 2026-05-17

---

## Background

This document records every change made to the original `credit_schema_v2` zip during a conformance review against NFH-012. Changes fall into two categories: **failures** (hard violations of MUST/MUST NOT rules in the guide) and **warnings** (SHOULD-level issues that were easy to correct in the same pass). A third category — pending items — records issues that require a design decision before they can be resolved.

The original zip was structurally sound. All seven schema packs (`resource`, `offer`, `contract`, `participant`, `settlement`, `performance`, `consideration`) had the required artifact set, correct versioning, proper `allOf` inheritance from `beckn:Attributes`, good regulatory anchoring, and well-written property descriptions. The issues found were concentrated in naming and vocabulary cross-linking.

---

## Changes Made

### Fix 1 — `resource/v1/attributes.yaml`: root object renamed `CreditResourceAttributes` → `CreditProduct`

**File:** `Schema/extensions/finance/resource/v1/attributes.yaml`

**What changed:** The root schema object key was `CreditResourceAttributes`. It is now `CreditProduct`.

A secondary stale reference in the same object's `description` block referred to `CreditOfferAttributes` (an old name). That was updated to `CreditApproval` to match the name used in every other artifact.

**Why:** NFH-012 explicitly lists `*Attributes` as a NOT RECOMMENDED suffix (section "NOT RECOMMENDED — generic suffixes such as `Attributes`, `Item`, `Offer`, `Resource`"). The guide's reasoning is that the suffix leaks the *carrier* (the Beckn `Resource` slot the schema attaches to) into the concept name, which conflates two separate concerns.

The concept this schema models is a credit product — a loan template published by a lender to a catalog. A developer, a borrower, or an AI agent processing this payload should read `CreditProduct` and understand what it is, without needing to know that it travels inside a Beckn `Resource` envelope. That carrier relationship is already declared correctly in:

- `profile.json` → `"extends": "beckn:Resource"`
- `attributes.yaml` → `x-beckn-attaches-to: Resource.resourceAttributes`
- `vocab.jsonld` → the `rdfs:subClassOf` chain

So the carrier information has a home — it just isn't the class name.

**Consistency note:** This brings `finance/resource/v1` into line with the pattern used in every other ION sector. In hospitality, the equivalent pack uses `RestaurantMenuItem` (not `HospitalityResourceAttributes`). In trade, it uses the category sub-object names directly. The rule is: directory names use Beckn slot / lifecycle stage language (`resource/`, `offer/`, `contract/`); class names describe the real-world concept.

| Sector | Beckn slot | Directory | Class name |
|---|---|---|---|
| Hospitality | `resourceAttributes` | `hospitality/delivery/v1` | `RestaurantMenuItem` |
| Finance | `resourceAttributes` | `finance/resource/v1` | `CreditProduct` ✓ (was `CreditResourceAttributes`) |
| Finance | `offerAttributes` | `finance/offer/v1` | `CreditApproval` |
| Finance | `contractAttributes` | `finance/contract/v1` | `LoanAgreement` |

---

### Fix 2 — `participant/v1/schema.json`: title corrected to `CreditParticipant`

**File:** `Schema/extensions/finance/participant/v1/schema.json`

**What changed:** The `title` field at the root of the JSON Schema document was `"Credit Participant Attributes (v1)"`. It is now `"CreditParticipant"`.

**Why:** The same reasoning as Fix 1 — the word "Attributes" in a schema title is the NOT RECOMMENDED pattern. Additionally, the version number `(v1)` does not belong in the title; versioning is expressed by the directory path and the `$id` URI, not the human-readable title.

Note that `participant/v1/attributes.yaml` was already correct — it uses `Borrower` and `Lender` as its root object keys (clean concept nouns, no suffix). The mismatch was only in `schema.json`. Both files now describe the same concept cleanly.

---

### Fix 3 — `offer/v1/vocab.jsonld`: `owl:sameAs` added to duplicate concept schemes

**File:** `Schema/extensions/finance/offer/v1/vocab.jsonld`

**What changed:** `ion:YieldType` and `ion:CalculationMethod` are declared as `skos:ConceptScheme` nodes in both `resource/v1/vocab.jsonld` and `offer/v1/vocab.jsonld`. Before this fix, both declarations used different namespace IRIs (`finance/resource/v1#` vs `finance/offer/v1#`) with no linkage between them, making them semantically opaque to any agent or processor trying to resolve whether they were the same concept.

`owl:sameAs` assertions have been added to the `offer/v1` copies pointing at the canonical IRIs in `resource/v1`:

```json
{
  "@id": "ion:YieldType",
  "owl:sameAs": { "@id": "https://schema.ion.id/finance/resource/v1#YieldType" },
  ...
}
```

The `owl` prefix was also added to the `offer/v1/vocab.jsonld` `@context` block.

**Why:** NFH-012 CON-012-03 (semantic stability) and the general vocabulary alignment principle. A JSON-LD processor encountering `ion:YieldType` in an `offer` payload and `ion:YieldType` in a `resource` payload had no way to know these were the same taxonomy without an explicit relation. The `owl:sameAs` provides the machine-readable bridge.

The canonical home for `YieldType` and `CalculationMethod` is `resource/v1` — that is where the full concept scheme with all values (`FIXED`, `FLOATING`, `COMBINED`, `MARGIN`, `NISBAH`, `UJRAH`, `ZERO_COST`) is defined. The `offer/v1` copies exist because the offer pack needs to reference these values in its rate fields. The long-term clean solution is a shared `finance/core/vocab.jsonld` (see Pending section), but the `owl:sameAs` linkage resolves the immediate semantic ambiguity.

---

### Fix 4 — `resource/v1/vocab.jsonld`: canonical comments updated

**File:** `Schema/extensions/finance/resource/v1/vocab.jsonld`

**What changed:** The `rdfs:comment` on `ion:YieldType` and `ion:CalculationMethod` was updated to explicitly state that these are the canonical definitions and that `offer/v1` holds `owl:sameAs` links pointing here.

**Why:** Companion to Fix 3. Makes the canonical/alias relationship visible to any human reader of `resource/v1` without needing to cross-reference the offer pack.

---

### Fix 5 — `error/finance.yaml`: `affected_field` path corrected for `ION-FIN-6000`

**File:** `error/finance.yaml`

**What changed:** Error code `ION-FIN-6000` ("Credit score below minimum threshold") had `affected_field: participantAttributes.creditScore`. This field does not exist in `participant/v1`. The correct path is `offerAttributes.internalCreditScore.score`, which is where the lender's credit assessment result lives in `offer/v1`.

**Before:**
```yaml
affected_field: participantAttributes.creditScore
```

**After:**
```yaml
affected_field: offerAttributes.internalCreditScore.score
```

**Why:** A broken `affected_field` path means any tooling that uses the error registry to surface contextual help to a developer — pointing them to the right schema field — will silently fail or point to a non-existent location. `internalCreditScore.score` in `offer/v1` is the structured credit assessment result; `participantAttributes.slik` (used correctly in `ION-FIN-6001`) is the bureau data input. These are different objects at different lifecycle stages.

---

## What Is Not Changed (Pending)

These issues were identified during the review but not fixed in this pass because they require a design decision or have downstream implications.

### Pending 1 — Shared concept schemes should move to `finance/core/vocab.jsonld`

`ion:YieldType` and `ion:CalculationMethod` are currently defined in `resource/v1/vocab.jsonld` and aliased from `offer/v1/vocab.jsonld` via `owl:sameAs`. The architecturally cleaner solution is a shared `Schema/extensions/finance/core/vocab.jsonld` that both packs import, so neither "owns" these cross-cutting taxonomies.

This mirrors how the ION schema structure handles cross-sector core concepts (the `core/` layer referenced in the Schema README). No code change is needed to make the schema work today — the `owl:sameAs` fix (Fix 3) resolves the immediate semantic issue. The consolidation into a `finance/core/` pack is a v1.1 task.

**Action required:** Create `Schema/extensions/finance/core/v1/vocab.jsonld` with `YieldType`, `CalculationMethod`, and any other concepts shared across more than one finance pack. Update `context.jsonld` imports in `resource/v1` and `offer/v1` to reference the shared file. Remove the duplicate declarations and the `owl:sameAs` workaround.

---

### Pending 2 — Closed enums in `schema.json` files: review against CON-012-28

Several fields across the packs use `"enum": [...]` in their JSON Schema definitions. NFH-012 CON-012-28 recommends replacing restrictive enums with open strings backed by `vocab.jsonld` vocabulary terms and `skos:broader` linkages, so that new values can be added without a schema version bump.

Fields to review:

| Pack | Field | Current enum values | Recommendation |
|---|---|---|---|
| `offer/v1` | `offerStatus` | `APPROVED`, `CONDITIONALLY_APPROVED`, `REJECTED`, `EXPIRED`, `UNDER_EVALUATION` | **Keep closed.** These are OJK regulatory decision states with legal meaning; deterministic filtering is required. |
| `offer/v1` | `internalCreditScore.category` | `EXCELLENT`, `GOOD`, `FAIR`, `POOR` | **Consider opening.** This is a lender-proprietary rating — different lenders may use different scales. Open string + `skos:broader ion:CreditRiskCategory` is more appropriate. |
| `offer/v1` | `internalCreditScore.slikGrade` | `"1"`, `"2"` | **Keep closed.** OJK SLIK collectibility grades are a fixed regulatory taxonomy. |
| `contract/v1` | `loanStatus` | `ACTIVE`, `SPECIAL_MENTION`, etc. | **Keep closed.** OJK POJK 40/2019 collectibility classification — fixed regulatory values. |
| `performance/v1` | `disbursementMethod` | `BANK_TRANSFER`, `DEVELOPER_TRANSFER`, etc. | **Consider opening.** New disbursement rails (e.g. GovTech virtual accounts, QRIS-based) should not require a schema bump. Open string + vocab term. |
| `settlement/v1` | `paymentStatus` | `UNPAID`, `PAID`, `OVERDUE`, `PARTIAL` | **Keep closed.** Maps directly to Beckn `Settlement.status.code` states — must be deterministic. |

**Action required:** For the two "consider opening" fields, replace the `enum` constraint in `schema.json` with `type: string` and add the corresponding `skos:Concept` declarations to `vocab.jsonld` with `skos:broader` pointing at a parent concept. This is a non-breaking change (MINOR version bump).

---

## Summary

| # | File | Type | Status |
|---|---|---|---|
| 1 | `Schema/extensions/finance/resource/v1/attributes.yaml` | Failure | ✅ Fixed |
| 2 | `Schema/extensions/finance/participant/v1/schema.json` | Failure | ✅ Fixed |
| 3 | `Schema/extensions/finance/offer/v1/vocab.jsonld` | Warning | ✅ Fixed |
| 4 | `Schema/extensions/finance/resource/v1/vocab.jsonld` | Warning | ✅ Fixed |
| 5 | `error/finance.yaml` | Warning | ✅ Fixed |
| 6 | Shared `finance/core/vocab.jsonld` for cross-pack concepts | Warning | ⏳ Pending — v1.1 task |
| 7 | Open enum review for `internalCreditScore.category` and `disbursementMethod` | Warning | ⏳ Pending — needs design decision |
