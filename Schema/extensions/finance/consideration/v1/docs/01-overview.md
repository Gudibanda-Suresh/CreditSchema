# Consideration Extension — Conceptual Overview

## Purpose

The `consideration` extension carries the **monetary exchange record** for a credit contract — the principal disbursed, total upfront fees, and the all-in APR equivalent. It attaches to `Consideration.considerationAttributes` within `Contract.consideration[]`.

In credit, consideration is asymmetric: the lender disburses principal (a monetary asset) and the borrower repays principal plus interest over time. The `LoanConsideration` captures the upfront monetary facts at akad signing.

---

## Relationship to Other Extensions

| Extension | Data | Timing |
|---|---|---|
| `offer` → `totalFees` | Quoted fees before signing | `on_select` |
| `consideration` → `totalFeesIDR` | Confirmed fees at signing | `on_confirm` |
| `contract` → `loanTerms` | Frozen principal and rate | `on_confirm` |
| `settlement` → rows | Ongoing repayment schedule | `on_confirm` + updates |

The `consideration` record confirms what was actually exchanged at the point of contract execution. It must match the RIPLAY document signed by the borrower.

---

## APR Equivalent — The Core Disclosure Number

`aprEquivalent` is the **all-in annualised percentage cost of credit** per **POJK 6/POJK.07/2022**. It is the single most important consumer protection number in the credit offer and contract.

### What is included in APR equivalent

- Nominal interest rate (or margin/nisbah for Islamic products)
- Administration and provision fees (amortised over tenor)
- Mandatory insurance premiums (credit life, MRTA, property insurance, vehicle insurance)
- Stamp duty (amortised)
- Any other mandatory costs the borrower cannot avoid

### What is excluded

- Voluntary insurance top-ups
- Late payment penalties (contingent)
- Voluntary prepayment penalties (contingent)
- Third-party costs outside lender control (notary, KJPP appraisal) when itemised separately

### Display requirement

Per POJK 6/2022, the BAP must display `aprEquivalent` **prominently and in a font at least as large as the nominal rate** in all offer comparison screens. The RIPLAY document must print it in a standardised box.

---

## Currency

All amounts are in `IDR` (Indonesian Rupiah). The schema enforces `currency: "IDR"` as the default. No multi-currency credit products are supported in v1 of this schema (foreign currency mortgages are out of scope).

---

## Consideration vs Price

In trade (Beckn), "price" refers to the transaction amount. In credit:
- There is no single "price" — the cost of credit is spread across time
- `approvedPrincipal` is the amount disbursed (what the borrower receives)
- The total cost of credit = total repayments - approvedPrincipal (sum of all interest + fees)
- `aprEquivalent` expresses this total cost as an annualised percentage

---

## Regulatory References

- **POJK 6/POJK.07/2022** — RIPLAY mandatory APR disclosure
- **SE OJK 13/SEOJK.07/2014** — APR-equivalent calculation methodology for consumer credit
- **POJK 22/2023** — Consumer protection: fee transparency and no hidden charges
