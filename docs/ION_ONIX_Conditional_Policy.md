# ION Conditional Requirements — ONIX Policy Layer

## How conditional requirements work in ION

The Beckn Schema Design Guide (NFH-012, CON-012-18) states:

> *"Shared schemas MUST NOT make any attribute mandatory by default. Mandatoriness MUST be expressed through network policy or manufacturer policy-as-code, not through the shared schema."*

ION implements this in two complementary ways:

**1. `x-ion-condition:` annotations in `attributes.yaml`**
Every conditionally-required field carries an `x-ion-condition:` annotation with a human- and agent-readable prose rule. This is documentation — it tells schema authors, AI agents, and developers exactly when a field is required. It does not enforce anything by itself.

**2. ONIX Rego policy files (this document)**
ONIX is ION's Network Facilitator Layer. It enforces conditional requirements at the API boundary by executing Rego (Open Policy Agent) rules against every inbound payload before it is accepted onto the network. The rules in this document are the machine-executable form of the `x-ion-condition:` annotations in the schemas.

---

## Rego policy structure

Each policy file lives at:
```
policies/onix-conditional/<sector>/<pack>/<version>/conditional.rego
```

All rules follow the pattern:
```rego
package ion.onix.conditional.<sector>.<pack>

deny[msg] {
  # condition that must be true for the requirement to activate
  # check that the required field is present
  # msg describes the violation
}
```

ONIX executes `deny` rules against `input` which is the full `Attributes` container from the message. A non-empty `deny` set causes ONIX to return a `NACK` with the message text as the error detail.

---

## Core sector policies

### `core/participant/v1`

```rego
package ion.onix.conditional.core.participant

# NPWP required for business participants
# x-ion-condition: Required when participant is a business (PKP or non-PKP) and NPWP is available
deny[msg] {
  p := input.participantAttributes
  p.role == "SELLER"
  not p.npwp
  msg := "participantAttributes.npwp is required for SELLER role participants (PMK 112/2022)"
}

deny[msg] {
  p := input.participantAttributes
  p.role == "MERCHANT"
  not p.npwp
  msg := "participantAttributes.npwp is required for MERCHANT role participants (PMK 112/2022)"
}

# NIB required for business participants
# x-ion-condition: Required for all seller/provider/merchant participants
deny[msg] {
  p := input.participantAttributes
  p.role == "SELLER"
  not p.nib
  msg := "participantAttributes.nib is required for SELLER role participants (PP 5/2021 OSS-RBA)"
}
```

### `core/payment/v1`

```rego
package ion.onix.conditional.core.payment

# Payment confirmation fields
# x-ion-condition: Required when payment is confirmed
deny[msg] {
  p := input.settlementAttributes
  p.status == "PAID"
  not p.transactionId
  msg := "settlementAttributes.transactionId is required when payment status is PAID"
}

deny[msg] {
  p := input.settlementAttributes
  p.status == "PAID"
  not p.paidAt
  msg := "settlementAttributes.paidAt is required when payment status is PAID"
}
```

### `core/tax/v1`

```rego
package ion.onix.conditional.core.tax

# Tax reference required when not exempt
# x-ion-condition: Required when taxRegime is not EXEMPT
deny[msg] {
  t := input.considerationAttributes
  t.taxRegime != "EXEMPT"
  not t.taxReferenceNumber
  msg := "considerationAttributes.taxReferenceNumber is required when taxRegime is not EXEMPT"
}

deny[msg] {
  t := input.considerationAttributes
  t.taxRegime != "EXEMPT"
  not t.ppnRate
  msg := "considerationAttributes.ppnRate is required when taxRegime is not EXEMPT"
}
```

---

## Hospitality sector policies

### `hospitality/delivery/v1`

