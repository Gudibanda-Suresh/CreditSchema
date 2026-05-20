# FIN-02 — Lending & Consumer Credit

**Sector:** finance  
**Category Code:** FIN-02  
**KBLI:** 64 (Jasa Keuangan, bukan Asuransi dan Dana Pensiun)

## Catalogue Resource Category (CRC)

| Code | Name | Status | Pattern | Pattern IRI |
|---|---|---|---|---|
| [FNC-lending](FNC-lending/README.md) | Lending & Credit | ACTIVE | `loan-application` | `ion://flow/finance/FIN-02/FNC-lending/loan-application/v1` |

Product-type differentiation (KTA, KPR, KKB, KUR_UMKM, REVOLVING, BNPL) is carried by `resourceAttributes.productType` on each published resource — not by separate CRC codes.

## Shared Variants

All variants live at this category level and are referenced by `FNC-lending/loan-application`:

| Variant | Active For | Description |
|---|---|---|
| [credit-assessment](variants/credit-assessment/v1/variant.yaml) | All | Hard SLIK inquiry, DTI check, credit scoring |
| [kyc-verification](variants/kyc-verification/v1/variant.yaml) | All | Dukcapil eKYC, liveness detection, face-match |
| [collateral](variants/collateral/v1/variant.yaml) | KPR, KKB, secured KUR_UMKM | Appraisal, SKMHT/APHT, fiducia BPKB, movable asset |
| [guarantor](variants/guarantor/v1/variant.yaml) | KPR, KUR_UMKM | Individual guarantor KYC, IJP KUR government guarantee |
| [disbursement](variants/disbursement/v1/variant.yaml) | All except REVOLVING | Direct, notary-escrow (KPR), dealer-direct (KKB), merchant-direct (BNPL) |
| [restructure](variants/restructure/v1/variant.yaml) | All except BNPL | Rescheduling, reconditioning, payment holiday |
| [early-settlement](variants/early-settlement/v1/variant.yaml) | All | Partial prepayment, full settlement, lien release |
| [delinquency](variants/delinquency/v1/variant.yaml) | All | DPD escalation, collections, write-off |
| [insurance](variants/insurance/v1/variant.yaml) | All | Credit life, MRTA, vehicle all-risk |
| [cross-cutting](variants/cross-cutting/v1/variant.yaml) | All | Status polling, support, reconcile, rate, cancel |

## Regulatory Basis

- OJK POJK 40/2019 — Credit quality and SLIK collectability classification
- OJK POJK 22/2023 — Consumer protection in financial services
- OJK POJK 18/2017 — SLIK reporting obligations
- OJK POJK 6/2022 — RIPLAY / APR equivalent disclosure
- OJK POJK 10/2022 — BNPL/Paylater (LPBBTI) licensing
- OJK POJK 12/2021 — Mortgage (KPR/KPA) product rules
- Permenko 1/2023 — KUR program rates and subsidy quota
- UU 4/1996 — Hak Tanggungan (property lien)
- UU 42/1999 — Jaminan Fidusia (fiducia)
