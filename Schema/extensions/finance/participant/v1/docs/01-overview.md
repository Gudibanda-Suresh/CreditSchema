# Participant Extension — Conceptual Overview

## Purpose

The `participant` extension carries borrower and lender identity attributes that cannot be held in core Beckn fields. It attaches to `Participant.participantAttributes` inside `Contract.participants[]` after `on_confirm`.

There are two distinct participant types in a credit transaction:

| `@type` | Role | Required in |
|---|---|---|
| `ion:Borrower` | Individual or legal entity receiving the loan | Always |
| `ion:Lender` | Licensed financial institution extending the loan | Always |

---

## Borrower Identity

### NIK — Primary Key

`nik` (Nomor Induk Kependudukan) is the 16-digit national ID and the primary de-duplication key across all Indonesian credit systems. It is:

- Validated against **Dukcapil** (Ditjen Dukcapil, Ministry of Home Affairs) during KYC
- Reported to **OJK SLIK/iDEB** as the borrower identifier in monthly SLIK submissions
- Never reassigned — one NIK, one individual, for life

The NIK encodes province, city/regency, sub-district, sex, date of birth, and a sequential number. Do not store NIK in plaintext in logs; treat as PII under **UU PDP 2022**.

### NPWP — Tax Identification

`npwp` (Nomor Pokok Wajib Pajak) is required for:

- Loans above IDR 50 juta (per OJK KYC Tier 3)
- Business loans (all amounts)
- Mortgage and vehicle loan applications at BANK_UMUM and MULTIFINANCE

NPWP validation is performed against the **DJP Online** (Direktorat Jenderal Pajak) API. Format: `XX.XXX.XXX.X-XXX.XXX`.

### KTP Address vs Domicile

`registeredAddress` holds the address printed on the KTP. Lenders may separately record a current domicile address (outside this schema) when it differs from the KTP address. The KTP address governs legal service of process.

---

## Lender Identity

### OJK License

`ojkLicenseNumber` references the formal operating license issued by OJK. License type varies by institution class:

| Institution Type | License Format Example |
|---|---|
| Bank Umum | `KEP-046/DIR/1992` |
| BPR | `KEP-BPR-2015-00123` |
| Multifinance | `KEP-0019/KMK.017/1992` |
| P2P (LPBBTI) | `S-0008/NB.111/2021` |

The BAP must verify the lender's OJK license is active before presenting offers to borrowers.

---

## Privacy and Data Protection

All participant fields contain personal data regulated by **UU 27/2022 tentang Perlindungan Data Pribadi (UU PDP)**. Key obligations:

- Borrower must provide explicit, granular consent before any field is collected
- Lender must not share borrower data with third parties without consent, except to OJK SLIK and other statutory reporting bodies
- Retention: loan participant data must be retained for 10 years post-loan-closure (OJK archiving rules)
- NIK, birthDate, and `registeredAddress` together constitute **sensitive PII** — apply field-level encryption at rest

---

## Lifecycle

Participant records are created at `on_confirm` and are **immutable** for the life of the contract. Any change to legal identity (e.g., name change by marriage) triggers a formal amendment process outside the ION protocol, with the updated record captured in a new `Participant` entry.

---

## Regulatory References

- **UU 27/2022** — Undang-Undang Perlindungan Data Pribadi
- **OJK POJK 12/2018** — Layanan Perbankan Digital (eKYC)
- **OJK POJK 13/2018** — Inovasi Keuangan Digital (fintech KYC)
- **POJK 18/2017** — SLIK reporting (NIK as borrower key)
- **Permendagri tentang Dukcapil** — NIK validation authority
