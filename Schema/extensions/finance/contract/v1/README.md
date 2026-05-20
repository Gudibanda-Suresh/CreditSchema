# CreditContractAttributes — v1 Property Table

**Pack ID:** `ion-finance-credit-contract-v1`  
**ION IRI:** `https://schema.ion.id/finance/credit-contract/v1#`  
**Beckn flow:** `on_confirm` → `message.contract`

This is a **multi-slot pack** — each sub-schema attaches to a different Beckn slot:

| Sub-schema | Beckn slot |
|------------|-----------|
| `CreditContractAttributes` | `Contract.contractAttributes` |
| `Borrower` | `Contract.participants[].participantAttributes` |
| `Lender` | `Contract.participants[].participantAttributes` |
| `AmortisationRow` | `Contract.settlements[].settlementAttributes` |
| `DisbursementPerformance` | `Contract.performance[].performanceAttributes` |
| `LoanConsideration` | `Contract.consideration[].considerationAttributes` |

---

## CreditContractAttributes (`Contract.contractAttributes`)

### Required

| Property | Type | Description |
|----------|------|-------------|
| `contractNumber` | string | Unique Perjanjian Kredit / Akad number assigned by lender |
| `loanAccountNumber` | string | Loan account number from core banking system |
| `agreementDate` | date | Date Akad Kredit was formally signed (YYYY-MM-DD) |
| `loanTerms` | ActiveLoanTerms | Frozen loan parameters — see sub-table below |
| `legalDocuments` | LegalDocuments | Document references — see sub-table below |

### Optional

| Property | Type | Description |
|----------|------|-------------|
| `maturityDate` | date | Final maturity date |
| `loanStatus` | enum | OJK SLIK collectibility grade (separate from Beckn Contract.status.code) |
| `restructuring` | LoanRestructuring | Present only if formally restructured per POJK 40/2019 |

### loanStatus Values

| Value | Meaning |
|-------|---------|
| `ACTIVE` | Current — all payments on time |
| `SPECIAL_MENTION` | 1–90 days overdue |
| `SUBSTANDARD` | 91–120 days overdue |
| `DOUBTFUL` | 121–180 days overdue |
| `LOSS` | > 180 days overdue |
| `SETTLED` | Fully repaid and closed |
| `RESTRUCTURED` | Terms modified per POJK 40/2019 |

---

## ActiveLoanTerms

### Required

| Property | Type | Description |
|----------|------|-------------|
| `approvedPrincipal` | number | Disbursed principal in IDR (plafond kredit per SLIK) |
| `effectiveRatePercent` | number | Annual rate % p.a. frozen at akad signing |
| `tenorMonths` | integer | Total approved tenure in months |
| `monthlyInstallmentIDR` | number | Fixed monthly installment in IDR |
| `monthlyDueDay` | integer | Day of month (1–31) installments are due |

### Optional

| Property | Type | Description |
|----------|------|-------------|
| `calculationMethod` | enum | ANNUITY \| FLAT \| EFFECTIVE |
| `yieldType` | enum | FIXED \| COMBINED \| FLOATING \| MARGIN \| NISBAH |
| `outstandingPrincipal` | number | Outstanding balance after most recent payment |
| `totalInterestPaid` | number | Cumulative interest / margin paid to date |
| `islamicContractType` | string | For Islamic products — akad type (e.g., MURABAHAH) |

---

## Borrower (`Contract.participants[]` where role = borrower)

### Required

| Property | Type | Description |
|----------|------|-------------|
| `fullName` | string | Full legal name as printed on KTP |
| `nik` | string | Nomor Induk Kependudukan — 16-digit national ID |

### Optional

| Property | Type | Description |
|----------|------|-------------|
| `npwp` | string | Nomor Pokok Wajib Pajak — Tax ID |
| `birthPlace` | string | City / regency of birth as on KTP |
| `birthDate` | date | Date of birth (YYYY-MM-DD) |
| `registeredAddress` | string | Full residential address as on KTP |
| `email` | string (email) | Active email for digital correspondence |
| `phone` | string | Mobile phone in E.164 format (+62XXXXXXXX) |

