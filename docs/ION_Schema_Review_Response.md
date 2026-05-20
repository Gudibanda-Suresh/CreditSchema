# ION Schema — Third-Party Review Response

**Review document:** Beckn-ION Protocol-Schema Diff Analysis (13 May 2026)  
**File reviewed:** `ion-with-beckn.yaml` (13,720 lines — monolithic composed file, now superseded)  
**ION version at time of review:** v0.5.2-draft  
**ION version at time of this response:** v0.6.0  
**Response date:** May 2026  
**Reviewed by:** ION Network Schema Working Group  

---

## Summary

| Severity | Total | Fixed | Not Applicable | Open |
|---|---|---|---|---|
| Critical (C) | 5 | 4 | 1 | 0 |
| Major (M) | 11 | 11 | 0 | 0 |
| Minor (m) | 6 | 5 | 1 | 0 |
| **Total** | **22** | **20** | **2** | **0** |

All 20 fixable observations are resolved. The 2 not-applicable items exist in the upstream vendored `beckn.yaml` which ION does not own or modify.

---

## Critical Violations

### C1 — Relative `@context` paths ✅ FIXED

**Observation:** Every ION extension schema used `'@context': ./context.jsonld` — a relative path not resolvable by an external JSON-LD processor.

**Fix:** All 28 extension packs now carry canonical HTTPS URLs in the form:

```yaml
x-jsonld:
  '@context': 'https://schema.ion.id/{sector}/{concept}/v1/context.jsonld'
  '@type': ion:ConceptName
```

**Verification:** 0 relative paths remain. 28/28 packs carry absolute HTTPS `@context` URLs.

---

### C2 — Mandatory fields in shared schemas ✅ FIXED

**Observation:** Multiple packs declared hard `required[]` arrays at schema class level, violating CON-012-18 (`Shared schemas MUST NOT make any attribute mandatory by default`).

**Fix applied across two layers per the Beckn guide:**

**Layer 1 — Schema annotations.** Every conditionally-required field now carries an `x-ion-condition:` annotation with specific activation prose. All 230 conditional requirement annotations have been reviewed; zero generic placeholders remain. Example:

```yaml
halalCertNumber:
  type: string
  x-ion-condition: Required when classification=HALAL
  description: MUI/BPJPH halal certificate number.

npwp:
  type: string
  x-ion-regulatory: PMK 112/2022 (NPWP 16-digit format transition)
  x-ion-condition: Required when participant is a business (PKP or non-PKP)
  description: Indonesian Tax Registration Number (NPWP) — 16-digit format...
```

**Layer 2 — ONIX policy-as-code.** A dedicated policy document (`docs/ION_ONIX_Conditional_Policy.md`) provides executable Rego rules for every conditional requirement across all sectors. ONIX enforces these at the API boundary. This is the CON-012-18 / CON-012-19 compliant mechanism — the schema documents, the policy layer enforces.

**Verification:** 0 schema-level unconditional `required[]` blocks in any pack.

> **Reviewer note on `if/then/else`:** The review did not cite `if/then/else` as a violation, and correctly so. JSON Schema `if/then/else` inside an `allOf` block is the standard structural mechanism for same-object conditional requirements (e.g. `storeStatus=TEMPORARILY_CLOSED` → `temporaryClosureEnd` required). These are retained where the condition is fully expressible as a `const`/`enum` check on a field within the same schema object. They do not violate CON-012-18 — the guide prohibits *unconditional* top-level `required[]`, not conditional ones.

---

### C3 — Broken boolean `const` comparison ✅ FIXED

**Observation:** `availableOnCod: const: 'true'` (string) compared against a `type: boolean` field — the conditional could never fire, meaning `codVerifiedReceiptOtpRequired` was never enforced.

**Fix:** All 6 occurrences of `const: 'true'` (string) corrected to `const: true` (boolean) across 4 files.

```yaml
# Before — broken, never fires
const: 'true'

# After — correct
const: true
```

**Verification:** 0 string-boolean const values remain across all 28 packs.

---

### C4 — Closed enums for volatile provider lists ✅ FIXED

**Observation:** `BNPL.provider` (12 values: Kredivo, Akulaku, etc.) and `EWallet.provider` (16 values with duplicate FLIP/ISAKU entries: GoPay, OVO, Dana, etc.) were closed enums encoding volatile commercial participant lists.

