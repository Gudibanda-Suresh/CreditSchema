# RestaurantCommitment — Overview

**Pack:** `hospitality/commitment/v1`  
**Beckn slot:** `Commitment.commitmentAttributes`  
**CRC:** HSC-restaurant-ordering  

## Purpose

RestaurantCommitment provides order line with customisations and dietary requests for online food ordering.

## Attachment point

`Commitment.commitmentAttributes` — see `x-beckn-attaches-to` in `attributes.yaml`.

## Conditional requirements

Fields with `x-ion-condition:` are enforced by ONIX Rego policy.
See `docs/ION_ONIX_Conditional_Policy.md` in the repo root.

## Examples

See `examples/` for complete payloads.
