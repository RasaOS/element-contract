# RasaOS Element Contract

**Contract Version:** 1.2.0
**Effective:** 2026-05-23
**Canon Version:** 1.2.0 (PENDING REVIEW — see canon CHANGELOG)
**Status:** Authoritative for every Element repo

## What changed in v1.2.0

Reflects the 10 amendments absorbed in canon v1.2.0 (engines bundle
+ Element distribution + RasaConsole repositioning):

- **`provides.engines`** (optional array of strings) — declares which
  engines this Element supports. Absence = engine-agnostic. (SA-008)
- **`provides.default_engine`** (optional string) — Element's preferred
  engine when caller doesn't specify. Must be a member of
  `provides.engines` if both set. (SA-008)
- **Install postures** — Elements may be installed kernel-wide
  (`/rasa/modules/<name>/`) or project-embedded
  (`<cwd>/.rasa/elements/<name>/`). Project-embedded wins on name
  collision. (SA-015 multi-instance)
- **Trust model** — at v1.2 every Element hits the unsigned/TOFU path
  on install (signing infrastructure ships v2 alongside Phase 9 auth).
  Operators see one prompt on first install per `(name, url)`; TOFU
  persists the decision. (SA-016)
- **Skill router** — Element skills register under their natural name
  only. The legacy `<skill>.<engine>` suffix pattern is deprecated;
  shim accepts it through v2.0 with a `deprecation_warnings` signal
  on the result. (SA-010)

Backwards-compatible — existing Elements at `contract_version: "1.1.0"`
or `"1.0.0"` remain valid; they MAY upgrade to `contract_version: "1.2.0"`
at their next commit but are not required to. Elements wishing to
declare engine support, opt into multi-instance install postures, or
participate in the trust model should bump their declaration.

## What changed in v1.1.0

Added `recipe` as the 7th Element kind (canon Spec §6, TASK-122
absorbed). Recipes are now Elements with the same
install/trust/dependency pipeline as any other Element.
Backwards-compatible — existing Elements at `contract_version: "1.0.0"`
remain valid; they MAY upgrade to `contract_version: "1.1.0"` at their
next commit but are not required to.

> What every Element MUST be, do, and not do. The condensed read.
> Canon Spec §5–§6 is the spec; this is the contract authors follow.
> When they disagree, **Spec wins** and this contract gets corrected.

---

## §1 — What is an Element

An **Element** is any artifact in the RasaOS system. Every domain,
orchestrator, module, kernel, frontend, and core is an Element. One
Element = one folder (or one image, for kernel/frontend) = one
GitHub repo under the `RasaOS` org.

Every Element MUST conform to this contract. No exceptions for
"alpha" or "internal" Elements — alpha just means a low version
number, not relaxed rules.

---

## §2 — The seven valid kinds

Canon Spec §6 (v1.1+). The `kind` field in every `rasa.json` is
exactly one of:

| Kind | Tier | Distribution | What it is |
|---|---|---|---|
| `domain` | L1 | Folder | Portable foundation for a vertical (`rasa.domain.code`, `rasa.domain.legal`). |
| `orchestrator` | L1 | Folder | Macro coordination across instances within a domain or across siblings in a workspace. |
| `module` | L1 | Folder | A focused capability that extends a domain or orchestrator. Mountable into one or more parents. Replaces the legacy "skill" kind. |
| `core` | L1 | Folder | The shared L1 foundation every vertical depends on. |
| `kernel` | L2 | Image | The execution substrate (`rasa.kernel`). One per deployment. |
| `frontend` | L4 | Image | Vertical-specific UI surfaces (`rasa.frontend.legalos`, etc.). |
| `recipe` | L4 | Folder | **(v1.1+)** A composition declaration naming a kernel + frontend + Element list + theme. Resolved by the Gateway at install time to produce a Vertical. Same install/trust/dependency pipeline as any other Element. The composition JSON lives in the Recipe Element's `content/`. Examples: `recipe-legalos`, `recipe-healthos`. |

There is no `skill`, no `vertical`, no `kit`, no `plugin` kind. The
seven above are the entire set.

---

## §3 — Naming (three forms, one identity)

