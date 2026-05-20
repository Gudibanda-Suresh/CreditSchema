# ION Finance Sector Extension Packs

**Sector:** `ion:finance`  
**Path:** `schema/extensions/finance/`

These schema packs define ION-specific attributes attached to Beckn 2.0 protocol objects for all Finance (FIN-XX) transactions.

## Extensions

| Extension | Beckn Attachment | Stage | Description |
|---|---|---|---|
| [resource/v1](resource/v1/) | `Resource.resourceAttributes` | on_discover | Credit product template — ranges, eligibility, cost-of-fund, regulatory declarations |
| [offer/v1](offer/v1/) | `Offer.offerAttributes` | on_select | Per-applicant credit decision with exact approved values |
| [contract/v1](contract/v1/) | `Contract.contractAttributes` | on_confirm | Immutable signed loan agreement and SLIK status |
| [participant/v1](participant/v1/) | `Participant.participantAttributes` | on_confirm | Borrower and lender identity (NIK, NPWP, OJK license) |
| [settlement/v1](settlement/v1/) | `Settlement.settlementAttributes` | on_confirm + updates | Amortisation schedule rows |
| [performance/v1](performance/v1/) | `Performance.performanceAttributes` | on_confirm + on_status | Disbursement execution tracking |
| [consideration/v1](consideration/v1/) | `Consideration.considerationAttributes` | on_confirm | Principal, fees, APR equivalent (POJK 6/2022) |

## Applicable To

FIN-02 (Lending & Consumer Credit) — CRC: `FNC-lending`

Product types (`resourceAttributes.productType`): `KTA` · `KPR` · `KKB` · `KUR_UMKM` · `REVOLVING` · `BNPL`