```rego
package ion.onix.conditional.hospitality.restaurant_ordering

# Halal certificate required when classification is HALAL
# x-ion-condition: Required when classification=HALAL
# Regulatory basis: UU 33/2014 tentang Jaminan Produk Halal; PP 39/2021
deny[msg] {
  r := input.resourceAttributes
  r.classification == "HALAL"
  not r.halalCertNumber
  msg := "resourceAttributes.halalCertNumber is required when classification is HALAL (UU 33/2014 JPH)"
}

# isAlcoholic flag required when item contains alcohol
# x-ion-condition: Required when item contains alcohol
deny[msg] {
  r := input.resourceAttributes
  r.isAlcoholic == true
  not r.ageRestricted
  msg := "resourceAttributes.ageRestricted must be true when isAlcoholic is true"
}

# customisationGroups required for COMPOSED structure
# x-ion-condition: Required when resourceStructure IN [COMPOSED, WITH_EXTRAS]
deny[msg] {
  r := input.resourceAttributes
  r.resourceStructure == "COMPOSED"
  not r.customisationGroups
  msg := "resourceAttributes.customisationGroups is required when resourceStructure is COMPOSED"
}
```

---

## Trade sector policies

### `trade/offer/v1`

```rego
package ion.onix.conditional.trade.offer

# Return policy details required when item is returnable
# x-ion-condition: Required when returnable is true
deny[msg] {
  o := input.offerAttributes
  o.returnable == true
  not o.returnPolicy
  msg := "offerAttributes.returnPolicy is required when returnable is true"
}

# Warranty details required when warranty is declared
# x-ion-condition: Required when warrantyType is not NONE
deny[msg] {
  o := input.offerAttributes
  o.warrantyType != "NONE"
  not o.warrantyPolicy
  msg := "offerAttributes.warrantyPolicy is required when warrantyType is not NONE"
}
```

### `trade/provider/v1`

```rego
package ion.onix.conditional.trade.provider

# Temporary closure end date required when store is temporarily closed
# x-ion-condition: Required when storeStatus is TEMPORARILY_CLOSED
deny[msg] {
  p := input.providerAttributes
  p.storeStatus == "TEMPORARILY_CLOSED"
  not p.temporaryClosureEnd
  msg := "providerAttributes.temporaryClosureEnd is required when storeStatus is TEMPORARILY_CLOSED"
}

# Invoicing entity required for centralised invoicing
# x-ion-condition: Required when invoicingModel is CENTRAL
deny[msg] {
  p := input.providerAttributes
  p.invoicingModel == "CENTRAL"
  not p.invoicingEntity
  msg := "providerAttributes.invoicingEntity is required when invoicingModel is CENTRAL"
}

# KYC caveats required when approval has caveats
# x-ion-condition: Required when kycStatus is APPROVED_WITH_CAVEATS
deny[msg] {
  p := input.providerAttributes
  p.kycStatus == "APPROVED_WITH_CAVEATS"
  not p.kycCaveats
  msg := "providerAttributes.kycCaveats is required when kycStatus is APPROVED_WITH_CAVEATS"
}

# Category licences required for regulated categories
# x-ion-condition: Required for regulated categories (pharmacy, alcohol, tobacco, meat)
deny[msg] {
  p := input.providerAttributes
  p.providerCategory == "PHARMACY"
  not p.categoryLicenses
  msg := "providerAttributes.categoryLicenses is required for PHARMACY category (PP 5/2021; PerBPOM)"
}
```

### `trade/contract/v1`

```rego
package ion.onix.conditional.trade.contract

# Tax invoice reference required for PKP sellers
# x-ion-condition: Required for PKP sellers when invoiceType=TAX_INVOICE
deny[msg] {
  c := input.contractAttributes
  c.invoiceType == "TAX_INVOICE"
  not c.taxInvoiceNumber
  msg := "contractAttributes.taxInvoiceNumber is required when invoiceType is TAX_INVOICE (UU 42/2009 PPN)"
}
```

---

## Logistics sector policies

### `logistics/offer/v1`

