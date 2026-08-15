# Luma Knowledge Format — Specification

- **Version:** `v0.0.2`
- **Status:** Released. Pre-1.0 — the `0.0.z` tier is unstable; breaking changes may still ship until `1.0.0`.

## Abstract

The Luma Knowledge Format (LKF) is a format for representing knowledge, designed to be equally friendly to humans and agents. Its core is small by design — flexible and made to be extended — so it adapts to whatever you need to capture.

Knowledge lives in plain markdown files with YAML frontmatter at the top, which you can create and maintain however you like. LKF is built to share knowledge across teams, systems, or organizations, and is designed to require minimal tooling.

## 1. Conventions

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as in RFC 2119.

Two roles are referenced throughout:
- A **producer** writes LKF — a person, an agent, or an export pipeline.
- A **consumer** reads LKF — a human, an agent, a UI, a search index, or tooling.

## 2. Terminology

- **Knowledge Bundle (or Bundle)**: A self-contained, hierarchical collection of knowledge documents. The unit of distribution.
- **Concept**: A single unit of knowledge within a Bundle, represented as one markdown document. It may describe a tangible asset (a table, an API), an abstract idea (a metric, a business process), or anything in between.
- **Concept ID**: The path of the Concept's file within the Bundle, with the `.md` suffix removed.
- **Slug**: A Concept's filename without its directory path or `.md` extension (e.g. `diffusion-models` for `wiki/concepts/diffusion-models.md`).
- **Concept Type** (or **Type**): The value of a Concept's `type` field — a short string naming the kind of Concept (e.g. `task`, `note`, `lab_result`). An open vocabulary; consumers tolerate unknown types.
- **Type Definition**: A Concept (with `type: type_definition`) that declares a type's contract — its fields, their obligations, and their field types (§10).
- **Field type**: The shape of a field's value (e.g. `text`, `number`, `wikilink`), declared in a Type Definition (§10.2). Distinct from a Concept's `type`.
- **Frontmatter**: A YAML metadata block delimited by `---` at the top of a markdown file.
- **Body**: Everything in the file after the frontmatter.
- **Link**: A markdown link from one Concept to another, expressing a relationship between them.
- **Source**: A material a Concept derives from, external or internal to the Bundle, recorded in the `sources` frontmatter field.

## 3. Concept ID

A Concept's **ID** is its file path within the Bundle, with the `.md` suffix removed. For example, `wiki/concepts/diffusion-models.md` has the ID `wiki/concepts/diffusion-models`.

LKF does not define a separate identifier field; the ID is path-based. Renaming or moving a Concept changes its ID, so producers SHOULD perform renames through tooling that rewrites inbound links (§8). Consumers MUST tolerate links whose target does not resolve (§8).

> Stable opaque identifiers were considered and deferred; see `PRINCIPLES.md` and the project rationale. Because links are name-based, introducing ids later is additive and optional.

## 4. Frontmatter layout and conformance

Core fields defined by this specification appear at the **top level** of the frontmatter, alongside any domain-specific fields (a flat layout — no nesting under a reserved map).

- The field names defined in §5–§7 are **reserved**; producers MUST NOT reuse them for unrelated domain data. The prefix `lkf_` is reserved for future core fields.
- Consumers MUST preserve unrecognized keys when rewriting a file, and MUST NOT reject a file for containing them.
- **Identifier casing (a recommendation).** Field names, `type` names, and `field_type` values prefer snake_case (lowercase words joined by `_`); Concept slugs and IDs prefer kebab-case (`-`), since they are path- and URI-like. Like nearly everything here, this is a strong recommendation, not a hard rule — the only hard requirement is a non-empty `type` (Conformance, below).

**Conformance.** A file is a conformant Concept if it has a parseable YAML frontmatter block containing a non-empty `type`. **This is the only hard requirement.** Consumers **MUST NOT** reject a Concept for: missing recommended or optional fields, an unrecognized `type`, unknown extra keys, or unresolved links. Everything a type declares (§10) is *published intent*, not an enforced rule — validation is a **suggested framework** (§10.5), never a conformance gate, and it never rejects by default.

## 5. Field obligation

Every field — a core field here, or a domain field declared by a Type Definition (§10) — carries an **`obligation`**: how strongly it should be present. Values are stored as full lowercase words:

| Obligation | Meaning |
|---|---|
| `mandatory` | Expected on every Concept of this type. |
| `recommended` | Not mandatory, but include it whenever the information is available; omit only when it genuinely doesn't apply or isn't known. |
| `optional` | May be present; its absence is unremarkable. |
| `deprecated` | Still accepted and read, but on its way out; migrate off it. |

