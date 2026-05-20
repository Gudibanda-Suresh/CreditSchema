# ION Logistics Variant — Multi-Stop (multi-stop)

Extends the parcel and hyperlocal patterns to support a single pickup with multiple sequential drop points served by one rider or one AWB booking.

## What this covers

A single `Commitment` carries one shipment booking against the LSP. Within that booking, the physical journey visits three or more locations in a declared sequence: one pickup stop followed by two or more drop stops. Each stop is represented as a separate `Performance` record linked back to the parent `Commitment` via `commitmentIds`.

Typical use cases:

- A P2P hyperlocal LSP (e.g. Pidge) dispatches a single rider to collect a parcel from a warehouse and deliver to two or more recipients in sequence within a single booking.
- A same-day parcel service consolidates a multi-recipient B2B delivery under one AWB with stops declared at booking time.
- A merchant-initiated reverse pickup visits multiple return addresses in one run.

## Relationship to other patterns and variants

- Applies to: `parcel` (P2P_MULTI_STOP routing topology), `hyperlocal`
- Does NOT apply to: `freight`, `roro`, `warehouse`, `cross-border`
- Compatible with: `cod` (each stop may independently carry a `codAmount`), `cancellation`, `cross-cutting`, `exception`
- Incompatible with: `ekyc` (multi-stop is always B2C or pre-cleared B2B)

## Performance record model

Each stop in the journey MUST be expressed as a distinct `Performance` object with a unique `id`. The stops MUST be linked to the parent commitment via `commitmentIds`. The `sequence` field inside `performanceAttributes` declares the order of execution (1 = first, ascending). `packageIds` inside `performanceAttributes` declares which packages from the `commitmentAttributes.packages[]` manifest are relevant to that stop.

```
Contract
  └── Commitment (cmmt-001)
        ├── Performance id=stop-001  sequence=1  type=PICKUP    packageIds=[pkg-001, pkg-002]
        ├── Performance id=stop-002  sequence=2  type=DROP      packageIds=[pkg-002]
        └── Performance id=stop-003  sequence=3  type=DROP      packageIds=[pkg-001]
```

## Tariff model

Multi-stop surcharges are declared as breakup lines of type `MULTI_STOP_FEE` in `considerationAttributes.breakup[]`. One fee line per additional stop beyond the first drop is typical, but LSPs may declare a flat per-booking fee. COD collection at individual stops is expressed via `codAmount` on each drop `performanceAttributes`. The per-stop COD handling fee is broken out as a `COD_FEE` breakup line with `stopRef` pointing to the relevant `Performance.id`.

## Verification at handover

Each stop MAY declare a `verification` object inside `performanceAttributes` with a `type` of `OTP`, `BARCODE`, or `PHOTO`. The type declared at `on_confirm` is the binding handover method for that stop. OTP values and barcode strings are populated at `on_confirm` for immediately-confirmed bookings, or delivered via a subsequent `on_status[READY_FOR_PICKUP]` event.

## Key fields introduced by this variant

`performanceAttributes.sequence`, `performanceAttributes.type` (PICKUP | DROP | RETURN), `performanceAttributes.packageIds[]`, `performanceAttributes.verification`, `performanceAttributes.codAmount`, `performanceAttributes.awbNumber` (shared across all stops of a single booking), `performanceAttributes.agentDetails` (populated at `on_confirm` for pre-assigned riders), `performanceAttributes.executionLogs[]` (populated by BPP on status push), `considerationAttributes.breakup[].titleType = MULTI_STOP_FEE`, `considerationAttributes.breakup[].stopRef`.
