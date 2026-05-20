# Performance Extension — Conceptual Overview

## Purpose

The `performance` extension tracks **loan disbursement execution** — the actual transfer of approved funds to the correct beneficiary. It attaches to `Performance.performanceAttributes` within `Contract.performance[]`.

`Performance.status.code = COMPLETE` once funds have been confirmed as transferred.

---

## When Performance Is Created

A `DisbursementPerformance` object is created at `on_confirm` with:
- `disbursementMethod` — how funds will be transferred
- `targetBusinessDays` — the SLA commitment from the lender

`disbursementDate` is populated on the `on_status` call that carries `status.code = DISBURSED`.

---

## Disbursement Methods

| `disbursementMethod` | Used For | Beneficiary | Rail |
|---|---|---|---|
| `BANK_TRANSFER` | KTA, business loans | Borrower's bank account | BI-FAST / RTGS / SKN |
| `DEVELOPER_TRANSFER` | KPR (new property) | Property developer | RTGS via notary escrow |
| `DEALER_TRANSFER` | KKB | Authorised vehicle dealer | BI-FAST / RTGS |
| `CASH` | BPR micro loans | Borrower (cash at counter) | Over-the-counter |
| `VIRTUAL_ACCOUNT` | BNPL | Merchant settlement | Virtual account / RTGS |

---

## SLA Measurement

`targetBusinessDays` is the lender's published commitment from **complete documentation submission** to funds transfer. The clock:

- **Starts** when the lender confirms all required documents are received and KYC is complete
- **Pauses** if the lender requests additional documents from the borrower
- **Restarts** when the borrower responds with the requested documents
- **Stops** when `disbursementDate` is confirmed

Breach of `targetBusinessDays` triggers borrower compensation per `ion://policy/finance.penalty.lender-disbursement-delay`.

### Typical SLAs by Product

| Product | `targetBusinessDays` |
|---|---|
| KTA (fully digital) | 1–3 |
| KKB (vehicle loan) | 2–5 |
| Business loan | 3–7 |
| KPR (mortgage, ready stock) | 7–14 |
| KPR (indent / under construction) | 14–30 (per construction milestone) |
| BNPL | < 5 minutes (real-time) |

---

## Notary Escrow Flow

For KPR, `disbursementMethod = DEVELOPER_TRANSFER` and funds pass through a notary escrow account before reaching the developer. The Beckn performance status progression:

```
on_confirm → [status: PENDING]
on_status  → [FUNDS_IN_ESCROW]   disbursementDate = escrow credit date
on_status  → [AJB_SIGNED]
on_status  → [APHT_REGISTERED]
on_status  → [DISBURSED]         funds released from escrow to developer
```

---

## Regulatory References

- `ion://policy/finance.disbursement.direct-to-account` — BI-FAST/RTGS SLA rules
- `ion://policy/finance.disbursement.notary-escrow` — Mortgage escrow requirements
- `ion://policy/finance.disbursement.dealer-direct` — KKB dealer disbursement
- `ion://policy/finance.penalty.lender-disbursement-delay` — SLA breach compensation
- **OJK POJK 12/2018** — Digital banking disbursement standards