```rego
package ion.onix.conditional.logistics.offer

# COD policy required when COD is available
# x-ion-condition: Required when availableOnCod is true
deny[msg] {
  o := input.offerAttributes
  o.availableOnCod == true
  not o.codPolicy
  msg := "offerAttributes.codPolicy is required when availableOnCod is true"
}

deny[msg] {
  o := input.offerAttributes
  o.availableOnCod == true
  not o.codVerifiedReceiptOtpRequired
  msg := "offerAttributes.codVerifiedReceiptOtpRequired must be declared when availableOnCod is true"
}

# Drop-off locations URL required for counter drop-off model
# x-ion-condition: Required when pickupModel includes DROP_OFF_AT_COUNTER
deny[msg] {
  o := input.offerAttributes
  o.pickupModel[_] == "DROP_OFF_AT_COUNTER"
  not o.dropOffLocationsUrl
  msg := "offerAttributes.dropOffLocationsUrl is required when pickupModel includes DROP_OFF_AT_COUNTER"
}
```

### `logistics/resource/v1`

```rego
package ion.onix.conditional.logistics.resource

# DG fields required when dangerous goods are present
# x-ion-condition: Required when dangerous goods are present (dgClassification is set)
deny[msg] {
  r := input.resourceAttributes
  r.dgClassification
  not r.hazmatUnNumber
  msg := "resourceAttributes.hazmatUnNumber is required when dgClassification is present (IATA DGR / IMDG)"
}

deny[msg] {
  r := input.resourceAttributes
  r.dgClassification
  not r.hazmatProperShippingName
  msg := "resourceAttributes.hazmatProperShippingName is required for all DG shipments (IATA DGR)"
}
```

### `logistics/participant-logistics/v1`

```rego
package ion.onix.conditional.logistics.participant_logistics

# PPJK licence required for customs brokers
# x-ion-condition: Required when role=CUSTOMS_BROKER
deny[msg] {
  p := input.participantAttributes
  p.role == "CUSTOMS_BROKER"
  not p.ppjkLicenceNumber
  msg := "participantAttributes.ppjkLicenceNumber is required for CUSTOMS_BROKER role (PMK 219/PMK.04/2019)"
}

# Driver licence required for driver participants
# x-ion-condition: Required when role=DRIVER
deny[msg] {
  p := input.participantAttributes
  p.role == "DRIVER"
  not p.driverLicenceNumber
  msg := "participantAttributes.driverLicenceNumber is required for DRIVER role participants"
}
```

---

## Cross-schema policies

Some conditions span multiple schema bags and cannot be expressed within any single pack's Rego file. These are handled by ONIX's cross-schema validator which receives the full message envelope.

Fields marked `x-ion-cross-schema-condition: true` in the schemas are exclusively enforced here.

```rego
package ion.onix.conditional.cross_schema

# Logistics participant details required for cross-border contracts
# when the trade contract has an international delivery address
deny[msg] {
  contract := input.message.order.contractAttributes
  performance := input.message.order.performanceAttributes
  contract.deliveryAddress.country != "ID"
  not performance.customsBrokerParticipantId
  msg := "A customs broker participant is required for international (cross-border) deliveries"
}
```

---

## Implementation notes for ONIX developers

1. **Input shape.** The `input` object for each pack-level policy is the relevant `Attributes` container from the message (e.g. `resourceAttributes`, `offerAttributes`). The cross-schema policy receives the full Beckn message envelope.

2. **Policy loading.** ONIX loads all `.rego` files under `policies/onix-conditional/` at startup. Each package namespace maps to a specific pack. New packs require a new package file — no changes to the ONIX codebase.

3. **Error format.** Each `deny` message is returned to the API caller as an `ION-4xx` error with the message text as the `error.message` field. Include the regulatory citation in every `msg` string where applicable.

4. **Adding a new condition.** When adding a new `x-ion-condition:` annotation to a schema field, a corresponding Rego `deny` rule MUST be added to this file in the same PR. The annotation is documentation; the Rego rule is enforcement. Both travel together.

5. **Testing.** Each `deny` rule MUST have a corresponding OPA unit test in `policies/onix-conditional/tests/`. The test MUST cover: (a) the condition active + field missing → deny fires, (b) the condition active + field present → deny does not fire, (c) the condition inactive → deny does not fire regardless of field presence.
