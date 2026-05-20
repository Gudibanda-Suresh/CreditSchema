# FNC-lending — Lending & Credit

**Sector:** finance  
**Category:** FIN-02 Lending & Consumer Credit  
**CRC Code:** `FNC-lending`  
**CRC Status:** ACTIVE  
**Pattern:** `loan-application`  
**Pattern IRI:** `ion://flow/finance/FIN-02/FNC-lending/loan-application/v1`

## What this CRC covers

`FNC-lending` is the single ION Catalogue Resource Category for all credit facilities under FIN-02. It enforces the `schema/extensions/finance/resource/v1` attribute pack on every loan product published to the catalog.

Product-type differentiation is carried by `resourceAttributes.productType`:

| `productType` | Indonesian Product | Collateral | Disbursement Target |
|---|---|---|---|
| `KTA` | Kredit Tanpa Agunan (unsecured personal loan) | None | Borrower bank account |
| `KPR` | Kredit Pemilikan Rumah / Apartemen (mortgage) | SKMHT/APHT (property) | Notary escrow |
| `KKB` | Kredit Kendaraan Bermotor (vehicle loan) | Fiducia / BPKB | Dealer direct |
| `KUR_UMKM` | KUR / Kredit UMKM (MSME loan) | Optional (IJP KUR waives for Mikro) | Borrower bank account |
| `REVOLVING` | Kartu Kredit / Kredit Rekening Koran | None | Credit limit drawdown |
| `BNPL` | Buy Now Pay Later / Paylater | None | Merchant direct |

## Pattern

| Pattern | IRI | Description |
|---|---|---|
| `loan-application` | `ion://flow/finance/FIN-02/FNC-lending/loan-application/v1` | Full Beckn transaction flow: catalog → search → select → init → confirm → fulfilment → post-fulfilment |

## Applicable Variants

All variants are shared at the FIN-02 category level. Variant activation is conditional on `productType`:

| Variant | Active For |
|---|---|
| [credit-assessment](../../variants/credit-assessment/v1/variant.yaml) | All product types |
| [kyc-verification](../../variants/kyc-verification/v1/variant.yaml) | All product types |
| [collateral](../../variants/collateral/v1/variant.yaml) | KPR, KKB, secured KUR_UMKM |
| [guarantor](../../variants/guarantor/v1/variant.yaml) | KPR, KUR_UMKM |
| [disbursement](../../variants/disbursement/v1/variant.yaml) | All except REVOLVING |
| [restructure](../../variants/restructure/v1/variant.yaml) | All except BNPL |
| [early-settlement](../../variants/early-settlement/v1/variant.yaml) | All |
| [delinquency](../../variants/delinquency/v1/variant.yaml) | All |
| [insurance](../../variants/insurance/v1/variant.yaml) | All |
| [cross-cutting](../../variants/cross-cutting/v1/variant.yaml) | All |

## Reference Path Examples

```
finance / FIN-02 / FNC-lending / loan-application / credit-assessment
finance / FIN-02 / FNC-lending / loan-application / collateral
finance / FIN-02 / FNC-lending / loan-application / disbursement
finance / FIN-02 / FNC-lending / loan-application / guarantor
finance / FIN-02 / FNC-lending / loan-application / early-settlement
finance / FIN-02 / FNC-lending / loan-application / delinquency
```

## Schema Attribute Pack

`FNC-lending` enforces `schema/extensions/finance/resource/v1` at `catalog/publish` time.  
Key mandatory fields on `Resource.resourceAttributes`:

- `productType` — one of `KTA | KPR | KKB | KUR_UMKM | REVOLVING | BNPL`
- `minLoanAmount`, `maxLoanAmount`
- `minTenureMonths`, `maxTenureMonths`
- `interestRateType` — `FLAT | EFFECTIVE | ANNUITY`
- `aprEquivalent` — mandatory per POJK 6/2022 RIPLAY disclosure
- `ojkProductCategory` — OJK product classification for rate cap routing
