# RestaurantPerformance — Overview

**Pack:** `hospitality/performance/v1`  
**Beckn slot:** `Performance.performanceAttributes`  
**CRC:** HSC-restaurant-ordering  

## Purpose

RestaurantPerformance provides kitchen state, rider tracking, live GPS, delivery proof for online food ordering.

## Attachment point

`Performance.performanceAttributes` — see `x-beckn-attaches-to` in `attributes.yaml`.

## Conditional requirements

Fields with `x-ion-condition:` are enforced by ONIX Rego policy.
See `docs/ION_ONIX_Conditional_Policy.md` in the repo root.

## Examples

See `examples/` for complete payloads.
