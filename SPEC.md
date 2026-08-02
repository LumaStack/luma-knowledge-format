# Luma Knowledge Format — Specification

- **Version:** `v0.0.1-draft`
- **Status:** Draft — not yet ratified; breaking changes expected.

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
- **Concept Type** (or **Type**): The value of a Concept's required `type` field — a short string naming the kind of concept (e.g. `task`, `note`, `lab-result`). An open vocabulary; consumers tolerate unknown types. 
- **Frontmatter**: A YAML metadata block delimited by `---` at the top of a markdown file.
- **Body**: Everything in the file after the frontmatter.
- **Link**: A markdown link from one Concept to another, expressing a relationship between them.
- **Source**: A material a Concept derives from, external or internal to the Bundle, recorded in the `sources` frontmatter field.
- **Extension**: A set of rules that specifies how a given `type` extends the base format with its own required, recommended, or optional fields — declared in a Type Definition (§10). 

## 3. Concept ID

A Concept's **ID** is its file path within the Bundle, with the `.md` suffix removed. For example, `wiki/concepts/diffusion-models.md` has the ID `wiki/concepts/diffusion-models`.

LKF does not define a separate identifier field; the ID is path-based. Renaming or moving a Concept changes its ID, so producers SHOULD perform renames through tooling that rewrites inbound links (§8). Consumers MUST tolerate links whose target does not resolve (§8).

> Stable opaque identifiers were considered and deferred; see `PRINCIPLES.md` and the project rationale. Because links are name-based, introducing ids later is additive and optional.

## 4. Frontmatter layout and conformance

Core fields defined by this specification appear at the **top level** of the frontmatter, alongside any domain-specific fields (a flat layout — no nesting under a reserved map).

- The field names defined in §5–§7 are **reserved**; producers MUST NOT reuse them for unrelated domain data. The prefix `lkf_` is reserved for future core fields.
- Consumers MUST preserve unrecognized keys when rewriting a file, and MUST NOT reject a file for containing them.

**Conformance.** A file is a conformant Concept if it has a parseable YAML frontmatter block containing a non-empty `type`. This is the only hard requirement. Consumers **MUST NOT** reject a Concept for: missing recommended or optional fields, an unrecognized `type`, unknown extra keys, or unresolved links. Validation is advisory (§5.1); a validator MAY offer a `--strict` mode that treats missing `required` fields as errors.

## 5. Field requirement levels

Every field — a core field here, or a domain field declared by a Type Definition (§10) — carries exactly one **level**:

| Level | Meaning |
|---|---|
| `required` | MUST be present (in canonical form). A validator reports absence as an error under `--strict`, a warning otherwise. |
| `recommended` | SHOULD be present. Reported as info. |
| `optional` | MAY be present. Not reported. |
| `deprecated` | Still accepted and read, but slated for removal. A validator warns: migrate off it. |

Field-level `deprecated` parallels document-level `lifecycle_status: deprecated` (§6): the same graceful-evolution idea at both scales.

### 5.1 Core fields

| Field | Level | Type | Meaning |
|---|---|---|---|
| `type` | required | string (open vocabulary) | What kind of Concept this is. **The one always-required field.** Consumers tolerate unknown types. |
| `title` | recommended | text | Human label; may fall back to the filename. |
| `description` | optional | text | One-sentence summary; used by indexes and search. |
| `tags` | optional | list of text | Categorization; nested via `/` (e.g. `ml/generative`). |
| `lifecycle_status` | optional | enum | `draft \| provisional \| stable \| deprecated`. §6. |
| `created` | optional | `{by, at}` | Original author + creation time. **Immutable.** §7.1. |
| `modified` | recommended | `{by, at}` | Last editor + last meaningful change. **Advances on edit.** §7.1. |
| `verified` | optional | list of `{by, at}` | Independent confirmation events. §7.2. |
| `sources` | optional | list | Materials the content derives from. §7.3. |
| `stale_after` | optional | date | The content SHOULD be re-checked after this date. |

> Some levels above (`title`, `description`, `tags`, `verified`, `sources`) are working defaults pending final ratification.

## 6. Lifecycle: `lifecycle_status`