Canon Brand Kit §3. Every Element has three name forms:

| Form | Pattern | Example | Used in |
|---|---|---|---|
| **Dotted logical** | `rasa.<kind>.<name>` | `rasa.domain.code` | `rasa.json#name`, all config refs |
| **Repo / folder** | `<kind>-<name>` (no `rasa-` prefix) | `domain-code` | GitHub repo, local folder |
| **Display** | "RasaOS \<Kind\> · \<Name\>" | "RasaOS Domain · Code" | Human-facing text, READMEs, marketing |

The dotted logical name MUST match the regex (v1.1+):

```
^rasa\.(domain|orchestrator|module|core|kernel|frontend|recipe)\..+$
```

…OR be exactly `rasa.core` / `rasa.kernel` (the two singletons).

---

## §4 — Required files at Element root

| File | Required? | Purpose |
|---|---|---|
| `rasa.json` | **Yes** (canon Spec §6) | The Connection Contract — Element's machine-readable identity + install protocol. |
| `VERSION` | **Yes** (workspace convention) | Single-line semver. Mirrors `rasa.json#version`. |
| `README.md` | **Yes** (workspace convention) | Human description. First thing a reader looks at. |
| `CLAUDE.md` | **Yes** (workspace convention) | Per-Element contract for Claude sessions opened in this folder. |
| `CHANGELOG.md` | **Yes** (workspace convention) | Reverse-chronological version history. |
| `.gitignore` | **Yes** | At minimum: `.DS_Store`, `._*`, editor cruft. |
| `content/` | **Conditional** | Required for `domain`, `orchestrator`, `module`, `core`. Optional for `kernel`, `frontend` (which ship as images). |
| `seed/` | Optional | One-time templates that get copied into install targets. |
| `bin/init` | Optional | Install script. Required if the Element installs into consumer projects. |
| `bin/check-manifest` | Optional but recommended | Validates `rasa.json` against the schema. |

---

## §5 — `rasa.json` required fields

Canon Spec §6. Every Element's `rasa.json` MUST include:

| Field | Type | Notes |
|---|---|---|
| `$schema` | string (URL) | `https://rasa-os.github.io/schema/rasa.schema.v1.json` |
| `contract_version` | string | **Which version of this Element Contract** the Element conforms to (e.g. `"1.0.0"`). Distinct from `version` below — see §5a. |
| `name` | string | Dotted logical form. Matches the regex in §3. |
| `version` | string | The Element's **own** project semver (`MAJOR.MINOR.PATCH`). Distinct from `contract_version`. |
| `kind` | enum | One of the six in §2. |

Minimum viable `rasa.json`:

```json
{
  "$schema": "https://rasa-os.github.io/schema/rasa.schema.v1.json",
  "contract_version": "1.0.0",
  "name": "rasa.<kind>.<name>",
  "version": "0.1.0",
  "kind": "<kind>"
}
```

This is enough to load under Phase 1's permissive schema. Phase 7's
strict schema (canon Spec §6) requires more.

### §5a — `contract_version` vs `version` (two distinct versions)

Every Element carries two version numbers, and they answer different
questions:

| Field | Question it answers | Bumped by | Example |
|---|---|---|---|
| `contract_version` | "Which Element-format spec does this Element conform to?" | A change to this contract (canon-level event, infrequent — every Element repo follows) | `1.0.0` |
| `version` | "What's the project semver of THIS Element?" | This Element's own development (frequent — every meaningful change) | `0.41.0` |

Concrete: `rasa.domain.code` at `version: "0.41.0"` conforms to
`contract_version: "1.0.0"`. Six months later `rasa.domain.code` may
be at `version: "0.55.0"` still conforming to `contract_version:
"1.0.0"` (nothing in the contract changed). When the contract bumps
to `1.1.0`, every Element migrates to declare `contract_version:
"1.1.0"` on their next push — a contract bump triggers a
coordinated drift-fix sweep across the workspace.

---

## §6 — `rasa.json` optional fields (commonly used)