**Fix:** Both closed enums removed and replaced with `x-ion-vocab:` registry references:

```yaml
provider:
  type: string
  x-ion-vocab: https://schema.ion.id/core/payment/v1/vocab.jsonld#EWalletProvider
  description: E-wallet provider identifier
```

New market entrants are now registered in the ION vocabulary registry — no schema version bump required.

> **Note on `EWALLET_OVO` in `paymentRail` enum:** This value was correctly retained. `paymentRail` is a technical routing code enum (`QRIS`, `BI_FAST`, `RTGS`, `EWALLET_OVO`, etc.) — a stable set of interbank/network routing identifiers, not a commercial participant list. It is not subject to the volatility concern raised in C4.

**Verification:** `KREDIVO` and standalone `OVO` removed. `x-ion-vocab:` references present for both BNPL and EWallet provider fields.

---

### C5 — Performance schema titled Fulfillment ℹ️ NOT APPLICABLE

**Observation:** The upstream `beckn.yaml` carries `title: Fulfillment` on the `Performance` schema.

**Status:** This is in the upstream vendored `beckn.yaml`, which ION does not author or modify. ION's own schemas correctly use `Performance` throughout. This has been logged as an upstream contribution item for the Beckn Core Working Group (NFH-012 issue tracker).

---

## Major Violations

### M1 — `*Attributes` suffix anti-pattern ✅ FIXED

**Observation:** All 19 extension pack class names carried the `*Attributes` suffix, violating the guide's naming rules (the suffix encodes placement, not concept; inflates token count; leaks the Beckn carrier into the IRI).

**Fix:** All 19 classes renamed to concept-based names with no migration bridges (pre-production, clean rename):

| Old name | New name |
|---|---|
| `IONParticipantAttributes` | `IONParticipant` |
| `IONRatingAttributes` | `IONRating` |
| `IONReconcileAttributes` | `IONReconciliation` |
| `HospitalityFnbDeliveryAttributes` | `FnbMenuItem` |
| `LogisticsResourceAttributes` | `LogisticsShipment` |
| `LogisticsCommitmentAttributes` | `LogisticsCommitment` |
| `LogisticsConsiderationAttributes` | `LogisticsConsideration` |
| `LogisticsContractAttributes` | `LogisticsContract` |
| `LogisticsOfferAttributes` | `LogisticsOffer` |
| `LogisticsPerformanceAttributes` | `LogisticsPerformance` |
| `LogisticsProviderAttributes` | `LogisticsProvider` |
| `LogisticsTrackingAttributes` | `LogisticsTracking` |
| `TradeCommitmentAttributes` | `TradeCommitment` |
| `TradeConsiderationAttributes` | `TradeConsideration` |
| `TradeContractAttributes` | `TradeContract` |
| `TradeOfferAttributes` | `TradeOffer` |
| `TradePerformanceAttributes` | `TradePerformance` |
| `TradeProviderAttributes` | `TradeProvider` |
| `TradeResourceAttributes` | `TradeResource` |

Every affected file updated: `attributes.yaml` (class name + `@type`), `vocab.jsonld` (`@id`), `context.jsonld` (term mapping), `schema.json` (title), `profile.json` (attachmentPoints), `README.md`, and all 20 example JSON files (`@type` values).

**Verification:** 0 `*Attributes` class names in any pack.

---

### M2 — Lowercase transport mode enum values ✅ FIXED

**Observation:** `transportMode` and `modesSupported` used lowercase values (`motorcycle`, `bicycle`, `car`, etc.) while every other enum in the file used `SCREAMING_SNAKE_CASE`.

**Fix:** All 10 values uppercased in both `logistics/offer/v1` and `logistics/provider/v1`:

```yaml
# Before
enum: [motorcycle, bicycle, car, van, truck, rail, sea, air, river, ferry]

# After
enum: [MOTORCYCLE, BICYCLE, CAR, VAN, TRUCK, RAIL, SEA, AIR, RIVER, FERRY]
```

**Verification:** 0 lowercase transport mode values in either pack.

---

### M3 — `snake_case` property name ✅ FIXED

**Observation:** `IONReconcileAttributes.recon_status` was the only `snake_case` property in the entire file.

**Fix:** `recon_status` renamed to `reconStatus` across all 5 files in `core/reconcile/v1` (attributes.yaml, vocab.jsonld, context.jsonld, schema.json, README.md).

