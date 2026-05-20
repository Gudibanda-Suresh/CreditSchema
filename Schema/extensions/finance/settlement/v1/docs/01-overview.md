# Settlement Extension — Conceptual Overview

## Purpose

The `settlement` extension carries a **single row of the loan amortisation schedule**. Each `Settlement` in `Contract.settlements[]` corresponds to one installment period. `settlementAttributes["@type"] = "ion:AmortisationRow"`.

The full schedule is transmitted at `on_confirm` and updated via `on_status` / `on_update` as payments are received.

---

## One Settlement Per Installment

Each `AmortisationRow` represents exactly one installment period (typically one month). A 36-month loan produces 36 `Settlement` objects in `Contract.settlements[]`.

Beckn `Settlement.status` maps to `paymentStatus`:

| Beckn status | `paymentStatus` | Meaning |
|---|---|---|
| `DRAFT` | `UNPAID` | Future installment, not yet due |
| `COMMITTED` | `PARTIAL` | Partial payment received; remainder outstanding |
| `COMPLETE` | `PAID` | Full payment received on or before due date |
| `COMPLETE` | `OVERDUE` | Full payment received after due date |

---

## Amortisation Methods

### ANNUITY (Most common — KPR, KKB, KTA at banks)

Constant total installment. Principal component grows each period; interest component shrinks.

```
Period 1:  installmentIDR = 4,193,000  →  principal = 630,500  +  interest = 3,562,500
Period 2:  installmentIDR = 4,193,000  →  principal = 635,490  +  interest = 3,557,510
...
Period 240: installmentIDR = 4,193,000  →  principal ≈ 4,160,000  +  interest ≈ 33,000
```

`outstandingPrincipalIDR` decreases slowly in early periods (interest-heavy) and faster in later periods.

### FLAT (Common — Multifinance KKB, P2P KTA)

Constant principal component and constant interest component (calculated on original principal). Higher effective rate than ANNUITY for the same nominal rate.

```
Period 1:  principal = 1,388,889  +  interest = 500,000
Period 2:  principal = 1,388,889  +  interest = 500,000
...
```

### EFFECTIVE (Less common — some business loans)

Interest calculated on actual outstanding balance each period (equivalent to ANNUITY when compounded monthly). Used for revolving credit lines.

---

## Outstanding Balance After Each Row

`outstandingPrincipalIDR` is the **remaining principal balance** immediately after the installment for that row is applied. At the final installment row, this value should be `0` (or within rounding tolerance of `0`).

The lender's core banking system is the authoritative source. `outstandingPrincipalIDR` in the schema reflects the latest known value, updated on each `on_status` call.

---

## Overdue Handling

When `paymentStatus` transitions to `OVERDUE`:
1. The lender increments the DPD (Days Past Due) counter
2. SLIK collectibility may be reclassified (see `contract/v1`)
3. Late payment penalties accrue per `ion://policy/finance.penalty.borrower-late-payment`
4. The `actualPaymentDate` is recorded when payment is eventually received

---

## Partial Payment

`PARTIAL` indicates the borrower paid less than the full `installmentIDR`. The shortfall accumulates. Lenders must record the partial amount received and the remaining shortfall separately in their core banking system (outside this schema).

---

## Regulatory References

- **POJK 40/POJK.03/2019** — Amortisation schedule as supporting evidence for SLIK quality
- **POJK 18/2017** — SLIK monthly reporting (outstanding balance, DPD per installment)
- **PSAK 71** — Expected Credit Loss (ECL) calculation using amortised cost schedule
