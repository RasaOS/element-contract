# RasaOS Element Contract

Public mirror of the authoritative contract every RasaOS Element repo
must follow. The source of truth lives in the (private) RasaOS canon
at `canon/ELEMENT_CONTRACT.md`; this repo exists so Element authors
can pull the contract by URL without canon access.

## Current version

**Contract v1.0.0** — initial release, 2026-05-22.

See [`ELEMENT_CONTRACT.md`](ELEMENT_CONTRACT.md) for the full text.

## How to pull

**Pinned to a specific version (recommended for production use):**

```sh
curl -O https://raw.githubusercontent.com/RasaOS/element-contract/v1.0.0/ELEMENT_CONTRACT.md
```

**Always-latest (for inspection only — never pin to `main`):**

```sh
curl -O https://raw.githubusercontent.com/RasaOS/element-contract/main/ELEMENT_CONTRACT.md
```

## How to declare conformance in an Element

In your Element's `rasa.json`, set `contract_version` to the version
you conform to:

```json
{
  "$schema": "https://rasa-os.github.io/schema/rasa.schema.v1.json",
  "contract_version": "1.0.0",
  "name": "rasa.<kind>.<name>",
  "version": "0.1.0",
  "kind": "<kind>"
}
```

`contract_version` answers "which Element-format spec does this
Element conform to?" — distinct from `version`, which is the
Element's own project semver. A `domain-code` at `version: "0.41.0"`
may conform to `contract_version: "1.0.0"`; both numbers evolve
independently.

## Versions

| Tag | Released | Summary |
|---|---|---|
| `v1.0.0` | 2026-05-22 | Initial contract. Six kinds, three name forms, required files, rasa.json schema (required + optional), install policies, vocabulary lock, validation gate, versioning discipline, authoring checklist, public mirror flow. |

## Publish flow (for canon maintainers only)

When the contract bumps in canon:

1. Author the change in `~/rAI/rasa-os/canon/ELEMENT_CONTRACT.md`,
   bumping the `Contract Version` line.
2. Append a `canon/CHANGELOG.md` entry.
3. Copy the updated file here: `cp ../canon/ELEMENT_CONTRACT.md
   ./ELEMENT_CONTRACT.md`.
4. Update this README's "Current version" and Versions table.
5. Commit with message `Contract vX.Y.Z — <one-line-summary>`.
6. Tag: `git tag vX.Y.Z` and `git push --tags`.

## License

Same as the surrounding RasaOS canon — see the canon `LICENSE` (when
published).

## What's NOT in this repo

- The full canon document set (private, in RasaOS/canon)
- The JSON Schema for `rasa.json` (separate work, Phase 7)
- Reference Element implementations (see RasaOS/domain-code,
  RasaOS/domain-legal as examples — also private for now)
