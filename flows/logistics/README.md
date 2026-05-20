# ION Logistics Sector Flows — Sector B

Movement and storage of physical goods. Every transaction follows a **Pattern**. **Variants** modify specific steps without changing the overall shape.

## Path notation

```
logistics / category-code / resource-category-code / pattern / variant
```

Example: `logistics / LOG-01 / LGC-lastmile / parcel / cash-on-delivery`

## Patterns

| Pattern | What it covers |
|---|---|
| [`parcel`](patterns/parcel/v1/pattern.yaml) | Hub-routed parcel delivery — domestic B2C and B2B (JNE, J&T, SiCepat). **Reference pattern.** All others delta from here. |
| [`hyperlocal`](patterns/hyperlocal/v1/pattern.yaml) | Sub-day rider-based on-demand delivery (GoSend, GrabExpress, Paxel) |
| [`freight`](patterns/freight/v1/pattern.yaml) | Capacity-based freight — FTL/LTL, FCL/LCL, air cargo, rail, river, bulk |
| [`roro`](patterns/roro/v1/pattern.yaml) | Self-accompanied cargo on Ro-Ro ferries (Java-Bali, Java-Sumatra strait crossings) |
| [`cross-border`](patterns/cross-border/v1/pattern.yaml) | International shipment with Bea Cukai customs clearance |
| [`warehouse`](patterns/warehouse/v1/pattern.yaml) | Storage, FBA-style fulfilment, inventory management, value-added logistics |

**`parcel` is the reference pattern.** Every other logistics pattern is defined as a delta from it.

## Variants

| Variant | When it activates |
|---|---|
| [`during-transaction`](variants/during-transaction/v1/variant.yaml) | Payment method, FWA activation, eKYC — set during select→confirm window |
| [`cancellation`](variants/cancellation/v1/variant.yaml) | Cancellation at any stage: pre-pickup, seller-initiated, in-transit, TAT breach |
| [`cash-on-delivery`](variants/cash-on-delivery/v1/variant.yaml) | COD collection at delivery, refusal, partial collection, remittance |
| [`cold-chain`](variants/cold-chain/v1/variant.yaml) | Temperature monitoring, excursion alerts, cold-chain integrity proof |
| [`customs-clearance`](variants/customs-clearance/v1/variant.yaml) | Export/import customs, duty calculation, clearance events, hold resolution |
| [`delivery-attempts`](variants/delivery-attempts/v1/variant.yaml) | Failed delivery: NDR, reschedule, address change, self-pickup conversion, auto-RTO |
| [`ekyc`](variants/ekyc/v1/variant.yaml) | B2B identity verification at first transaction without active FWA |
| [`exception`](variants/exception/v1/variant.yaml) | Damage claim, loss claim, address unserviceable, pickup failed |
| [`incident`](variants/incident/v1/variant.yaml) | Accident, breakdown, robbery, force majeure during transit |
| [`reverse`](variants/reverse/v1/variant.yaml) | Return/reverse logistics: simple, with QC, freight, cross-border, inventory release |
| [`rts-handoff`](variants/rts-handoff/v1/variant.yaml) | Ready-to-ship signal from seller or warehouse to transport provider |
| [`value-added-services`](variants/value-added-services/v1/variant.yaml) | QC, kitting, labelling, packaging, returns processing at warehouse |
| [`weight-dispute`](variants/weight-dispute/v1/variant.yaml) | Weight discrepancy between declared and measured — raised, countered, resolved |
| [`cross-cutting`](variants/cross-cutting/v1/variant.yaml) | Available on all patterns: tracking, support, rating, reconciliation, raise escalation |