---

## Lender (`Contract.participants[]` where role = lender)

### Required

| Property | Type | Description |
|----------|------|-------------|
| `institutionName` | string | Full legal entity name registered with OJK |
| `ojkLicenseNumber` | string | OJK operating license number |

### Optional

| Property | Type | Description |
|----------|------|-------------|
| `branchOffice` | string | Branch handling this loan account |
| `relationshipManagerName` | string | Account Officer / Relationship Manager name |

---

## AmortisationRow (`Contract.settlements[]`)

### Required

| Property | Type | Description |
|----------|------|-------------|
| `installmentNumber` | integer | Sequence number (1-based) |
| `dueDate` | date | Due date for this installment (YYYY-MM-DD) |
| `installmentIDR` | number | Total installment due in IDR |

### Optional

| Property | Type | Description |
|----------|------|-------------|
| `principalIDR` | number | Principal component in IDR |
| `interestIDR` | number | Interest / margin component in IDR |
| `outstandingPrincipalIDR` | number | Remaining principal after this installment |
| `paymentStatus` | enum | UNPAID \| PAID \| OVERDUE \| PARTIAL |
| `actualPaymentDate` | date | Actual payment receipt date (PAID/OVERDUE/PARTIAL only) |

---

## DisbursementPerformance (`Contract.performance[]`)

### Required

| Property | Type | Description |
|----------|------|-------------|
| `disbursementMethod` | enum | BANK_TRANSFER \| DEVELOPER_TRANSFER \| DEALER_TRANSFER \| CASH \| VIRTUAL_ACCOUNT |

### Optional

| Property | Type | Description |
|----------|------|-------------|
| `disbursementDate` | date | Actual disbursement date (YYYY-MM-DD) |
| `targetBusinessDays` | integer | Target business days from docs to disbursement |

---

## LoanConsideration (`Contract.consideration[]`)

### Required

| Property | Type | Description |
|----------|------|-------------|
| `approvedPrincipal` | number | Approved principal amount in IDR |
| `currency` | string | ISO 4217 code (always IDR for domestic credit) |

### Optional

| Property | Type | Description |
|----------|------|-------------|
| `totalFeesIDR` | number | Total upfront fees paid at akad signing |
| `aprEquivalent` | number | All-in annualised cost of credit % (mandatory RIPLAY disclosure per POJK 6/2022) |

---

## LegalDocuments

### Required

| Property | Type | Description |
|----------|------|-------------|
| `creditAgreementUrl` | uri | URL to signed Akad Kredit PDF (HTTPS, stable 10 years) |

### Optional

| Property | Type | Description |
|----------|------|-------------|
| `collateralCertificateUrl` | uri | URL to collateral certificate (SHM/SHGB with HT, or BPKB with Fidusia) |
| `notarialDeedNumber` | string | Nomor Akta Notaris / PPAT reference |
| `mortgageLienNumber` | string | Hak Tanggungan registration number from BPN |
| `riplayDocumentUrl` | uri | URL to signed RIPLAY per POJK 6/POJK.07/2022 |
| `approvalLetterUrl` | uri | URL to SPPP approval letter per POJK 6/2022 |
| `eMateraiSerial` | string | e-Meterai serial number per UU No. 10/2020 |

---

## LoanRestructuring (conditional — present only when loanStatus = RESTRUCTURED)

| Property | Type | Description |
|----------|------|-------------|
| `restructuringDate` | date | Date restructuring agreement was signed |
| `restructuringType` | enum | TENOR_EXTENSION \| RATE_REDUCTION \| ARREARS_WAIVER \| PRINCIPAL_CONVERSION \| NEW_FACILITY \| COMBINED |
| `regulatoryBasis` | string | OJK regulation authorising this restructuring |
| `newTenorMonths` | integer | New total tenor in months after restructuring |
| `newRatePercent` | number | New annual rate % p.a. after restructuring |
| `restructuringReason` | string | Plain-language reason (audit trail per POJK 40/2019) |
