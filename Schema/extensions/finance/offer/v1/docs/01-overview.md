# Offer Extension — Conceptual Overview

## Purpose

The `offer` extension carries a lender's **per-applicant credit decision** — the exact, time-limited approval issued after evaluating a specific borrower's profile. It attaches to `Offer.offerAttributes` within `Contract.commitments[].offer` on `on_select`.

### CreditProduct vs CreditApproval

| Dimension | `CreditProduct` (resource) | `CreditApproval` (offer) |
|---|---|---|
| Scope | Product template for all borrowers | Decision for one specific borrower |
| Rate | Range: `7.5% – 14%` | Exact: `9.5%` |
| Principal | Range: `IDR 50jt – 2M` | Exact: `IDR 450,000,000` |
| Tenor | Range: `12 – 240 months` | Exact: `240 months` |
| Validity | Permanent until product changes | Time-limited (`validUntil`) |

---

## Offer Lifecycle

```
UNDER_EVALUATION → APPROVED ──────→ [borrower confirms] → LoanAgreement
                 ↘ CONDITIONALLY_APPROVED → [conditions met] → APPROVED
                 ↘ REJECTED
                 ↘ EXPIRED (if validUntil passes before confirmation)
```

### UNDER_EVALUATION
The lender has received the borrower's application and is running credit assessment (SLIK inquiry, income verification, DTI calculation). The BAP should display a pending state.

### APPROVED
Full approval with no outstanding conditions. All fields in `CreditOfferAttributes` are populated. `validUntil` is typically 14–30 calendar days from issue date.

### CONDITIONALLY_APPROVED
Approval subject to conditions listed in `additionalConditions[]`. Common conditions:
- Submit original SHM (land certificate) free of encumbrance
- Provide payslips for the last 3 months
- Open a savings account at the lender for auto-debit

### REJECTED
The application did not meet the lender's credit criteria. `additionalConditions[]` must contain the rejection reason per **POJK 6/2022** (SPPP letter obligation). Common reasons: insufficient income, adverse SLIK, DTI exceeded, collateral shortfall.

### EXPIRED
The offer was not accepted before `validUntil`. A new application is required. The lender may or may not run a fresh SLIK inquiry for a re-application.

---

## Locked Price Principle

Once `on_select` returns `offerStatus: APPROVED`, all monetary fields are **locked**:

- `approvedPrincipal` — cannot be increased post-approval
- `effectiveRate.percentPerYear` — fixed for the stated `yieldType` / `fixedPeriodYears`
- `monthlyInstallment.totalIDR` — the auto-debit amount that will be debited monthly
- `totalFees` — all upfront costs are quoted and cannot increase before `on_confirm`

The locked price principle protects the borrower and is enforced by OJK's **RIPLAY** (Ringkasan Informasi Produk dan Layanan) disclosure requirement.

---

## APR Equivalent — POJK 6/2022 Obligation

`totalFees.aprEquivalent` is the **all-in annualised cost of credit** expressed as a percentage. It combines:

- Nominal interest rate
- All upfront fees amortised over the tenor
- Mandatory insurance premiums
- Stamp duty

Per **POJK 6/POJK.07/2022**, every lender must disclose the APR equivalent in the RIPLAY document before the borrower signs the Akad Kredit. The BAP must surface this field prominently in the offer comparison UI.

**Formula (simplified):**
```
APR = IRR([-totalDisbursement, PMT₁, PMT₂, ..., PMTₙ]) × 12
```
where `totalDisbursement = approvedPrincipal - totalUpfrontCostsIDR`.

---

## Internal Credit Score

`internalCreditScore` is optional and **must not be surfaced to the borrower** in the BAP UI. It is included for:
- Lender audit trails
- Regulatory examination
- BAP analytics (aggregated, anonymised)

`slikGrade` maps to OJK SLIK collectibility: `"1"` = Lancar (pass), `"2"` = Dalam Perhatian Khusus (watch).

---

## Disbursement Schedule

`disbursementSchedule.targetBusinessDays` measures the SLA from **complete documentation submission** to funds hitting the beneficiary account. Non-compliance triggers the lender compensation policy at `ion://policy/finance.penalty.lender-disbursement-delay`.

---

## Regulatory References

- **POJK 6/POJK.07/2022** — RIPLAY and SPPP disclosure obligations
- **POJK 40/POJK.03/2019** — Credit quality classification
- **SE OJK 13/SEOJK.07/2014** — APR-equivalent calculation method
