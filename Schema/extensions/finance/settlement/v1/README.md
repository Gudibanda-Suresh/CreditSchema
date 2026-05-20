# settlement/v1 — Credit Settlement Attributes

Attaches to: **`Settlement.settlementAttributes`**

One `AmortisationRow` per installment. Each row in `Contract.settlements[]` holds one entry in the repayment schedule.

## Fields

| Field | Type | Required | Description |
|---|---|---|---|
| `installmentNumber` | integer | Yes | 1-based sequence number |
| `dueDate` | date | Yes | Payment due date (YYYY-MM-DD) |
| `installmentIDR` | number | Yes | Total amount due in IDR |
| `principalIDR` | number | No | Principal component in IDR |
| `interestIDR` | number | No | Interest / margin component in IDR |
| `outstandingPrincipalIDR` | number | No | Remaining principal after this row |
| `paymentStatus` | enum | No | UNPAID / PAID / OVERDUE / PARTIAL |
| `actualPaymentDate` | date | No | Date payment was received |

## `paymentStatus` values

| Value | Beckn Settlement.status | Meaning |
|---|---|---|
| `UNPAID` | DRAFT | Not yet due or not paid |
| `PAID` | COMPLETE | Paid in full on or before due date |
| `OVERDUE` | COMPLETE | Paid after due date |
| `PARTIAL` | COMMITTED | Partially paid |