Obligation describes *intent*. Whether and how a tool checks it is a suggested validation framework, not a rule (§10.5), and nothing about obligations changes whether a file is conformant (§4). The sole hard requirement remains a non-empty `type`.

### 5.1 Core fields

| Field | Obligation | Field type | Meaning |
|---|---|---|---|
| `type` | mandatory | text | What kind of Concept this is. **The one hard conformance requirement (§4).** Consumers tolerate unknown types. |
| `title` | recommended | text | Human label; may fall back to the filename. |
| `description` | optional | text | One-sentence summary; used by indexes and search. |
| `tags` | optional | list of text | Categorization; typically nested via `/` (e.g. `ml/generative`). Kept intentionally loose — organizations define their own tag conventions. |
| `lifecycle_status` | optional | enum | `draft \| provisional \| stable \| archived`. §6. |
| `created` | optional | actor_event | Original author + creation time. **Immutable.** §7.1. |
| `modified` | recommended | actor_event | Last editor + last meaningful change. **Advances on edit.** §7.1. |
| `verified` | optional | list of actor_event | Independent confirmation events. §7.2. |
| `sources` | optional | list | Materials the content derives from (bespoke shape). §7.3. |
| `stale_after` | optional | date | The content SHOULD be re-checked after this date. |

> Some obligations above (`title`, `description`, `tags`, `verified`, `sources`) are working defaults pending final ratification.

## 6. Lifecycle: `lifecycle_status`

A Concept's lifecycle stage — nascent to active, with `archived` as the retired terminal. Default (when absent): `provisional`.

| Value | Meaning |
|---|---|
| `draft` | Work in progress; not ready to rely on. |
| `provisional` | Usable but not yet ratified — may still change. (Default.) |
| `stable` | Ratified and trusted. |
| `archived` | Retired from active use; kept for the record. (Supersession — "replaced by X" — is a relationship, not this status.) |

The field is named `lifecycle_status` (not `status`) so it never collides with a tool's own workflow state (e.g. a task's `todo | in-progress | done`), which is often a separate, tool-defined field.

## 7. Provenance and trust

### 7.1 `created` and `modified`

```yaml
created:  { by: human:fsmith, at: 2026-06-14T10:00:00Z }        # original author; fixed forever
modified: { by: agent:gemini-2.5-pro, at: 2026-06-20T22:53:05Z } # last editor; advances on each change
```

- `created.by` is the original-author record. It is more reliable than git for authorship: a git commit's author is whoever committed (often the human running an agent), and history can be squashed or exported.
- `modified.at` is the "last meaningful change" — the freshness signal a consumer uses to tell a recent edit from a stale fact.
- `by` values follow the actor convention (§7.4). Both fields have field type `actor_event` (§10.2).

### 7.2 `verified` and trust tiers

```yaml
verified:
  - { by: human:fsmith, at: 2026-06-25T09:00:00Z }
  - { by: process:cron-nightly, at: 2026-06-26T02:00:00Z }
```

`verified` is a **list** of independent confirmation events. A single verifier MAY be written as a bare `{by, at}` mapping; consumers MUST treat it as a one-element list. `verified` is independent of `modified` — content can change without re-confirmation, and be re-confirmed without changing.

**Trust tiers** are *derived* from `verified`, never stored (to avoid drift):
- no `verified` ⇒ **unverified**
- verified only by non-`human:` actors ⇒ **machine-confirmed**
- verified by any `human:<id>` ⇒ **human-reviewed**

Trust tier is **orthogonal** to `lifecycle_status`: a Concept can be `provisional` yet human-reviewed, or `stable` yet only machine-confirmed.

### 7.3 `sources`

```yaml
sources:
  - id: export-schema                         # local key for body footnotes
    resource: https://path.to/export-schema
    title: Export schema
    author: team:foobar                       # who produced it — authority signal
    last_modified: 2026-05-30                 # when it last changed — recency signal
```

- `resource` — where/what the source is: a URL, file, database export, API response, or a Concept path.
- `author` and `last_modified` are separate fields — **authority** (who) and **recency** (when) are different trust signals.
- `id` is a **local** footnote key within this Concept (unrelated to the Concept ID). In the body, `text.[^export-schema]` attributes a claim to that source (keyed, not positional, so rewrites do not misattribute).

### 7.4 Actor convention

Every `by:` and `author:` value follows one grammar — **`<kind>:<producer>/<version>`**, where `/<version>` is optional:

| Kind | Example | For |
|---|---|---|
| `human:` | `human:fsmith` | a person |
| `agent:` | `agent:gemini-2.5-pro`, `agent:opus-4.8/luma-wiki` | an AI agent or tool (optional `/version` names the tool or wrapper) |
| `process:` | `process:cron-nightly` | an automated process |
| `team:` | `team:foobar` | a team or organization |
| `unknown:` | `unknown:unknown` | the actor was not recorded |

**`unknown` is permitted as either half.** `agent:unknown` records that an agent wrote something without naming which; `human:unknown` records that a person did; `unknown:unknown` records that neither is known.

A tool that cannot tell who invoked it should write the honest value rather than guessing a plausible one. The alternative — omitting `by` — is worse, because it discards the `at` timestamp sharing the same `actor_event`, and because a missing author reads as an oversight where `unknown:unknown` reads as a fact.

The uniform `<kind>:<value>` shape means a consumer parses any actor by splitting on the first `:`.

### 7.5 Date granularity

`at` = a full datetime (ISO 8601 / RFC 3339, e.g. `2026-06-20T22:53:05Z`). `on` and standalone date fields = a date only (`YYYY-MM-DD`).

## 8. Links

All links are by human-readable **slug/path** (LKF has no id-links):

- **Body prose** — slug wikilinks: `[[diffusion-models]]`, `[[diffusion-models|DDPM]]`, `[[note#Heading]]`, `[[note#^block-id]]` (block ids MUST be human-readable, not generated hashes).
- **Frontmatter typed edges** — named keys holding quoted slug/path wikilinks; the key names the relationship. Such a field has field type `wikilink` (§10.2):
  ```yaml
  depends_on: ["[[diffusion-models]]"]
  relates_to: ["[[gut-brain-axis]]"]
  parent:     "[[topic-ml]]"
  ```

Both forms resolve through the consuming tool's index. How a bare slug resolves to a full Concept ID — and how ties between same-slug Concepts are broken — is governed by the link-resolution rules, which are not yet specified (see `ROADMAP.md`). **Unresolved links are legal** — a missing target MAY simply represent not-yet-written knowledge. Renames rewrite inbound links atomically via tooling (§3).

## 9. Body conventions

The body is CommonMark. Producers SHOULD favor structural markdown (headings, lists, tables, code fences) over prose. Two portable extensions are supported:

- **Callouts** — `> [!note]`, restricted to the GitHub ∩ Obsidian intersection for maximum portability: `note`, `tip`, `important`, `warning`, `caution`. Unknown types fall back to `note` (they degrade to a plain blockquote anywhere).
- **Footnote attribution** — `[^id]` keyed to a `sources[].id` (§7.3).

## 10. Type extensions

Any `type` MAY declare a **contract** for its Concepts — which fields they carry and what shape those fields take — so producers and consumers can discover exactly what, say, a `lab_result` expects, and tools MAY validate against it. This is how LKF stays a small core with an open, extensible edge.

Nothing in this section is a conformance requirement. A Type Definition publishes *intent*; §10.5 describes a suggested way to check it; §4 remains the only hard rule.

### 10.1 Type Definitions

A `type` is declared by a **Type Definition** — an ordinary Concept with `type: type_definition`, living in the bundle's reserved `_types/` directory. Because a Type Definition is itself a Concept, it is plain markdown, git-committed, and self-documenting (its body carries docs and examples).

```yaml
---
type: type_definition
defines: lab_result
extends: source
fields:
  test_name: { obligation: mandatory,   field_type: text,   desc: "e.g. LDL cholesterol" }
  value:     { obligation: mandatory,   field_type: number }
  unit:      { obligation: mandatory,   field_type: text }
  patient:   { obligation: mandatory,   field_type: wikilink, desc: "→ the person Concept" }
  panel:     { obligation: recommended, field_type: list of wikilink }
  status:    { obligation: optional,    field_type: enum, values: [pending, final, corrected] }
---

# Lab Result

A single quantitative lab measurement. One file per result.
```

- `defines` — the `type` name this document governs.
- `extends` — a single parent type to inherit from (§10.3).
- `fields` — the field declarations (§10.2).

### 10.2 Field declarations

Each entry under `fields` declares one field with up to four keys:

| Key | Meaning |
|---|---|
| `obligation` | how strongly the field should be present — `mandatory` / `recommended` / `optional` / `deprecated` (§5) |
| `field_type` | the shape of the field's value (below) |
| `desc` | a one-line human/agent description (surfaced by discovery tooling, §10.6) |
| `values` | the allowed values — **required when `field_type` is `enum`**, ignored otherwise |

The key is **`field_type`**, not `type`, so it never collides with a Concept's `type`.

**Field types:**