Ordered least → most trusted. Default (when absent): `stable`.

| Value | Meaning |
|---|---|
| `draft` | Work in progress; not ready to rely on. |
| `provisional` | Usable and reviewed, but not yet ratified — may still change. |
| `stable` | Ratified and trusted. (Default.) |
| `deprecated` | Superseded or retired; kept for history. |

The field is named `lifecycle_status` (not `status`) so it never collides with a tool's own workflow state (e.g. a task's `todo | in-progress | done`), which is a separate, tool-defined field.

## 7. Provenance and trust

### 7.1 `created` and `modified`

```yaml
created:  { by: human:fsmith, at: 2026-06-14T10:00:00Z }   # original author; fixed forever
modified: { by: gemini-2.5, at: 2026-06-20T22:53:05Z }  # last editor; advances on each change
```

- `created.by` is the original-author record. It is more reliable than git for authorship: a git commit's author is whoever committed (often the human running an agent), and history can be squashed or exported.
- `modified.at` is the "last meaningful change" — the freshness signal a consumer uses to tell a recent edit from a stale fact.
- `by` values follow the actor convention (§7.4).

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

Trust tier is **orthogonal** to `lifecycle_status`: a concept can be `provisional` yet human-reviewed, or `stable` yet only machine-confirmed.

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

Every `by:` and `author:` value uses one convention:

| Form | Example | For |
|---|---|---|
| `human:<id>` | `human:fsmith` | a person |
| `<producer>/<version>` | `gemini-2.5-pro`, `opus-4.8/luma-wiki` | an agent or tool |
| `process:<id>` | `process:cron-nightly` | an automated process |
| `team:<id>` | `team:foobar` | a team or organization |

### 7.5 Date granularity

`at` = a full datetime (ISO 8601 / RFC 3339, e.g. `2026-06-20T22:53:05Z`). `on` and standalone date fields = a date only (`YYYY-MM-DD`).

## 8. Links

All links are by human-readable **slug/path** (LKF v0.0.1 has no id-links):

- **Body prose** — slug wikilinks: `[[diffusion-models]]`, `[[diffusion-models|DDPM]]`, `[[note#Heading]]`, `[[note#^block-id]]` (block ids MUST be human-readable, not generated hashes).
- **Frontmatter typed edges** — named keys holding quoted slug/path wikilinks; the key names the relationship:
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

Any `type` MAY declare its own frontmatter contract, so producers and consumers can discover exactly which fields a `lab-result` (or `task`, `recipe`, …) requires, and validators can check it.

- A type is declared by a **Type Definition** — an ordinary Concept with `type: type-definition`, in the reserved `_types/` directory.
- Illustrative shape:
  ```yaml
  defines: lab-result
  extends: source
  fields:
    test_name: { req: required, type: text, desc: "e.g. LDL cholesterol" }
    value:     { req: required, type: number }
  edges:
    patient:   { req: required, type: link, desc: "→ the person entity" }
  ```

> **Not yet fully specified in this draft:** the property-type vocabulary, `extends`/inheritance and conflict rules, tool-default vs. bundle precedence, and exact validator severities. See open items.

## 11. Reserved files

- **`index.md`** — derived navigation for a directory; a rebuildable cache, not a source of truth. Optional.
- **`log.md`** — append-only history for a directory, newest first. Creating is optional, MUST append when it exists.
- **`_types/`** — Type Definitions (§10).

> **Not yet fully specified in this draft:** the exact structure of `index.md` and `log.md`.

## 12. Versioning

- Scheme: **semver `major.minor.patch`**, starting at **0.0.1** (the earliest, most-unstable tier — breaking changes are expected in `0.0.z`). patch = clarifications/errata; minor = backward-compatible additions; major = breaking. Fields are `deprecated` before removal.
- Published versions are **git tags** (`v0.0.1`); the spec at `HEAD` is the current version.
- A Bundle MAY declare `lkf_version: "0.0.1"` on its root `index.md`; a Concept MAY override with its own `lkf_version` (file-level wins). This is the *format-grammar* version — not content version (git's job) and not a Type Definition's own `version`.

> Known gaps and deferred features are tracked in [`ROADMAP.md`](ROADMAP.md).