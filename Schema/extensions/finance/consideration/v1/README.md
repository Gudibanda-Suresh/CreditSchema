# consideration/v1 — Credit Consideration Attributes

Attaches to: **`Consideration.considerationAttributes`**

Schema: `LoanConsideration` — the monetary value exchanged under the credit contract (principal disbursed plus fees).

## Fields

| Field | Type | Required | Description |
|---|---|---|---|
| `approvedPrincipal` | number | Yes | Disbursed principal in IDR |
| `currency` | string | Yes | ISO 4217 code (default: IDR) |
| `totalFeesIDR` | number | No | Total upfront fees at akad signing |
| `aprEquivalent` | number | No | All-in annualised cost of credit % |

## Regulatory note

`aprEquivalent` is mandatory for RIPLAY disclosure per **POJK 6/POJK.07/2022**.
It must equal the value in `offer/v1` `CreditOfferAttributes.totalFees.aprEquivalent` at the time of offer acceptance.