| Field | Type | Purpose |
|---|---|---|
| `aliases` | array of strings | Legacy names this Element used to be called. Helps discovery / migration. |
| `description` | string | One paragraph: what this Element is, who uses it, why. No marketing. |
| `source.repo` | string (URL) | `https://github.com/rasa-os/<repo-name>`. Lowercase `rasa-os` per workspace convention. |
| `source.branch` | string | Default `"main"`. |
| `tier` | enum | `Free` / `Pro` / etc. — license tier (canon Auth Service spec). |
| `capabilities` | array of strings | Dotted capability names (e.g. `engineering.task-orchestration`). What this Element provides. |
| `permissions` | array of strings | What the Element needs at runtime: `shell:exec`, `fs:write`, `git:write`, `network:fetch`, `hooks:install`, `identity:bind`. |
| `requires` | object | Other Elements this depends on. Empty `{}` if standalone. |
| `requires.parent_kind` | array of enum | **module kind only**. Restricts mountable parents — subset of `["domain", "orchestrator"]`. Default: both. |
| `rasa` | object | RasaOS-specific metadata (namespace, domain, shape, notes). |
| `scaffold.directories` | array of strings | Empty directories created at install with a `.gitkeep`. |
| `element.files` | array of objects | Files mirrored into install targets. Each entry declares `from`, `to`, `policy`. |
| `seed.files` | array of objects | One-time templates. Same shape as `element.files` but with one-shot policies. |
| `provides.engines` | array of strings | **(v1.2+)** Engine names this Element supports (e.g. `["claude", "codex"]`). Absence means engine-agnostic (any kernel-supported engine accepted). (SA-008) |
| `provides.default_engine` | string | **(v1.2+)** Element's declared default engine when caller doesn't specify. If present, MUST be a member of `provides.engines`. Drives SA-009 resolution chain rung 2. |

---

## §7 — Install policies (canon Spec §6)

When `bin/init` runs against a target directory, each file in
`element.files[]` and `seed.files[]` is applied by its declared
`policy`:

| Policy | When | Behavior |
|---|---|---|
| `directory-mirror` | element.files | Overlay a directory's contents into the target. Used for skills/, agents/, rules/, etc. |
| `file-replace` | element.files | Copy a single file (overwrites). Used for rules files. |
| `opt-in` | element.files | Registered but NOT installed automatically. Must be enabled by hand. |
| `skip-if-exists` | seed.files | Copy only if the target file is absent. Used for project-owned files (CLAUDE.md, etc.). |
| `init-only-with-sha` | seed.files | Copy + substitute `{{ELEMENT_SHA}}`, `{{ELEMENT_BRANCH}}`, `{{TODAY}}` placeholders. Used for `rasa.lock.json.template`. |

Adding a new policy is a canon-level event (Spec §6 amendment).

---

## §8 — Vocabulary lock (forbidden terms)

Per canon Brand Kit §3 and workspace `CLAUDE.md` conventions. These
legacy terms MUST NOT appear in any Element's `rasa.json`, source
files, or docs:

| Forbidden | Replacement |
|---|---|
| `kit` | `Element` |
| `MANIFEST.json` | `rasa.json` |
| `bootstrap/` | `seed/` |
| `foundation.json` | `rasa.lock.json` |
| `manifest contract` | `Connection Contract` |

Adding a new forbidden term is a canon-level event. Existing
pre-canon Elements (e.g. absorbed kits) get a one-time drift-fix to
bring them into compliance; they SHOULD NOT ship new commits in the
legacy shape.

---

## §9 — Validation gate

Before pushing any Element change, the author MUST verify:

1. `rasa.json` is valid JSON (`python3 -c "import json; json.load(open('rasa.json'))"`).
2. `rasa.json#name` matches the regex in §3.
3. `rasa.json#kind` is one of the six in §2.
4. `rasa.json#version` matches `VERSION` file contents.
5. No forbidden vocabulary in any file (grep against the §8 table).
6. `bin/check-manifest` exits 0 (if the Element ships one).

Once the `schema/` sibling lands (Phase 7), a real JSON Schema
validator will enforce §5–§7 strictly. Until then, the rules above
are the contract.

---

## §10 — Versioning discipline

semver. Increments mean:

| Bump | When |
|---|---|
| **Patch** (0.1.0 → 0.1.1) | New skill / new file / doc improvement / additive content. No interface change. |
| **Minor** (0.1.0 → 0.2.0) | New capability category, new permission, new dependency, new install policy used. Existing consumers still work. |
| **Major** (0.x.x → 1.0.0 → 2.0.0) | Breaking interface change. `rasa.json` field removed/renamed, capability removed, install policy semantics changed. Consumers MAY break. |

Every bump:
1. Edit `VERSION` file.
2. Update `rasa.json#version`.
3. Append `CHANGELOG.md` entry under a new dated header.
4. Commit with message including the new version.
5. Push.

Elements at `v0.x.x` are alpha — the interface may change without a
major bump. Once `v1.0.0` is reached, semver discipline is strict.

---

## §11 — When this contract itself changes

The contract is independently versioned. Bump rules above apply to
this document too. A contract bump is a canon-level event:

1. Edit this document's `Contract Version` line at the top.
2. Append `canon/CHANGELOG.md` entry recording the contract bump.
3. Sync the canon copy to the public `RasaOS/element-contract` repo
   (see §11a for the publish flow).
4. Tag the new version on the public repo: `git tag vX.Y.Z`.
5. If the contract change is breaking for existing Elements, file a
   drift-fix task per affected Element to update their
   `contract_version` declaration.

When canon Spec §5–§6 changes (the real source), this contract
follows in the same canon CHANGELOG entry. The contract version and
canon version track independently but always advance together when
the Spec moves.

### §11a — Public mirror at `RasaOS/element-contract`

The authoritative source of this contract is `canon/ELEMENT_CONTRACT.md`
(private, this folder). The **public mirror** is at
`https://github.com/RasaOS/element-contract` (public repo, single
file).

Element authors pull the contract from the public mirror at a
specific tag:

```sh
# Pin to a specific contract version (recommended):
curl -O https://raw.githubusercontent.com/RasaOS/element-contract/v1.0.0/ELEMENT_CONTRACT.md

# Always-latest (only for inspection, never for pinning):
curl -O https://raw.githubusercontent.com/RasaOS/element-contract/main/ELEMENT_CONTRACT.md
```

Each Element repo SHOULD keep a local copy of the contract version
it conforms to (at root, named `ELEMENT_CONTRACT.md`), so future
sessions can read it without a network round-trip. The local copy
MUST match the version declared in `rasa.json#contract_version`.

**Publish flow** (when the contract bumps in canon):

1. Update `canon/ELEMENT_CONTRACT.md` per §11.
2. Copy it to the local clone of `RasaOS/element-contract`.
3. Commit with message `"Contract vX.Y.Z — <one-line-summary>"`.
4. Tag `git tag vX.Y.Z` and push tags.
5. Bump the canon CHANGELOG with the public URL of the new tag.

---

## §12 — Authoring checklist (the practical view)

When creating a new Element from scratch:

1. Pick a `kind` from §2.
2. Pick a `name` (dotted form per §3).
3. Create the folder under `~/rAI/rasa-os/<kind>-<name>/`.
4. Author `rasa.json` with the required fields from §5 (start with
   the minimum viable).
5. Write `VERSION` (start at `0.1.0`).
6. Write `README.md` (what it is, status, layout).
7. Write `CLAUDE.md` (per-Element contract for Claude sessions).
8. Write `CHANGELOG.md` with one entry for `v0.1.0`.
9. Add `.gitignore`.
10. Author `content/` as appropriate to the kind.
11. Validate per §9.
12. `git init -b main`, initial commit, `gh repo create RasaOS/<repo-name> --private`, push.
13. Update workspace `CLAUDE.md` table to add the new sibling.
14. Update workspace `CHANGELOG.md`.

---

## See also

- Canon `01_system_specification.html` §5–§6 — the authoritative Spec.
- Canon `02_brand_kit.html` §3 — the naming spec.
- Workspace `CLAUDE.md` — conventions that extend this contract.
- Existing Elements as reference shapes: `domain-code/` (mature, v0.41.0),
  `domain-legal/` (v0.10.0), `orchestrator-workspace/` (v0.1.0, alpha).
