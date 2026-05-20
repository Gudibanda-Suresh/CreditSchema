# ION Trade Sector Flows — Sector A

Physical goods commerce on ION. Every transaction follows a **Pattern** — the blueprint for the complete flow. **Variants** modify specific steps within a Pattern without changing the overall shape.

## Path notation

```
trade / category-code / resource-category-code / pattern / variant
```

Example: `trade / TRD-04 / TRC-health-beauty / storefront / delivery-time-kyc`

## Patterns

One Pattern per commerce scenario. Each defines the full API sequence from catalog publish through to reconciliation.

| Pattern | What it covers | Default happy-flow |
|---|---|---|
| [`storefront`](patterns/storefront/v1/pattern.yaml) | Standard consumer purchase — browse, select, buy. B2C physical goods. | Yes |
| [`made-to-order`](patterns/made-to-order/v1/pattern.yaml) | Item made after order placement. Food QSR, custom tailoring, bespoke furniture. | Yes |
| [`subscription`](patterns/subscription/v1/pattern.yaml) | Recurring delivery — customer confirms once, system auto-renews each cycle. | Yes |
| [`live-commerce`](patterns/live-commerce/v1/pattern.yaml) | Purchase during a live broadcast. TikTok Shop, Shopee Live, Tokopedia Play, OTT-embedded commerce. | Yes |
| [`digital-goods`](patterns/digital-goods/v1/pattern.yaml) | Electronically delivered items — pulsa, PLN token, game credits, vouchers, e-books. | Yes |
| [`business-procurement`](patterns/business-procurement/v1/pattern.yaml) | B2B purchase order. Wholesale, enterprise, government supply chains. | Prepaid |
| [`marketplace-inhouse`](patterns/marketplace-inhouse/v1/pattern.yaml) | Marketplace holds and fulfils inventory. Platform-managed stock. | Yes |
| [`marketplace-listed`](patterns/marketplace-listed/v1/pattern.yaml) | Seller lists on marketplace. Marketplace intermediates discovery; seller fulfils. | Yes |
| [`forward-auction`](patterns/forward-auction/v1/pattern.yaml) | Seller lists; buyers bid up. Highest bid at close wins. | Yes |
| [`reverse-auction`](patterns/reverse-auction/v1/pattern.yaml) | Buyer posts requirement; sellers bid down. Lowest qualifying bid wins. | Yes |
| [`cross-border`](patterns/cross-border/v1/pattern.yaml) | Transaction crosses an international border. Customs, duties, cross-currency settlement. | Yes |
| [`government`](patterns/government/v1/pattern.yaml) | Government and public sector procurement rules. | Yes |

**`storefront` is the reference pattern.** All other patterns are defined relative to it — read storefront first.

## Variants

Variants activate against a live Pattern transaction when a specific condition is met. A transaction can have multiple Variants active simultaneously.

| Variant | When it activates |
|---|---|
| [`during-transaction`](variants/during-transaction/v1/variant.yaml) | Payment method selection, fulfillment type, COD, on-network LSP — set during select→confirm window |
| [`cancellation`](variants/cancellation/v1/variant.yaml) | Any cancellation after confirm: buyer-initiated, seller-initiated, partial, force-cancel |
| [`returns`](variants/returns/v1/variant.yaml) | Return initiated after delivery: pickup, QC, refund, replacement, exchange |
| [`rto`](variants/rto/v1/variant.yaml) | Return-to-origin: delivery failed, package returning to seller |
| [`mid-transaction-changes`](variants/mid-transaction-changes/v1/variant.yaml) | Buyer-initiated updates after confirm: address, instructions, invoice, price adjustment, ready-to-ship |
| [`cross-cutting`](variants/cross-cutting/v1/variant.yaml) | Available on all patterns always: tracking, support, rating, reconciliation, raise escalation |

## How patterns and variants combine

```
Example: Pak Ahmad orders prescription Metformin monthly

Pattern:  subscription          ← recurring monthly flow
Variant:  during-transaction    ← subscription payment method selected at confirm
Variant:  cross-cutting         ← always active (track, support, reconcile)

Path: trade / TRD-04 / TRC-health-beauty / subscription / delivery-time-kyc
```

The `delivery-time-kyc` behaviour (OTP at door for prescription medicines) is derived automatically from `prescriptionRequired=true` on the resource — it is not a separately declared Variant in this sector. It is a field-driven behaviour documented in the pattern.

> **Note:** Food delivery and online food ordering (GoFood, GrabFood, restaurant apps) are **Hospitality** sector — see `flows/hospitality/`. Trade covers packaged food products (FMCG, groceries, packaged beverages) sold through e-commerce.