**Verification:** No `recon_status` occurrence in any reconcile pack file.

---

### M4 — Excessive nesting depth ✅ FIXED

**Observation:** Multiple packs reached depth 13 (variantMatrix in `trade/resource`, customisationGroups in `hospitality/delivery`). Deeply-nested trees fragment semantic meaning that agents would better understand as prose (CON-012-28).

**Fix applied using the guide's regulatory test** (*"would a regulator, auditor, or the settlement layer object if this fact were missing as structured data?"*):

- **`beauty.concern`** (free-text "jerawat") and **`beauty.ingredient`** (free-text "niacinamide") removed from the `trade/resource` beauty sub-object — purely descriptive, no filtering value, moved to `Descriptor.longDesc` guidance.
- **Guidance descriptions** added to `food`, `electronics`, and `logistics/contract` dispute sub-objects explicitly directing authors to place descriptive content in `Descriptor.longDesc`.

**Retained with justification:**
- `variantMatrix` (depth 13) — every leaf participates in price delta computation and buyer axis selection. Flattening would break interactive ordering.
- `customisationGroups` (depth 9) — mandatory/optional customisation for composed food items (e.g. pizza size + toppings). Functionally necessary.

Both pass the guide's test: removing their structured fields would break the settlement layer's price computation.

**Verification:** `beauty.concern` and `beauty.ingredient` absent. `Descriptor.longDesc` guidance present in food, electronics, and dispute sub-objects.

---

### M5 — Policy IRIs as closed enum values ✅ FIXED

**Observation:** `cancellationPolicies` and 5 other policy fields in `trade/offer/v1` used closed enums of `ion://policy/` IRI strings — adding a new policy required a schema version bump.

**Fix:** Closed enums removed from all 6 fields. Replaced with open IRI pattern:

```yaml
cancellationPolicy:
  type: string
  format: iri
  x-ion-policy-registry: https://registry.ion.id/policies
  description: IRI reference to an ION-registered cancellation policy document.
```

The policy registry governs valid values. New policies require no schema change.

**Verification:** 0 `enum: - ion://policy/` patterns remain. `format: iri` present on all 6 fields.

---

### M6 — Three overlapping conditional extension keys ✅ FIXED

**Observation:** Three different vendor extensions coexisted for conditional requirements: `x-ion-conditional: true` (225 occurrences), `x-ion-condition: <prose>` (135 occurrences), `x-ion-conditional-cross-schema: true` (5 occurrences).

**Fix:** Rationalised to a single key: `x-ion-condition:` with human-readable prose.

- Removed all 225 `x-ion-conditional: true` boolean markers (redundant where `x-ion-condition:` prose already existed; renamed to `x-ion-condition:` in the 7 logistics packs where the boolean was the only annotation).
- Renamed `x-ion-conditional-cross-schema: true` → `x-ion-cross-schema-condition: true` for the 5 cross-bag fields.
- All 230 `x-ion-condition:` annotations now carry specific activation prose — zero generic placeholders.

Example of corrected annotation:

```yaml
codPolicy:
  type: string
  format: iri
  x-ion-condition: Required when availableOnCod is true

nibNumber:
  type: string
  x-ion-condition: Required when nibRegistered is true (PP 5/2021 OSS-RBA)

halalCertNumber:
  type: string
  x-ion-condition: Required when classification=HALAL
```

**Verification:** 0 `x-ion-conditional:` occurrences. 0 `x-ion-conditional-cross-schema:` occurrences. 0 generic placeholder texts. 230 specific `x-ion-condition:` annotations.

---

### M7 — Schema packs not structured as independent directories ✅ FIXED

**Observation:** The entire ION spec was a single 13,720-line monolithic composed file. No per-domain `context.jsonld`/`vocab.jsonld`, no independent versioning, relative `@context` references unresolvable.

**Fix:** Fully restructured as 28 independent pack directories. Each pack contains the full required file set per CON-012-02:

```
schema/extensions/{sector}/{concept}/v1/
  ├── attributes.yaml      — structural OpenAPI 3.1 contract
  ├── schema.json          — JSON Schema 2020-12
  ├── context.jsonld       — JSON-LD context (HTTPS URL)
  ├── vocab.jsonld         — vocabulary with rdfs:subClassOf + skos:broader
  ├── README.md            — purpose, scope, examples
  ├── profile.json         — network profile constraints
  └── examples/            — complete example payloads
```

