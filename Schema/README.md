# ION Finance Credit Extension Packs

Finance sector (Layer 5) attribute packs. These attach to Beckn-native `*Attributes` slots and apply to all credit-sector transactions on ION — KPR (mortgage), KKB (vehicle financing), KUR (government MSME credit), Kredit Multiguna (multipurpose consumer loan), kartu kredit (credit card), and paylater (BNPL).

## Packs in this directory

| Pack | Attaches to | What it covers |
|---|---|---|
| `resource/v1` | `beckn:Resource.resourceAttributes` | Credit product template — product type, principal range, tenor options, cost-of-fund mechanism (fixed/floating/combined/nisbah), repayment schedule type, fee structure, collateral requirements, borrower eligibility criteria, OJK/BI regulatory declarations, required documents, Islamic contract structure, government programme flags (KUR, FLPP), paylater configuration |
| `offer/v1` | `beckn:Offer.offerAttributes` | Per-applicant personalised credit decision — approved principal, approved tenor, exact interest/margin rate, monthly installment breakdown, total upfront fees, APR equivalent (POJK 6/2022 RIPLAY), offer validity, offer status (APPROVED / CONDITIONALLY_APPROVED / REJECTED), outstanding conditions, disbursement schedule, internal credit score |
| `contract/v1` | `beckn:Contract.contractAttributes` | Immutable signed loan agreement — Perjanjian Kredit / Akad Pembiayaan reference numbers, agreement date, maturity date, OJK SLIK collectibility status, frozen loan terms (ActiveLoanTerms), legal document URLs (akad, RIPLAY, HT/fidusia, e-Meterai), restructuring history per POJK 40/2019 |
| `participant/v1` | `beckn:Participant.participantAttributes` | Borrower and lender identity — Borrower: full name, NIK (16-digit KTP), NPWP, birth data, registered address, contact; Lender: institution name, OJK license number, branch, relationship manager |
| `settlement/v1` | `beckn:Settlement.settlementAttributes` | Amortisation schedule row — installment sequence number, due date, total installment IDR, principal component, interest/margin component, outstanding principal balance, payment status (UNPAID / PAID / OVERDUE / PARTIAL), actual payment date |
| `performance/v1` | `beckn:Performance.performanceAttributes` | Loan disbursement execution — disbursement date, target SLA in business days, disbursement method (BANK_TRANSFER / DEVELOPER_TRANSFER / DEALER_TRANSFER / CASH / VIRTUAL_ACCOUNT) |
| `consideration/v1` | `beckn:Consideration.considerationAttributes` | Monetary exchange under the contract — disbursed principal, currency, total upfront fees in IDR, APR equivalent for POJK 6/2022 RIPLAY disclosure |

## Cross-sector packs used by credit (Layer 4 — in `../core/`)

Credit does not redeclare these. Core packs apply directly:

| Core pack | Used for |
|---|---|
| `core/identity/v1` | Business identity (NPWP, NIB, PKP) for lender KYC and B2B credit |
| `core/participant/v1` | Role taxonomy (BORROWER, LENDER, GUARANTOR, NOTARY, APPRAISER, INSURANCE), address hierarchy, authorised signatory — see `participant/v1` here for NIK/NPWP credit-specific fields |
| `core/payment/v1` | Auto-debit setup, virtual account payment rails, instalment payment channel declarations, refund policies, credit-card payment processing fees |
| `core/localization/v1` | Language / currency / timezone — all credit products use IDR and WIB (+07:00) |
| `core/rating/v1` | Borrower and lender rating input post-disbursement |
| `core/raise/v1` | Network dispute escalation — payment disputes, document rejection appeals |
| `core/reconcile/v1` | Settlement reconciliation — instalment batch remittance fields |
| `core/support/v1` | Support tickets — complaint handling per POJK 6/2022 pengaduan nasabah |
| `core/tax/v1` | PPN on fees, PPh withholding on interest income, Bea Meterai (UU 10/2020) |

## History

- **v1.0.0** — Initial release. Previous standalone schema had a single multi-slot `credit-contract` pack covering all five contract-stage Beckn slots. Per the logistics extension convention, this was split into five dedicated packs: `contract/v1`, `participant/v1`, `settlement/v1`, `performance/v1`, `consideration/v1`. Sub-directory names changed from `credit-resource/`, `credit-offer/`, `credit-contract/` to the slot-aligned names `resource/`, `offer/`, `contract/` consistent with the logistics reference.

## Credit product availability model

Lenders publish product templates continuously; per-applicant decisions are not transmitted until `/select`. The BPP manages internal credit scoring and limit checks. What the lender publishes in `/on_search` is a product eligibility signal:

```yaml
borrowerCriteria:
  applicantTypes: [EMPLOYEE, SELF_EMPLOYED, PROFESSIONAL]
  minimumMonthlyIncomeIDR: 5000000
  minimumSLIKGrade: 1
  serviceRegions: ["ID-JB", "ID-JK", "ID-JT"]
  maxDSRPercent: 35
```

For principal-constrained products (KPR above IDR 5 billion, sindikasi), firm capacity confirmation happens at `/select` — the catalog declares eligibility intent; `/select` triggers the credit scoring engine and returns an `APPROVED` or `CONDITIONALLY_APPROVED` offer with exact terms.

## Product-type-specific attributes

Product-type-specific attributes (collateral for KPR/KKB, Islamic akad type for syariah products, government programme flags for KUR/FLPP, credit-limit model for kartu kredit, billing cycle for paylater) are conditional fields within the relevant pack. They are not separate product-type packs. Each pack's README documents which fields apply to which product type. Key conditional markers:

- `resource/v1`: `islamicFinance` object — present only when `financeType = SYARIAH`
- `resource/v1`: `governmentProgram` object — present only when `financeType` ∈ {`KUR`, `FLPP`, `KUR_SUPER_MICRO`}
- `resource/v1`: `paylaterService` object — present only when `productType = PAYLATER`
- `resource/v1`: `collateral.required = true` — triggers `collateral.maxLTVPercent` and `collateral.collateralTypes`
- `contract/v1`: `restructuring` object — present only when `loanStatus = RESTRUCTURED`

## Rate logic in catalog vs firm quote in `/select`

The catalog carries rate **logic** — yield type, rate ranges, reference index, spread, fixed-period years. It does not carry a per-applicant rate. Firm rates for a specific borrower-profile and principal-tenor combination are returned via `/on_select` in the `offer/v1` pack:

| Catalog (`resource/v1`) | Personalised offer (`offer/v1`) |
|---|---|
| `costOfFund.rateMinPercent` — `rateMaxPercent` | `effectiveRate.percentPerYear` — exact approved rate |
| `costOfFund.yieldType` (FIXED / FLOATING / COMBINED) | `effectiveRate.yieldType` — confirmed type for this borrower |
| `costOfFund.fixedPeriodYears` | `effectiveRate.fixedPeriodYears` — confirmed fixed window |
| `principalRange.minimum` — `maximum` | `approvedPrincipal.value` — exact approved amount |
| `tenor.minimum` — `maximum` | `approvedTenor.value` — exact approved tenor |
| `feeStructure.provisionFeePercent` | `totalFees.provisionFeeIDR` — exact rupiah amount |

This separation is how BAPs can rank lenders in search results (using catalog rate ranges) while honouring per-applicant pricing dynamics at offer stage.
