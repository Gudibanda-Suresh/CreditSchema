# ION Logistics Commitment Extension (v1) — Overview

**Pack:** `logistics/commitment/v1`  
**Sector:** Logistics  
**Version:** v1  

## Purpose

ION Logistics Commitment Extension (v1) defines the ION extension attributes for the `commitment` slot in the logistics sector. See `attributes.yaml` for the full property catalogue and `vocab.jsonld` for the JSON-LD vocabulary.

## Attachment point

See `x-beckn-attaches-to` in `attributes.yaml` for the Beckn v2 core object slot this pack extends.

## Conditional requirements

Fields marked `x-ion-condition:` in `attributes.yaml` are conditionally required. Enforcement is at the ONIX API boundary via Rego policy — see `docs/ION_ONIX_Conditional_Policy.md` in the repo root.

## Examples

See `examples/` directory for complete payload examples.