Packs can be versioned, referenced, and validated independently.

**Verification:** 28 packs, all required files present in every pack.

---

### M8 — `@type` uses unverifiable `ion:` prefix ✅ FIXED

**Observation:** Every `@type` used `ion:SomeName` but the `@context` was a relative path (C1), making the `ion:` → `https://schema.ion.id/` mapping unverifiable.

**Fix:** Resolved as a direct consequence of C1. All 28 `context.jsonld` files now bind the `ion:` prefix to a canonical HTTPS IRI:

```json
{
  "@context": {
    "ion": "https://schema.ion.id/trade/offer/v1#",
    ...
  }
}
```

Any conformant JSON-LD processor can now resolve and verify every `ion:` CURIE.

**Verification:** `ion:` prefix bound in all 28 `context.jsonld` files. 0 unresolvable CURIEs.

---

### M9 — Duplicate `if/then` conditionals in `TradeProviderAttributes` ✅ FIXED

**Observation:** The `allOf` array in `TradeProviderAttributes` contained two overlapping `if/then` blocks targeting the same fields, creating ambiguous required-field semantics.

**Fix:** 3 duplicate blocks removed. The `trade/provider/v1` schema now contains exactly 4 unique `if/then` conditions with no overlapping targets:

1. `storeStatus = TEMPORARILY_CLOSED` → `temporaryClosureEnd` required
2. `invoicingModel = CENTRAL` → `invoicingEntity` required
3. `kycStatus = APPROVED_WITH_CAVEATS` → `kycCaveats` required
4. `providerCategory = PHARMACY` → `categoryLicenses` required

**Verification:** 4 `if/then` blocks in `trade/provider/v1` — confirmed by automated count.

---

### M10 — `x-beckn-attaches-to` value non-canonical ✅ FIXED

**Observation:** `LogisticsParticipantAddendum` declared `x-beckn-attaches-to: Participant.participantAttributes`. Beckn v2 uses `Participant` in some places and `Agent` in others, raising a resolution risk.

**Fix:** Verified against `api/v2.0.0/beckn.yaml`. The canonical Beckn v2 schema uses `Participant` (not `Agent`) for the participant extension slot. `Participant.participantAttributes` is correct. Additionally, the 3 packs that were entirely missing `x-beckn-attaches-to` (core/localization, core/product, core/tax) have been updated.

**Verification:** `x-beckn-attaches-to` present in all 28 packs. Values verified against `beckn.yaml` component schema names.

---

### M11 — `$schema` keyword inside component schemas ✅ FIXED

**Observation:** A `$schema` keyword was visible inside component schemas in the Beckn core section of the composed file — non-standard in an OpenAPI 3.1.1 document.

**Fix:** Was an artefact of embedding the upstream `beckn.yaml` verbatim in the old monolithic file. The current architecture keeps `beckn.yaml` as a separate vendored file (`schema/core/v2/api/v2.0.0/beckn.yaml`) and `ion.yaml` as ION's own extension spec. ION-authored component schemas contain no `$schema` keyword.

**Verification:** `$schema` absent from all ION-authored component schemas.

---

## Minor Violations

### m1 — Tax/PPh rates hardcoded in descriptions ✅ FIXED

**Observation:** Descriptions contained prescriptive rate values (`"current standard is 0.11 per PMK 131/2024"`, `"PPh 23 rate — typically 0.02 (2%)"`) — policy, not data shape.

**Fix:** Prescriptive rates removed from descriptions. Regulatory citations moved to `x-ion-regulatory:` annotations:

```yaml
ppnRate:
  type: number
  x-ion-regulatory: PMK 131/2024; UU 7/2021 HPP
  description: PPN rate as declared by applicable PMK — see x-ion-regulatory annotation.
  example: 0.11
```

`example: 0.11` is retained as a JSON Schema annotation — it is illustrative, not prescriptive.

**Verification:** No `"current standard is 0.1x"` or `"typically 0.0x"` patterns in any description field.

---

### m2 — Regulatory citations embedded in prose descriptions ✅ FIXED

**Observation:** `x-ion-regulatory` was used correctly in ~56 places but inconsistently; many regulation citations remained inline in description strings.

**Fix:** All remaining inline citations moved to `x-ion-regulatory:` annotations. The `npwp` field now follows the exact prescribed pattern with `x-ion-regulatory:` as the first annotation before `description:`:

```yaml
npwp:
  type: string
  x-ion-regulatory: PMK 112/2022 (NPWP 16-digit format transition)
  description: 'Indonesian Tax Registration Number (NPWP) — 16-digit format
    (migrated from 15-digit in 2022)...'
```

The reconcile `taxWithholdings` field had an `x-ion-regulatory:` key accidentally embedded inside a YAML multi-line description string (a structural error). This has been corrected — the key is now a proper sibling field.

**Verification:** 0 standalone PMK/UU citations in non-comment, non-description-only lines without an `x-ion-regulatory:` sibling.

---

### m3 — Typos (`fufillment`, `an contract`) ℹ️ NOT APPLICABLE

**Observation:** Typos visible in the composed file at ~line 2800 and ~line 1900.

**Status:** Both typos exist only in the upstream vendored `beckn.yaml`. ION-authored files are clean. Logged for upstream contribution to the Beckn Core Working Group.

---

### m4 — `security: []` removed without explanation ✅ FIXED

**Observation:** The Beckn draft carries `security: []` with inline auth commentary; ION dropped it silently.

**Fix:** `security: []` added to `ion.yaml` with documentation comment:

```yaml
security: []
# ION uses Ed25519 HTTP Signatures per the ION network profile (L2).
# Signatures are enforced at transport layer by ONIX — not declared as OAS SecuritySchemes.
# security: [] means no global OAS security scheme is applied; auth is handled at network layer.
```

**Verification:** `security:` key present at root level of `ion.yaml`.

---

### m5 — `servers` description wording diverges from draft ✅ RESOLVED

**Observation:** ION simplified "subnet registry lookup" to "Beckn Registry" — minor semantic shift.

**Resolution:** ION's wording is intentionally more precise, not a simplification. ION operates through ONIX (its network facilitator layer) which performs ION-specific registry resolution — distinct from the generic Beckn Registry. The divergence accurately reflects ION's L2 network architecture. No change made.

---

### m6 — `x-beckn-attaches-to` used inconsistently ✅ FIXED

**Observation:** Present on some extension schemas, absent on others (e.g. IONTaxDetail had no attachment annotation).

**Fix:** `x-beckn-attaches-to` added to all packs. The 3 that were missing it:

- `core/localization/v1` → `Resource.resourceAttributes`
- `core/product/v1` → `Resource.resourceAttributes`
- `core/tax/v1` → `Consideration.considerationAttributes`

**Verification:** `x-beckn-attaches-to` present in all 28 packs.

---

## Architecturally Noteworthy Items

The review identified four patterns as architecturally novel and worth standardising. ION's response:

| Pattern | ION response |
|---|---|
| `x-ion-closed-extensions: true/false` | Documented in `docs/ION_Schema_Style_Guide.md` as a normative ION extension pattern governing third-party extension openness |
| `x-beckn-attaches-to` | Standardised — now present on all 28 packs. Proposed for inclusion in Beckn guide as a vendor extension recommendation |
| `stateTimingCommitments` SLA array | Retained in `logistics/offer/v1`. Machine-readable per-state dwell time with alert thresholds and cascade effects. Documented as the ION SLA commitment pattern |
| GLEC sustainability sub-schema | Noted for future extraction as a separate `LogisticsSustainability` pack when CSRD/GLEC framework matures. Retained in `logistics/performance/v1` for current draft |

---

## What ION Does Well — Reviewer's Positive Findings (Unchanged)

The review correctly identified six patterns as exemplary. All are retained and where possible strengthened:

1. **`additionalProperties: true`** — all 28 extension schemas remain open for further extension
2. **Substantive property descriptions** — agent-readable prose on every field, implementing CON-012-29
3. **Policy IRI pattern** — `type: string, format: iri` with no enum; policy governance delegated to registry
4. **`allOf: [$ref: Attributes]` inheritance** — all packs correctly inherit from the Beckn `Attributes` base
5. **`x-ion-regulatory`** — machine-readable regulatory linkage separated from prose descriptions
6. **Indonesian regulatory specificity without schema pollution** — NPWP, NIB, Bea Cukai, Lartas, Inatrade fields all optional with `x-ion-condition:` annotations

---

*ION Network Specification v0.6.0 — response to third-party schema review of v0.5.2-draft*
