# ION Network — Governance

## ION Council

The ION Council is the governing body for the ION Network Specification. The Council:

- Ratifies new spec versions (major, minor, and patch)
- Approves new sector activations
- Manages the Policy Terms Registry
- Sets network participation rules and fees

## Specification versioning

ION follows semantic versioning (`MAJOR.MINOR.PATCH-STAGE`):

- **MAJOR** — wire-breaking changes (new required fields, removed fields, endpoint removal)
- **MINOR** — additive changes (new optional fields, new endpoints, new policy categories)
- **PATCH** — clarifications, bug fixes, tooling improvements, no wire changes
- **STAGE** — `draft` (under review), `rc` (release candidate), `ga` (generally available)

Upgrade notice: network participants receive **90 days' notice** before any mandatory upgrade to a new version. Policy registry versioning follows the same model.

## Change process

1. Open an issue in the ION Council issue tracker with label `spec-change`
2. Draft a pull request against `main` with the proposed changes
3. ION Council review (minimum 2 approvals required)
4. Merge and tag; CHANGELOG.md updated with the release notes
5. 90-day notice broadcast to all registered network participants before mandatory adoption

## Extension conventions

All ION schema extensions MUST:

- Carry an `x-beckn-attaches-to` annotation naming the Beckn `*Attributes` slot
- `allOf` with `../../../../core/v2/api/v2.0.0/beckn.yaml#/components/schemas/Attributes`
- Include `@context` and `@type` as required fields (inherited from the Beckn `Attributes` base)
- Follow naming conventions in `docs/ION_Schema_Style_Guide.md`
- Reference `beckn.yaml` via the local relative path, never via external GitHub URL

See `docs/ION_Schema_Style_Guide.md` for the complete pack-authoring guide.

## Sectors

Active sectors are declared in `schema/core/v2/api/v2.0.0/ion.yaml → x-ion-sectors`. Opening a new sector requires Council ratification with a defined working group and at least two committed implementation partners.
