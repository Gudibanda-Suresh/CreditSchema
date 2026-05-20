# performance/v1 — Credit Performance Attributes

Attaches to: **`Performance.performanceAttributes`**

Schema: `DisbursementPerformance` — tracks the execution of fund disbursement after the loan contract is confirmed.

## Fields

| Field | Type | Required | Description |
|---|---|---|---|
| `disbursementMethod` | enum | Yes | Transfer mechanism |
| `disbursementDate` | date | No | Actual disbursement date (YYYY-MM-DD) |
| `targetBusinessDays` | integer | No | SLA: target working days to disburse |

## `disbursementMethod` values

| Value | Description |
|---|---|
| `BANK_TRANSFER` | Direct transfer to borrower's savings account |
| `DEVELOPER_TRANSFER` | Transfer directly to property developer (KPR) |
| `DEALER_TRANSFER` | Transfer directly to vehicle dealer (KKB) |
| `CASH` | Cash disbursement at branch |
| `VIRTUAL_ACCOUNT` | Transfer via virtual account |

## Status mapping

`Performance.status.code = COMPLETE` once `disbursementDate` is set and funds confirmed.
