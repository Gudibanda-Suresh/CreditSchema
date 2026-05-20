# performance-states/v1

Canonical state machine definitions for all ION Logistics patterns.

## Two-level status model (Beckn 2.0)

ION Logistics maintains **two distinct state levels** per Beckn 2.0 — only the upper level is constrained by Beckn.

### Level 1 — `Contract.status` (Beckn 2.0 native, 4 values)

Beckn 2.0 `Contract.status.code` is constrained to exactly:
- `DRAFT` — contract being negotiated (pre-confirm)
- `ACTIVE` — confirmed and in execution
- `CANCELLED` — cancelled before completion
- `COMPLETE` — fulfilled successfully

Every contract on the network reports one of these four states. Network-level observers (ION Gateway, regulators, analytics) consume this level.

### Level 2 — `Performance.status` (pattern-specific, detailed)

Each performance execution unit carries its own detailed state value. Beckn 2.0 imposes no enum constraint on `Performance.status` — it's free-form per domain. ION Logistics uses state machines with 10-30+ states per pattern, defined in `states.yaml`.

### Mapping

| Performance state category | Contract.status |
|---|---|
| Pre-confirm states (DRAFT, QUOTED, SELECTED) | DRAFT |
| Active fulfilment states (PICKED_UP, IN_TRANSIT, OUT_FOR_DELIVERY, etc.) | ACTIVE |
| Cancellation terminal states | CANCELLED |
| Successful terminal states (DELIVERED, RTO_DELIVERED with RTO acknowledged, COMPLETED) | COMPLETE |

BPP updates `Performance.status` at every state transition. When a performance enters a terminal state, BPP also updates `Contract.status` to the mapped Beckn value.

## State machines defined
| Name | Used by | Terminal states |
|---|---|---|
| `hyperlocal-pre-assigned` | hyperlocal (PRE_ASSIGNED, DEDICATED_FLEET) | DELIVERED, CANCELLED |
| `hyperlocal-fifo` | hyperlocal (FIFO_AT_PICKUP, POOL_ASSIGNED) | DELIVERED, CANCELLED |
| `parcel-p2p` | parcel (P2P topology) | DELIVERED, RTO_DELIVERED |
| `parcel-p2h2p` | parcel (P2H2P topology) | DELIVERED, RTO_DELIVERED |
| `parcel-p2h2h2p` | parcel (P2H2H2P topology) | DELIVERED, RTO_DELIVERED |
| `parcel-reverse-qc` | reverse variant (on parcel) | RETURN_DELIVERED_TO_ORIGIN, RETURN_DISPUTED |
| `freight-capacity` | freight | DELIVERED, FREIGHT_RETURN_INITIATED |
| `roro` | roro | EXITED_PORT, MISSED_SAILING |
| `xb` | cross-border | DELIVERED, IMPORT_CUSTOMS_REJECTED, DESTROY_IN_PLACE |
| `warehouse-inventory` | warehouse | INVENTORY_RELEASED, STORAGE_CONTRACT_EXPIRED |

## How to reference
In pattern.yaml: `stateMachine: schema/extensions/logistics/performance-states/v1/states.yaml#{name}`

## Routing topology selection (parcel)
The topology variant is determined by `offer.routingTopology` at catalog time. The pattern field is `performanceAttributes.routingTopology` set in `on_select` or inherited from the offer.
