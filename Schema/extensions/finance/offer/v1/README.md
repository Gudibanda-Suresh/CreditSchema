# CreditOfferAttributes — v1 Property Table

**Pack ID:** `ion-finance-credit-offer-v1`  
**Attaches to:** `Offer.offerAttributes`  
**ION IRI:** `https://schema.ion.id/finance/credit-offer/v1#CreditOfferAttributes`  
**Beckn flow:** `on_select` / `on_init` → `message.contract.commitments[].offer.offerAttributes`

> **Key distinction:** `CreditResourceAttributes` carries product RANGES. `CreditOfferAttributes` carries EXACT values approved for a specific borrower.

---

## Required Properties

| Property | Type | Description |
|----------|------|-------------|
| `offerReference` | string | Unique lender-generated offer ID |
| `validUntil` | datetime | Offer expiry (ISO 8601 + WIB +07:00) |
| `offerStatus` | enum | OJK credit decision |
| `approvedPrincipal` | object | Exact approved principal `{value, currency}` |
| `approvedTenor` | object | Exact approved tenor `{value, unit}` |
| `effectiveRate` | object | Exact rate, method, yieldType |
| `monthlyInstallment` | object | First installment breakdown |
| `totalFees` | object | All upfront fees + APR (RIPLAY mandatory) |

## Optional Properties

| Property | Type | Condition |
|----------|------|-----------|
| `additionalConditions` | string[] | CONDITIONALLY_APPROVED / REJECTED |
| `disbursementSchedule` | object | All products |
| `internalCreditScore` | object | Lender-proprietary, not disclosed to borrower |

---

## offerStatus Values

| Value | Meaning |
|-------|---------|
| `UNDER_EVALUATION` | Credit evaluation in progress; poll status |
| `APPROVED` | Fully approved; borrower may confirm immediately |
| `CONDITIONALLY_APPROVED` | Approved pending conditions in `additionalConditions` |
| `REJECTED` | Rejected; reason in `additionalConditions`; SPPP letter required per POJK 6/2022 |
| `EXPIRED` | Offer window passed; borrower must re-apply |

---

## approvedPrincipal

| Field | Type | Notes |
|-------|------|-------|
| `value` | number | Exact IDR amount (rounded to nearest 1,000,000 for KPR) |
| `currency` | string | Must match CreditResourceAttributes currency |

## approvedTenor

| Field | Type | Notes |
|-------|------|-------|
| `value` | integer | Exact tenure value |
| `unit` | enum | DAYS \| MONTHS \| YEARS |

## effectiveRate

| Field | Type | Notes |
|-------|------|-------|
| `percentPerYear` | number | **Required.** Annual rate % (frozen in CreditContractAttributes) |
| `method` | enum | **Required.** ANNUITY \| FLAT \| EFFECTIVE |
| `yieldType` | enum | FIXED \| FLOATING \| COMBINED \| MARGIN \| NISBAH |
| `fixedPeriodYears` | integer | For COMBINED products |
| `floatingRateAfterFixedPercent` | number | For COMBINED — rate after fixed period |

## monthlyInstallment

| Field | Type | Notes |
|-------|------|-------|
| `totalIDR` | number | **Required.** Auto-debit monthly payment |
| `principalIDR` | number | ANNUITY: increases monthly; FLAT: constant |
| `interestIDR` | number | ANNUITY: decreases monthly; FLAT: constant |

## totalFees

| Field | Type | Notes |
|-------|------|-------|
| `provisionFeeIDR` | number | IDR 0 for KUR/FLPP |
| `administrationFeeIDR` | number | IDR 0 for government programs |
| `appraisalFeeIDR` | number | KJPP fee for secured products |
| `notaryFeeIDR` | number | PPAT/notary estimate |
| `lifeInsurancePremiumIDR` | number | Single premium for full tenor |
| `annualFireInsuranceIDR` | number | Mandatory for KPR (PBI 12/2010) |
| `stampDutyIDR` | integer | Default 10,000 per document |
| `totalUpfrontCostsIDR` | number | Sum of all upfront costs (RIPLAY item) |
| `aprEquivalent` | number | **Mandatory RIPLAY disclosure** per POJK 6/POJK.07/2022 |

## disbursementSchedule

| Field | Type | Notes |
|-------|------|-------|
| `targetBusinessDays` | integer | KPR ≤ 14, KUR ≤ 3, Paylater near-instant |
| `disbursementMethod` | enum | BANK_TRANSFER \| DEVELOPER_TRANSFER \| DEALER_TRANSFER \| CASH \| VIRTUAL_ACCOUNT |

## internalCreditScore

| Field | Type | Notes |
|-------|------|-------|
| `score` | integer | Lender proprietary scale |
| `category` | enum | EXCELLENT \| GOOD \| FAIR \| POOR |
| `slikGrade` | enum | `"1"` = Lancar \| `"2"` = Special Mention |
