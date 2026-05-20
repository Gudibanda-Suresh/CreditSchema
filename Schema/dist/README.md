# dist/ — Generated distribution files

**Do not edit these files directly. Regenerate with `python3 tools/build_ion_full.py`.**

| File | Purpose |
|---|---|
| `ion-full.yaml` | Merged OpenAPI 3.1.1 spec for ONIX `Schemav2Validator` |

## Why ion-full.yaml exists

ONIX's `Schemav2Validator` loads a **single** OpenAPI spec file.
- `schema/core/v2/api/v2.0.0/beckn.yaml` — 30 core Beckn paths + 69 Beckn schemas (vendored)
- `schema/core/v2/api/v2.0.0/ion.yaml` — 8 ION paths + 50 ION schemas (ION-authored)

`ion-full.yaml` merges both: **38 paths · 119 schemas** · all `$ref`s resolved inline.

## ONIX configuration

```yaml
plugins:
  - id: schemav2validator
    config:
      api_file_path: /path/to/schema/dist/ion-full.yaml
```

## Regenerate

```bash
python3 tools/build_ion_full.py
```
