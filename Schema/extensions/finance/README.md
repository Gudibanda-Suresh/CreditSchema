# Schema Extensions — Finance Sector

**Sector:** finance  
**Path:** `schema/extensions/finance/`

These schema packs define ION-specific attributes attached to Beckn 2.0 protocol objects for all Finance (FIN-XX) transactions.

## Extensions

| Extension | Beckn Attachment | Stage | Description |
|---|---|---|---|
| [participant](participant/v1/) | `Participant.participantAttributes` | on_confirm | Borrower & lender identity (NIK, NPWP, OJK license) |
| [resource](resource/v1/) | `Resource.resourceAttributes` | on_discover / on_search | Credit product template with ranges and eligibility |
| [offer](offer/v1/) | `Offer.offerAttributes` | on_select | Per-applicant credit decision with exact approved values |
| [contract](contract/v1/) | `Contract.contractAttributes` | on_confirm | Immutable signed loan agreement and SLIK status |
| [settlement](settlement/v1/) | `Settlement.settlementAttributes` | on_confirm + updates | Amortisation schedule rows |
| [performance](performance/v1/) | `Performance.performanceAttributes` | on_confirm + on_status | Disbursement execution tracking |
| [consideration](consideration/v1/) | `Consideration.considerationAttributes` | on_confirm | Principal, fees, APR equivalent (POJK 6/2022) |

## Applicable To

FIN-02 (Lending & Consumer Credit) — CRC: `FNC-lending`

Product types (`resourceAttributes.productType`): `KTA` · `KPR` · `KKB` · `KUR_UMKM` · `REVOLVING` · `BNPL`
