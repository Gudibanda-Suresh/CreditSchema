# Contract Extension — Conceptual Overview

## Purpose

The `contract` extension carries the **immutable signed loan agreement** and its associated state. It attaches to `Contract.contractAttributes` (the top-level contract record) and also contributes sub-schemas to `settlements[]`, `performance[]`, and `consideration[]`.

---

## Sub-Schemas

The credit contract extension defines five distinct object types, each placed in a different Beckn slot:

| Object | Beckn Attachment | Purpose |
|---|---|---|
| `CreditContractAttributes` | `Contract.contractAttributes` | Core agreement: contract numbers, frozen loan terms, legal documents, SLIK status |
| `ActiveLoanTerms` | Nested in `contractAttributes` | Rate, tenor, installment, and outstanding balance — immutable at akad |
| `LegalDocuments` | Nested in `contractAttributes` | Document URLs (akad PDF, RIPLAY, collateral cert, e-Meterai) |
| `LoanRestructuring` | Nested in `contractAttributes` (conditional) | Restructuring record per POJK 40/2019 |
| `AmortisationRow` | `Settlement.settlementAttributes` | One installment row; see settlement/v1 |
| `DisbursementPerformance` | `Performance.performanceAttributes` | Disbursement execution; see performance/v1 |
| `LoanConsideration` | `Consideration.considerationAttributes` | Monetary consideration; see consideration/v1 |

---

## Immutability of Loan Terms

`loanTerms` (the `ActiveLoanTerms` object) is **frozen at akad signing** (`agreementDate`). It cannot be modified unless a formal restructuring is executed per **POJK 40/POJK.03/2019**.

Fields that must not change post-akad:
- `approvedPrincipal`
- `effectiveRatePercent` (for FIXED yieldType)
- `calculationMethod`
- `tenorMonths`
- `monthlyInstallmentIDR`

If any of these change, a new `LoanRestructuring` record must be appended.

---

## Loan Status — OJK SLIK Collectibility

`loanStatus` maps directly to the OJK SLIK collectibility classification and is reported monthly:

| `loanStatus` | SLIK Kualitas | DPD Range | Meaning |
|---|---|---|---|
| `ACTIVE` | 1 — Lancar | 0 days | Current, no arrears |
| `SPECIAL_MENTION` | 2 — DPK | 1–90 days | Early watch; lender monitors |
| `SUBSTANDARD` | 3 — Kurang Lancar | 91–120 days | NPL threshold — provisions required |
| `DOUBTFUL` | 4 — Diragukan | 121–180 days | Higher provisions |
| `LOSS` | 5 — Macet | > 180 days | Full write-down; collections |
| `SETTLED` | — | N/A | Fully repaid; SLIK cleared |
| `RESTRUCTURED` | Per POJK 40 | Post-restructure | Min. DPK for 6 months post-restructure |

---

## Legal Documents

### creditAgreementUrl

The signed Akad Kredit / Perjanjian Kredit PDF must be:
- Accessible via HTTPS for 10 years post-loan-closure per OJK archiving requirements
- Affixed with `eMateraiSerial` (UU 10/2020 stamp duty, IDR 10,000 per document)
- Signed by all parties before disbursement

### riplayDocumentUrl

The **RIPLAY** (Ringkasan Informasi Produk dan Layanan) is a standardised one-page summary disclosing:
- Product name and type
- APR equivalent
- Total cost of credit simulation
- Key risks

Per **POJK 6/POJK.07/2022**, the borrower must sign the RIPLAY before or at the same time as the Akad Kredit. The signed copy must be retained and accessible.

### e-Meterai

`eMateraiSerial` is the electronic stamp duty serial from **PERURI** (Perum Percetakan Uang RI). Required on every credit agreement per **UU 10/2020**. Electronically affixed during digital signing, not separately printed.

---

## Restructuring Record

When `loanStatus` transitions to `RESTRUCTURED`, a `LoanRestructuring` object must be present with:
- `restructuringDate` — date the restructuring akad was signed
- `restructuringType` — one of the POJK 40/2019 Article 52 categories
- `regulatoryBasis` — the specific OJK regulation authorising this restructuring
- Updated `loanTerms` values in the parent object

SLIK reporting: a restructured loan must be classified at minimum `SPECIAL_MENTION` (DPK) and cannot be upgraded above that classification for 6 months from the restructuring date.

---

## Regulatory References

- **POJK 40/POJK.03/2019** — Penilaian Kualitas Aset Bank Umum (collectibility, restructuring)
- **POJK 6/POJK.07/2022** — RIPLAY, SPPP, and APR disclosure
- **UU 10/2020** — Bea Meterai (e-Meterai)
- **POJK 18/2017** — SLIK monthly reporting
- **PSAK 71** — IFRS 9 alignment for loan impairment (provisions)