| `field_type` | Value is |
|---|---|
| `text` | a string |
| `number` | a number |
| `boolean` | `true` / `false` |
| `date` | a date, `YYYY-MM-DD` |
| `datetime` | a full timestamp (ISO 8601 / RFC 3339) |
| `enum` | one of the strings listed in `values` |
| `wikilink` | an internal `[[…]]` link to another Concept in the bundle (of any `type`) |
| `uri` | an external address (URL/URI) |
| `actor` | an actor string (§7.4) |
| `actor_event` | `{ by: actor, at: datetime }` |
| `list of <type>` | a list whose items are any of the above (e.g. `list of wikilink`) |

A **relationship** (a typed edge in the Concept graph) is simply a field whose `field_type` is `wikilink` or `list of wikilink` — the field's *key* names the relationship (`depends_on`, `parent`, `patient`).

### 10.3 Inheritance

- **`extends`** names a single parent type (single inheritance). A type inherits all of its parent's fields and adds its own.
- Every type implicitly extends the built-in **`concept`** root, which supplies the LKF core fields (§5.1). A Type Definition therefore declares only its *domain* fields — never the core fields. This is self-hosting: `type_definition` is itself a type that extends `concept`.
- **Add-only.** A type may only *add* fields. It MUST NOT redefine or remove an inherited field — core or domain. This keeps every inherited field's meaning stable everywhere the type is used.

### 10.4 Resolution and namespacing

- **Resolution.** To find a type's contract, a tool looks in exactly two places: the format's **built-in types** (`concept`, `type_definition`) and the bundle's **`_types/`** directory. There is no remote lookup — a shared type library is used by **vendoring** (copying the `_types/*.md` you want into your own bundle), so a bundle is always self-contained.
- **Reserved built-ins.** The names `concept` and `type_definition` belong to the format; a bundle MUST NOT redefine them.
- **Namespacing (for consideration, not required).** To avoid collisions when types are shared or published, a `type` name SHOULD be namespaced — typically by domain (`health/lab_result`, `finance/invoice`) or organization. At larger scale a team or department dimension MAY be added to disambiguate (e.g. `sales/report`, `engineering/report`). These are examples, not a mandated scheme: namespace however fits your context, or not at all.

### 10.5 Validation — a suggested framework, not a contract

Validation is **entirely optional.** LKF never requires a validator, and no validator's opinion changes whether a file conforms (§4). The rules below are a **recommended, consistent behavior** for tools that choose to offer validation — nothing more. By default, validation **never rejects**; a `--strict` mode is an opt-in that escalates real violations to errors.

| What a validator finds | Default | `--strict` |
|---|---|---|
| Missing `mandatory` field | warning | error |
| A value whose shape ≠ its declared `field_type` | warning | error |
| An `enum` value not listed in `values` | warning | error |
| `extends` naming a type that doesn't exist (broken definition) | warning | error |
| Missing `recommended` field | info | info |
| A `deprecated` field is present | warning | warning |
| An unknown field (not declared by the type) | none¹ | none¹ |
| A `type` with no Type Definition at all | none¹ | none¹ |

¹ Never an error — the permissive-conformance law (§4). A tool MAY surface these as info in a verbose mode.

Two deliberate choices: a `deprecated` field stays a *warning* even under `--strict` (it is still valid, just discouraged), and unknown fields / undefined types are *never* errors (open vocabulary, never reject).

### 10.6 Discovery

Because Type Definitions are just files, humans and agents discover a type's contract the same way: read `_types/<type>.md`, or use tooling such as `luma type list` / `luma type show <type>`.

## 11. Reserved files

- **`index.md`** — derived navigation for a directory; a rebuildable cache, not a source of truth. Optional.
- **`log.md`** — append-only history for a directory, newest first. Creating it is optional, but when it exists writers MUST append rather than rewrite.
- **`_types/`** — Type Definitions (§10).

> **Not yet fully specified:** the exact structure of `index.md` and `log.md`.

## 12. Versioning

- Scheme: **semver `major.minor.patch`**, starting at **0.0.1** (the earliest, most-unstable tier — breaking changes are expected in `0.0.z`). patch = clarifications/errata; minor = backward-compatible additions; major = breaking. Fields are `deprecated` before removal.
- Published versions are **git tags**; the newest tag is the current version.
- A Bundle MAY declare an `lkf_version` on its root `index.md`; a Concept MAY override with its own (file-level wins). This is the *format-grammar* version — not content version (git's job) and not a Type Definition's own `version`.

> Known gaps and deferred features are tracked in [`ROADMAP.md`](ROADMAP.md).
