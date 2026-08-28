# Luma Knowledge Format — Specification

- **Version:** `v0.0.16`
- **Status:** Released. Pre-1.0 — the `0.0.z` tier is unstable; breaking changes may still ship until `1.0.0`.

## Abstract

The Luma Knowledge Format (LKF) is a format for representing knowledge, designed to be equally friendly to humans and agents. Its core is small by design — flexible and made to be extended — so it adapts to whatever you need to capture.

Knowledge lives in plain markdown files with YAML frontmatter at the top, which you can create and maintain however you like. LKF is built to share knowledge across teams, tools, or organizations, and is designed to require minimal tooling.

## Conventions

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as in RFC 2119.

Two roles are referenced throughout:
- A **producer** writes LKF — a person, an agent, or an export pipeline.
- A **consumer** reads LKF — a human, an agent, a UI, a search index, or tooling.

### Sections are named, not numbered

**Number what is ordered; name everything else.** A numbered step is numbered
because step three follows step two — the number carries meaning. A numbered
section carries nothing beyond *where it currently sits*.

**Position is not identity.** Citing `§12` cites *whatever is twelfth*. Reorder,
and every citation is silently wrong — pointing at a real section that is not the
one meant. **This specification carried seven live references to a `§5.2` that
had not existed for four versions**, and nothing reported it.

**Names fail loudly; numbers fail quietly.** A stale name resolves to nothing and
somebody notices. A stale number resolves to the wrong thing and nobody does.
That matters more for an agent than for a person, because an agent acts on what
it found.

**So cite by name.** Where the name is already in the sentence, that is the
citation and nothing further is needed. Where a pointer genuinely helps, link the
heading.

**A heading is now an identifier, so choose it for stability.** Short noun
phrases, no trailing punctuation, no clause after a dash — renaming one breaks
every link to it, which is the cost this convention trades numbering churn for.
It is the better trade because a rename is deliberate and rare, where a reorder
is neither.

**Unless numbers demonstrably help.** If numbering makes a consumer — human or
agent — read, navigate or behave more reliably in a given document, number it and
say why. **This rule exists to stop churn, not to enforce a style**, and evidence
beats it.

## Terminology
- **Knowledge Bundle (or Bundle)**: A self-contained, hierarchical collection of files — the unit of distribution. Every file in a Bundle is either a Document or an Asset. **Self-contained means nothing is fetched in order to read it:** its Documents, its Assets and every Type Definition its Documents use are present in the Bundle, so it may be moved, archived or copied whole and still be read offline. **It is a statement about lookup, not about relationships** — a Bundle may *name* another it expects alongside it, which is a claim about what should be adopted, never a lookup performed while reading.
- **Asset**: A file in a Bundle that is not a Document — a script, a template, an image, a binary. It carries no frontmatter and no `type`, and Type Definitions ([Type extensions](#type-extensions)) do not apply to it.
- **Attachment**: An Asset that a Document links to ([Links](#links)). A relationship, not a category: the same Asset may be an Attachment of several Documents, and an Asset that nothing links to is nobody's Attachment.
- **Document**: A single unit of knowledge within a Bundle, represented as one markdown file with YAML frontmatter. Every Document declares a `type`; what it describes — a table, an API, a metric, a task, a lab result — is that type's business, not the format's.
- **Document ID**: The path of the Document's file within the Bundle, with the `.md` suffix removed.
- **Slug**: A Document's filename without its directory path or `.md` extension (e.g. `diffusion-models` for `wiki/concepts/diffusion-models.md`).
- **Document Type** (or **Type**): The value of a Document's `type` field — a short string naming the kind of Document (e.g. `task`, `note`, `lab_result`). An open vocabulary; consumers tolerate unknown types.
- **Type Definition**: A Document (with `type: type_definition`) that declares a type's contract — its fields, how strongly each should be present, and their field types ([Type extensions](#type-extensions)).
- **Field type**: The shape of a field's value (e.g. `text`, `number`, `wikilink`), declared in a Type Definition ([Field declarations](#field-declarations)). Distinct from a Document's `type`.
- **Frontmatter**: A YAML metadata block delimited by `---` at the top of a markdown file.
- **Body**: Everything in the file after the frontmatter.
- **Link**: A markdown link from one Document to another, expressing a relationship between them.
- **Source**: A material a Document derives from, external or internal to the Bundle, recorded in the `sources` frontmatter field.

## Document ID
A Document's **ID** is its file path within the Bundle, with the `.md` suffix removed. For example, `wiki/concepts/diffusion-models.md` has the ID `wiki/concepts/diffusion-models`.

LKF does not define a separate identifier field; the ID is path-based. Renaming or moving a Document changes its ID, so producers SHOULD perform renames through tooling that rewrites inbound links ([Links](#links)). Consumers MUST tolerate links whose target does not resolve ([Links](#links)).

> Stable opaque identifiers were considered and deferred; see [`PRINCIPLES.md`](docs/PRINCIPLES.md) and the project rationale. Because links are name-based, introducing ids later is additive and optional.

## Frontmatter layout and conformance
Core fields defined by this specification appear at the **top level** of the frontmatter, alongside any domain-specific fields (a flat layout — no nesting under a reserved map).

- The field names defined in [Field presence](#field-presence)–[Provenance and trust](#provenance-and-trust) are **reserved**; producers MUST NOT reuse them for unrelated domain data. The prefix `lkf_` is reserved for future core fields.
- Consumers MUST preserve unrecognized keys when rewriting a file, and MUST NOT reject a file for containing them.
- **Identifier casing (a recommendation).** Field names, `type` names, and `field_type` values prefer snake_case (lowercase words joined by `_`); Document slugs and IDs prefer kebab-case (`-`), since they are path- and URI-like. Like nearly everything here, this is a strong recommendation, not a hard rule — the only hard requirement is a non-empty `type` (Conformance, below).

**Conformance.** A file is a conformant Document if it has a parseable YAML frontmatter block containing a non-empty `type`. **This is the only hard requirement.** Consumers **MUST NOT** reject a Document for: missing recommended or optional fields, an unrecognized `type`, unknown extra keys, or unresolved links. Everything a type declares ([Type extensions](#type-extensions)) is *published intent*, not an enforced rule — validation is a **suggested framework** ([Validation — a suggested framework, not a contract](#validation--a-suggested-framework-not-a-contract)), never a conformance gate, and it never rejects by default.

## Field presence
Every field — a core field here, or a domain field declared by a Type Definition ([Type extensions](#type-extensions)) — carries a **`field_presence`**: how strongly it should be present. Values are stored as full lowercase words:

| Presence | Meaning |
|---|---|
| `required` | Expected on every Document of this type. |
| `recommended` | Not required, but include it whenever the information is available; omit only when it genuinely doesn't apply or isn't known. |
| `optional` | May be present; its absence is unremarkable. |
| `deprecated` | Still accepted and read, but on its way out; migrate off it. |

**`required / recommended / optional` is RFC 2119's adjectival set**, deliberately. Every specification uses those three words for exactly this ladder, so a reader recognises all of them at once rather than two and a synonym. `deprecated` sits beside the ladder rather than on it — it states a field's future rather than its strength, which is why [Inheritance](#inheritance) cannot reach it by strengthening.

Presence describes *intent*. Whether and how a tool checks it is a suggested validation framework, not a rule ([Validation — a suggested framework, not a contract](#validation--a-suggested-framework-not-a-contract)), and nothing about presence changes whether a file is conformant ([Frontmatter layout and conformance](#frontmatter-layout-and-conformance)). The sole hard requirement remains a non-empty `type`.

**Nothing here says when a Document should be placed in front of a reader, and that is deliberate.** A field named `preload` did, through `v0.0.12`, and it was the one place this specification described how a Document should be *consumed* rather than what it is. **Consumption belongs to whatever distributes and loads Bundles**, not to the format that defines them. `matches` ([`matches`, declared by `policy` and `workflow`](#matches-declared-by-policy-and-workflow)) is not its replacement in that sense: it says what makes a Document **surface**, which is a property of the content, and any decision about loading is one a consumer *derives* from it. **It is not a core field** — only `policy` and `workflow` declare it, for the reason given there. **The name `preload` is released** and is no longer reserved.

### Core fields
| Field | Presence | Field type | Meaning |
|---|---|---|---|
| `type` | required | text | What kind of Document this is. **The one hard conformance requirement ([Frontmatter layout and conformance](#frontmatter-layout-and-conformance)).** Consumers tolerate unknown types. |
| `title` | recommended | text | Human label; may fall back to the filename. |
| `description` | optional | text | One-sentence summary; used by indexes and search. |
| `tags` | optional | list of text | Categorization; typically nested via `/` (e.g. `ml/generative`). Kept intentionally loose — organizations define their own tag conventions. |
| [`lifecycle_status`](#lifecycle) | optional | enum | `draft \| provisional \| stable \| archived \| unknown`. Default `unknown`. |
| [`survival`](#survival) | optional | enum | `experimental \| intended \| promised`. Default `intended`, and often left unwritten. |
| [`created`](#created-and-modified) | optional | actor_event | Original author + creation time. **Immutable.** |
| [`modified`](#created-and-modified) | recommended | actor_event | Last editor + last meaningful change. **Advances on edit.** |
| [`verified`](#verified-and-trust-tiers) | optional | list of actor_event | Independent confirmation events. |
| [`sources`](#sources) | optional | list | Materials the content derives from (bespoke shape). |
| `stale_after` | optional | date | The content SHOULD be re-checked after this date. |

> Some presence values above (`title`, `description`, `tags`, `verified`, `sources`) are working defaults pending final ratification.

## Lifecycle
A Document's lifecycle stage — nascent to active, with `archived` as the retired terminal, plus the named absence of a stage. Default (when absent): `unknown`.

| Value | Meaning |
|---|---|
| `draft` | Work in progress; not ready to rely on. |
| `provisional` | Usable but not yet ratified — may still change. |
| `stable` | Ratified and trusted. |
| `archived` | Retired from active use; kept for the record. (Supersession — "replaced by X" — is a relationship, not this status.) |
| `unknown` | **Not a stage.** The value was not filled in, so at read time nobody knows it. (Default.) |

**`unknown` means not filled in, not unknowable.** Whether the fact is lost or was simply never stated is not this field's business, and collapsing both into one value is what lets a single word serve wherever it is needed — the same sense [Actor convention](#actor-convention) gives it for actors.

**It is the default because both real defaults would be wrong guesses.** Defaulting to `provisional` makes a `draft` thing read as more settled than it is; defaulting the other way makes a `stable` thing read as less. Neither direction is safe, which is the `consumers` case described in [`BUNDLE.md`](#bundlemd) rather than the `lifecycle_status` one — and where no default is safe, the honest answer is to say nobody has declared.

**Absent and explicitly `unknown` mean the same thing**, so nothing is ambiguous. Writing it is worth doing anyway: silence cannot distinguish *considered and undecided* from *never thought about*.

*It is not spelled `none`. `none`, `null` and `nil` are absence words in one language or another, so a value that looks like a null gets conflated with one — and `lifecycle_status: none`, an empty value that YAML reads as null, and the field being absent are three states that look alike to a reader and differ to a parser.*

The field is named `lifecycle_status` (not `status`) so it never collides with a tool's own workflow state (e.g. a task's `todo | in-progress | done`), which is often a separate, tool-defined field.

## Provenance and trust
### `created` and `modified`
```yaml
created:  { by: human:fsmith, at: 2026-06-14T10:00:00Z }        # original author; fixed forever
modified: { by: agent:gemini-2.5-pro, at: 2026-06-20T22:53:05Z } # last editor; advances on each change
```

- `created.by` is the original-author record. It is more reliable than git for authorship: a git commit's author is whoever committed (often the human running an agent), and history can be squashed or exported.
- `modified.at` is the "last meaningful change" — the freshness signal a consumer uses to tell a recent edit from a stale fact.
- `by` values follow the actor convention ([Actor convention](#actor-convention)). Both fields have field type `actor_event` ([Field declarations](#field-declarations)).

**Why one nested field rather than `created_by` and `created_at`.** The flat pair reads better in a diff and greps in one line, which is a real cost paid here deliberately. Three things buy it back:

- **`verified` is a *list* of these** ([`verified` and trust tiers](#verified-and-trust-tiers)), and a flat form would be parallel arrays correlated by index. That is the positional-correlation failure this specification avoids everywhere else — [`sources`](#sources) keys source footnotes precisely so *"rewrites do not misattribute."* The composite has to exist for `verified` regardless; using it for `created` and `modified` is consistency rather than extra machinery.
- **The pair is atomic, and [Actor convention](#actor-convention) depends on that.** Its argument for writing `unknown:unknown` rather than omitting `by` is that omission *"discards the `at` timestamp sharing the same `actor_event`."* That reasoning only holds because they are one object. Flat, a `created_at` with no `created_by` is a natural half-record and nothing marks it as incomplete.
- **One declaration instead of an invisible invariant.** A Type Definition says `field_type: actor_event` once; the flat form is two fields plus a rule that they travel together, which no validator can see.

### `verified` and trust tiers
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

Trust tier is **orthogonal** to `lifecycle_status`: a Document can be `provisional` yet human-reviewed, or `stable` yet only machine-confirmed.

### `sources`
```yaml
sources:
  - id: export-schema                         # local key for body footnotes
    resource: https://path.to/export-schema
    title: Export schema
    author: team:foobar                       # who produced it — authority signal
    last_modified: 2026-05-30                 # when it last changed — recency signal
```

- `resource` — where/what the source is: a URL, file, database export, API response, or a Document path.
- `author` and `last_modified` are separate fields — **authority** (who) and **recency** (when) are different trust signals.
- `id` is a **local** footnote key within this Document (unrelated to the Document ID). In the body, `text.[^export-schema]` attributes a claim to that source (keyed, not positional, so rewrites do not misattribute).

### Actor convention
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

**Supported, and an anti-pattern.** A tool that *can* identify its actor should. `unknown:unknown` exists for genuine ignorance — a command invoked through one interface by both people and agents, with nothing to tell them apart — not as a default for tools that never asked. A body of Documents where most authors are `unknown:unknown` has thrown away provenance it could have kept, and no reader can tell which of those were unavoidable and which were laziness.

Prefer the most specific value available: `agent:opus-5` over `agent:unknown` over `unknown:unknown`. Where a tool has a way for the caller to say who is acting, the honest default is `unknown:unknown` and the expected practice is to pass the real one.

The uniform `<kind>:<value>` shape means a consumer parses any actor by splitting on the first `:`.

### Date granularity
`at` = a full datetime (ISO 8601 / RFC 3339, e.g. `2026-06-20T22:53:05Z`). `on` and standalone date fields = a date only (`YYYY-MM-DD`).

## Links
All links are by human-readable **slug/path** (LKF has no id-links):

- **Body prose** — slug wikilinks: `[[diffusion-models]]`, `[[diffusion-models|DDPM]]`, `[[note#Heading]]`, `[[note#^block-id]]` (block ids MUST be human-readable, not generated hashes).
- **Frontmatter typed edges** — named keys holding quoted slug/path wikilinks; the key names the relationship. Such a field has field type `wikilink` ([Field declarations](#field-declarations)):
  ```yaml
  depends_on: ["[[diffusion-models]]"]
  relates_to: ["[[gut-brain-axis]]"]
  parent:     "[[topic-ml]]"
  ```

  > **The quotes are load-bearing, and omitting them fails silently.** `[[…]]` is YAML flow-sequence syntax, so an unquoted wikilink parses as a **nested array** rather than a string, with no error from any YAML parser:
  >
  > ```yaml
  > parent: [[topic-ml]]      # → [["topic-ml"]]   a list containing a list
  > parent: "[[topic-ml]]"    # → "[[topic-ml]]"   a string, as intended
  > ```
  >
  > A validator ([Validation — a suggested framework, not a contract](#validation--a-suggested-framework-not-a-contract)) catches this as a value whose shape does not match its declared `field_type`, but nothing else will — the document stays conformant, and a consumer simply never resolves the link. Producers writing frontmatter wikilinks MUST quote them.

**Assets use ordinary markdown links.** `[[…]]` links a Document; `[…](…)` links anything else — an Asset, or an external address:

```markdown
Run [the setup script](scripts/setup.sh) before the first import.
```

An Asset link is a path relative to the linking Document, and MUST point inside the Bundle — a link reaching outside breaks self-containment, which is the property that lets a Bundle be copied whole. Whether the target *exists* is a separate question: as with Document links, an unresolved Asset link is legal ([Frontmatter layout and conformance](#frontmatter-layout-and-conformance)), and a consumer MUST NOT reject a Bundle for one.

The two forms are distinguishable on sight and neither needs the other's rules — an Asset has no slug and no ID, only a path.

Both forms resolve through the consuming tool's index. How a bare slug resolves to a full Document ID — and how ties between same-slug Documents are broken — is governed by the link-resolution rules, which are not yet specified (see [`ROADMAP.md`](docs/ROADMAP.md)). **Unresolved links are legal** — a missing target MAY simply represent not-yet-written knowledge. Renames rewrite inbound links atomically via tooling ([Document ID](#document-id)).

## Body conventions
The body is CommonMark. Producers SHOULD favor structural markdown (headings, lists, tables, code fences) over prose. Two portable extensions are supported:

- **Callouts** — `> [!note]`, restricted to the GitHub ∩ Obsidian intersection for maximum portability: `note`, `tip`, `important`, `warning`, `caution`. Unknown types fall back to `note` (they degrade to a plain blockquote anywhere).
- **Footnote attribution** — `[^id]` keyed to a `sources[].id` ([`sources`](#sources)).

## Type extensions
Any `type` MAY declare a **contract** for its Documents — which fields they carry and what shape those fields take — so producers and consumers can discover exactly what, say, a `lab_result` expects, and tools MAY validate against it. This is how LKF stays a small core with an open, extensible edge.

Nothing in this section is a conformance requirement. A Type Definition publishes *intent*; [Validation — a suggested framework, not a contract](#validation--a-suggested-framework-not-a-contract) describes a suggested way to check it; [Frontmatter layout and conformance](#frontmatter-layout-and-conformance) remains the only hard rule.

### Type Definitions
A `type` is declared by a **Type Definition** — an ordinary Document with `type: type_definition`, living in the bundle's reserved `_types/` directory. Because a Type Definition is itself a Document, it is plain markdown, git-committed, and self-documenting (its body carries docs and examples).

```yaml
---
type: type_definition
defines: lab_result
extends: source
fields:
  test_name: { field_presence: required,   field_type: text,   desc: "e.g. LDL cholesterol" }
  value:     { field_presence: required,   field_type: number }
  unit:      { field_presence: required,   field_type: text }
  patient:   { field_presence: required,   field_type: wikilink, desc: "→ the person Document" }
  panel:     { field_presence: recommended, field_type: list of wikilink }
  status:    { field_presence: optional,    field_type: enum, values: [pending, final, corrected] }
---

# Lab Result

A single quantitative lab measurement. One file per result.
```

- `defines` — the `type` name this document governs.
- `extends` — a single parent type to inherit from ([Inheritance](#inheritance)).
- `fields` — the field declarations ([Field declarations](#field-declarations)).
- `version` — this Type Definition's own version, `semver`. Optional (below).
- `vendored_from` — where this copy came from, when it is a copy (below).

#### `version`

A Type Definition MAY carry its own `version`, independent of the Bundle it ships in:

```yaml
version: "1.2.0"
```

**It exists because a Bundle's version answers the wrong question for a copied type.** Vendor one type out of a Bundle holding six, and a bump caused by any of the other five reports your copy as out of date when it is byte-identical. A type that versions itself is comparable on its own terms.

**What a bump *means* is deliberately not defined yet.** Treat it as a label rather than a promise. Semantic versioning is the obvious starting point and the tiers have not been worked through for types, so a consumer SHOULD compare versions for equality and difference and SHOULD NOT infer compatibility from the tier that changed.

**Optional, and absence is ordinary.** A type that only ever ships inside one Bundle has nothing to gain from a second version number.

#### `vendored_from`

A Type Definition copied from elsewhere SHOULD record where it came from:

```yaml
vendored_from:
  resource: https://example.org/shared-types
  version: "0.1.0"
  at: 2026-08-22
```

**It is provenance, never a lookup.** Nothing fetches it, and a consumer that cannot reach `resource` reads the Document exactly as before — the contract is the local file and always was. Without this the copy is anonymous, and [Resolution and namespacing](#resolution-and-namespacing)'s vendoring model has no way to tell a current copy from a stale or edited one.

**`version` records the Type Definition's own version where it declares one, and the containing Bundle's otherwise.** Say which in the copy if it could be read either way.

**It answers two questions, and the second is easy to miss.** *Is my copy still current?* — compare against `resource` at `version`, on demand. And *do two copies in one place agree?* — two Bundles that vendored the same type at different versions hold two contracts under one name, which [Resolution and namespacing](#resolution-and-namespacing) permits and which nothing else would surface.

### Field declarations
Each entry under `fields` declares one field with up to four keys:

| Key | Meaning |
|---|---|
| `field_presence` | how strongly the field should be present — `required` / `recommended` / `optional` / `deprecated` ([Field presence](#field-presence)) |
| `field_type` | the shape of the field's value (below) |
| `desc` | a one-line human/agent description (surfaced by discovery tooling, [Discovery](#discovery)) |
| `values` | the allowed values — **required when `field_type` is `enum`**, ignored otherwise |

The key is **`field_type`**, not `type`, so it never collides with a Document's `type`.

**Field types:**

| `field_type` | Value is |
|---|---|
| `text` | a string |
| `number` | a number |
| `boolean` | `true` / `false` |
| `date` | a date, `YYYY-MM-DD` |
| `datetime` | a full timestamp (ISO 8601 / RFC 3339) |
| `semver` | a semantic version — `MAJOR.MINOR.PATCH`, optionally with pre-release and build metadata (`1.0.0-alpha.1+build.5`), exactly as [semver.org](https://semver.org) defines it. No `v` prefix: `v1.2.3` is a tag convention, not a version. |
| `enum` | one of the strings listed in `values` |
| [`wikilink`](#links) | an internal `[[…]]` link to another Document in the bundle (of any `type`). **Quoted in frontmatter** — see the warning in |
| `uri` | an external address (URL/URI) |
| `actor` | an actor string ([Actor convention](#actor-convention)) |
| `actor_event` | `{ by: actor, at: datetime }` |
| `list of <type>` | a list whose items are any of the above (e.g. `list of wikilink`) |

A **relationship** (a typed edge in the Document graph) is simply a field whose `field_type` is `wikilink` or `list of wikilink` — the field's *key* names the relationship (`depends_on`, `parent`, `patient`).

### Inheritance
- **`extends`** names a single parent type (single inheritance). A type inherits all of its parent's fields and adds its own.
- Every type implicitly extends the built-in **`document`** root, which supplies the LKF core fields ([Core fields](#core-fields)). A Type Definition therefore declares only its *domain* fields — never the core fields, except to strengthen a presence value as below. This is self-hosting: `type_definition` is itself a type that extends `document`.
- **Add-only.** A type may only *add* fields. It MUST NOT remove an inherited field, nor redefine its `field_type`, its `values`, or its meaning — core or domain. This keeps every inherited field's meaning stable everywhere the type is used.
- **Presence may be strengthened, never weakened.** A type MAY raise an inherited field's presence — `optional` → `recommended` → `required` — by redeclaring the field with a higher `field_presence` and nothing else changed. It MUST NOT lower one. Where a field is declared at several points in a chain, the **strongest presence wins**; an attempted weakening SHOULD be reported rather than honoured.

  `deprecated` is not on that ladder and is not reachable this way. It states something about a field's future rather than its strength, so a type inheriting a `deprecated` field may not mandate it — a field both deprecated and required is a contradiction rather than a precedence question.

  **This is consistent with add-only because presence is not meaning.** The field means exactly what its declaring type said; a subtype only states how strongly *it* expects the field. Nothing becomes non-conformant either — presence describes intent ([Field presence](#field-presence)) and the sole hard requirement remains a non-empty `type` ([Frontmatter layout and conformance](#frontmatter-layout-and-conformance)) — so a consumer that knows only the parent and one that knows the subtype may reach different completeness verdicts, and each is right at its own level.

  **Without this, a type whose semantics rest on inherited fields cannot state them.** Where a type's growth stage *is* `lifecycle_status` and its age *is* `created` — both `optional` on the root — the type has no way to say that a Document missing either is incomplete, and its own contract calls unremarkable exactly the omissions that break it.

### Resolution and namespacing
- **The Bundle is the resolution scope, and that has a consequence worth stating.** Because a contract is found in *this* Bundle's `_types/`, two Bundles may hold different versions of the same type without contradiction — each one's Documents are checked against the copy that travelled with them. This is the scoping mechanism prose does not have, and it is why vendoring a type is safe where duplicating a policy would not be.

  **A Document outside every Bundle has no such scope.** Nothing prevents a Document living beside Bundles rather than inside one — describing a repository, or a place Bundles are published from. Such a Document declares a `type` like any other, and the format offers no rule for where its contract is found: there is no Bundle to look in. **Whoever puts a Document there owes it an answer**, and where two Bundles disagree about that type, nothing decides between them.

- **Resolution.** To find a type's contract, a tool looks in exactly two places: the format's **built-in types** (`document`, `workflow`, `policy`, `bundle`, `type_definition`) and the bundle's **`_types/`** directory. The built-ins ship as real Type Definitions in this repository's `bundle/` directory — itself a Bundle, so that the unit of distribution is exactly the types and not the project around them — so they are both a normative rendering and a worked example; a tool MAY supply them itself rather than requiring every bundle to vendor them. There is no remote lookup — a shared type library is used by **vendoring** (copying the `_types/*.md` you want into your own bundle), so a bundle is always self-contained.
- **Built-in names.** The names `document`, `workflow`, `policy`, `bundle` and `type_definition` belong to the format; a bundle SHOULD NOT redefine them. Doing so is legal — the permissive-conformance law ([Frontmatter layout and conformance](#frontmatter-layout-and-conformance)) means no consumer rejects a bundle for it — but unwise: a redefinition travels inside the bundle while every tool and every other bundle still assumes the format's meaning.
- **Two base types, because the third thing a consumer can do is the root itself.** `workflow` and `policy` declare no fields between them. They are not labels for subjects — they name **what a consumer does with the content**:

  | type | what a consumer does with it |
  |---|---|
  | `workflow` | **runs it.** A procedure, projected into whatever form the consumer executes |
  | `policy` | **is bound by it.** A rule that constrains the consumer's own behaviour, rather than informing it |

  **The third thing — reads it — needs no type, because it is what `document` already is.** Naming that would be naming the default, and **a type that names the default dispatches on nothing**: every consumer already treats anything without a more specific type exactly that way.

  That is what closes the set rather than leaving it open to any word a Bundle finds useful. **A further base type would have to name a further thing to *do*** — not a further subject matter.

  **None of this says when a Document is loaded.** The format does not say, deliberately — see [Frontmatter layout and conformance](#frontmatter-layout-and-conformance) on unknown keys, and the note at the end of [Field presence](#field-presence). A `policy` binds whether or not it is present, and a `policy` that reaches a reader only when its subject arises is an ordinary thing rather than a contradiction.

  **A rule nobody loads governs nothing, and that is a reachability problem rather than a definitional one.** The answer is that something always present says the rule *exists* — an index costs a line where the rule costs a page — not that every rule is forced into context.

  The distinction is worth having because it is the one a consumer must act on. Two Documents can be identical prose with identical fields, and one belongs in permanent context while the other belongs behind an invocation. Nothing but the `type` can say which.

- **A type earns its name when a consumer dispatches on it.** LKF does not fix what types exist (`PRINCIPLES.md`). The bar for declaring one — in this list or in a Bundle's own `_types/` — is concrete: **name the consumer, and name what it does differently.** If you cannot name both, the `type` is a label, and a label costs a name every other Bundle must then avoid.

  Three forms that difference takes:

  | | the consumer | what it does differently |
  |---|---|---|
  | **checked** | a validator | has a contract to check the Document against — `decision` requires a `decided` date, and nothing can enforce that without the type |
  | **transformed** | whatever converts Documents into another form | `workflow` is projected into a harness; no other Document in a Bundle is |
  | **consulted** | the format's own machinery | reads it to decide how to handle *other* Documents — `type_definition` and `bundle` are both of this kind |

  **Enumeration is not enough.** *"Show me every Document of this kind"* works for any type at all, so admitting it as a reason would grow the list without limit.

  A difference that is **intended but not yet built** may still justify a type, provided the consumer and the difference are named plainly. One that never acquires them has been *falsified* rather than merely unused, and should be deprecated.

- **Being built in is a further bar, not the same one.** A type may earn its name and still belong to a single Bundle. It joins this list only if it is also **unavoidable**, which means one of:

  - **Structural** — the format's own machinery depends on it. Without `document`, `bundle` or `type_definition` there is no root, no unit of distribution, and no way to resolve a contract at all.
  - **Ubiquitous** — nearly every Bundle defines it independently anyway. **This is a measurement, not a prediction:** count the copies before making the claim, and say how many.

    **Then check what you counted.** A sample drawn from Bundles that all serve one domain will find that domain's vocabulary everywhere — ubiquity across seven governance Bundles is not ubiquity. A count is corroboration for a reason that stands on its own, never the reason itself.

  The second is the weaker of the two and cuts both ways. Declining to define a name everyone reaches for does not prevent the name — it produces many private definitions that drift apart. But a list that grows on popularity alone stops being a small core, which is the property the whole format rests on.

  **A built-in is the format's only mandatory surface.** Everything else here is permissive by law ([Frontmatter layout and conformance](#frontmatter-layout-and-conformance)) — unknown types tolerated, missing fields tolerated, unresolved links tolerated, and no consumer may reject a Document for any of it. This list is the one place the format *requires* something of every implementation.

  **So the question is never whether a type is important. It is whether a consumer that ignored it would fail to read a conformant Document, or engage with one in a way this specification says is wrong.** Both count, and they fail differently: without `bundle` or `type_definition` there is no Document ID and no way to obtain a contract, so nothing can be read at all; without `workflow` or `policy` a Document parses perfectly and is then read as information when it is a rule that binds, or a procedure to run — a distinction this specification draws and the consumer got wrong.

  ***My tooling would break* is the wrong kind of broken.** It is true of every domain type ever written. A consumer ignoring one still reads the Document correctly — as a plain `document`, which is what it is — and merely does not participate in something built on top of the format. **That is the format working, not failing.**

  **The cost of a built-in is a word taken from everyone, permanently.** An unprefixed name belongs to the format for good: every Bundle in every domain must then avoid it, and releasing one later collides with whoever defined it privately in the meantime. So *important to us* is not an argument for this list — **importance is what a namespace is for**, and a namespaced type costs nobody anything.

  **A cheap further check: does it change at the format's rate?** A built-in's contract is versioned with the format. A type that gains fields as somebody's tooling matures drags the format's version along with it, and a specification whose releases track one adopter's roadmap has stopped being a specification.

  **Removing a built-in is cheaper than adding one.** Removal costs a deprecation cycle and a frontmatter migration. A late addition costs the same migration *plus* a collision with every Bundle that had already defined the name for itself.

  **That asymmetry is a tiebreaker, not an entry route.** It applies only to a candidate that has already cleared the checks above and is still genuinely balanced — there, admitting now and deprecating later is the cheaper error. It is not a reason to admit something that failed them, because a namespace costs nothing and is available immediately.

- **Namespacing.** **Unprefixed means the format defines it; a prefix means somebody else does.** That is the whole convention, and it lets a reader tell a type's origin from its name without a lookup — `workflow` is LKF's, `acme/deploy_check` is an organization's.

  A `type` published beyond the Bundle that wrote it SHOULD therefore be namespaced — typically by domain (`health/lab_result`, `finance/invoice`) or organization. At larger scale a team or department dimension MAY be added to disambiguate (e.g. `sales/report`, `engineering/report`). These are examples, not a mandated scheme: namespace however fits your context, or not at all.

### Validation — a suggested framework, not a contract
Validation is **entirely optional.** LKF never requires a validator, and no validator's opinion changes whether a file conforms ([Frontmatter layout and conformance](#frontmatter-layout-and-conformance)). The rules below are a **recommended, consistent behavior** for tools that choose to offer validation — nothing more. By default, validation **never rejects**; a `--strict` mode is an opt-in that escalates real violations to errors.

| What a validator finds | Default | `--strict` |
|---|---|---|
| Missing `required` field | warning | error |
| A value whose shape ≠ its declared `field_type` | warning | error |
| An `enum` value not listed in `values` | warning | error |
| `extends` naming a type that doesn't exist (broken definition) | warning | error |
| Missing `recommended` field | info | info |
| A `deprecated` field is present | warning | warning |
| An unknown field (not declared by the type) | none¹ | none¹ |
| A `type` with no Type Definition at all | none¹ | none¹ |

¹ Never an error — the permissive-conformance law ([Frontmatter layout and conformance](#frontmatter-layout-and-conformance)). A tool MAY surface these as info in a verbose mode.

Two deliberate choices: a `deprecated` field stays a *warning* even under `--strict` (it is still valid, just discouraged), and unknown fields / undefined types are *never* errors (open vocabulary, never reject).

### Discovery
Because Type Definitions are just files, humans and agents discover a type's contract the same way: read `_types/<type>.md`. Tooling may wrap that in a lookup command and needs no index to do so — the file *is* the contract, so reading it directly is always available and never stale.

### `matches`, declared by `policy` and `workflow`
**Not a core field.** Two of the built-in types declare it, and no other Document carries it.

**The two kinds that carry a trigger are the two that act on you** — a rule that binds, and a procedure you run. Background does not act; it is reached *through* the things that do. Rationale has no moment either: it is wanted when somebody wonders *why*, and wondering is not a trigger. A concept whose subject genuinely arises somewhere is already reachable from the policy or workflow that arises there, and advertising it separately makes it compete with them for attention.

**What makes this Document surface.** Three forms, and each reads as a sentence:

```yaml
matches: always                 # nothing gates it
```
```yaml
matches:                        # these situations do — any one of them
  - command: git commit
  - path: "src/**/*.css"
  - event: before-release
```
```yaml
matches: nothing                # nothing surfaces it; it is fetched deliberately
```

| Kind | Arises when |
|---|---|
| `path` | a file matching the glob is read or written |
| `tool` | a named tool or capability is invoked |
| `command` | a command of the given shape runs |
| `event` | a lifecycle point is reached — `session-start`, `session-end`, `before-commit`, `before-push`, `before-merge`, `before-release` |
| `topic` | the work is *about* something, recognised by meaning rather than by pattern |

**`always` and `nothing` are values of the field, never members of that list.** A kind narrows; these two decline to. As a list member `always` could sit beside a condition it renders dead — `[always, path: "src/**"]` parses, validates and silently ignores the path under OR semantics — and a form whose invalid state cannot be written needs no rule forbidding it.

*The field's declared `field_type` is `list_or_keyword`, which is new. `field_type` is an open vocabulary ([Field declarations](#field-declarations)), so this adds a name rather than a mechanism, and it is declared where a validator will look for it: the built-in `policy` and `workflow` Type Definitions.*

**Absent means `nothing`.** A Document that declares no `matches` is one nothing surfaces on its own behalf, and a consumer MUST NOT read the omission as a claim to be delivered unconditionally. **The expensive reading is the one that has to be asked for**, because a Document acquiring a permanent claim on a reader's attention by an author forgetting a field is the costliest possible default.

**Entries combine with OR: any one arising is enough.** This is a closed vocabulary and not an expression language — there is no AND, no negation and no grouping, because the moment those exist a consumer needs a parser and this stops being a list. Glob syntax already absorbs the common compound cases (`src/**/*.py` is *Python and under src*), and a Document needing more than that writes it in its body.

**An unknown kind is a Document that never arises**, which is indistinguishable from one whose subject has not come up. A consumer SHOULD report it rather than ignore it, and MUST NOT reject the Document for it ([Frontmatter layout and conformance](#frontmatter-layout-and-conformance)).

**`applies_to` was this field's name through `v0.0.13` and is gone.** It is not deprecated and consumers do not read it — a name nobody writes is not compatibility, it is a second spelling every reader has to know about. **The name is released** and is no longer reserved.

*Why the rename: the old name obliged an author to write a false sentence. `applies_to: everything` claims a Document governs everything, and none does — what a rule governs is stated in its body, and no frontmatter value widens or narrows it. The field says what makes a Document **surface**, which is a smaller and honest claim. The vocabulary had also outgrown the name: `path` is a target, but `event` is a moment, and nothing about a moment is a resource a rule scopes over.*

**It says nothing about loading.** What a consumer does with knowing *when a subject arises* — put the Document in front of a reader then, keep it always, do nothing — is the consumer's business (see the note at the end of [Field presence](#field-presence)).

**`policy` also declares `on_violation`; `workflow` does not.** A rule can be broken at a moment and something can act on that. The only way to fail a procedure is not to run it, which is the **absence** of an action — detecting absence needs state no consumer is obliged to keep, so the format does not ask for it.

**`event` reaches what the others cannot: a lifecycle point, however it is arrived at.** `command: git commit` catches that literal invocation; `event: before-commit` catches the point itself. A Document may reasonably declare both, and under OR semantics that is redundancy in the useful direction.

## Reserved files
**The rule, and it is the whole of it:**

> **ALL CAPS names a file that speaks for the thing containing it. Lowercase names one of the things contained.**

**In plain terms:** a file in CAPITALS is the folder's own file — it tells you what the folder *is*. Everything in lowercase is one of the things the folder *holds*. The test, when you are unsure: *is this file describing what surrounds it, or is it one of the things surrounded?*

`BUNDLE.md` speaks for its Bundle; `LOG.md` for its directory's history. A Type Definition at `_types/bundle.md` does not — it describes what a Bundle *is*, living inside one thing while being about another. A template at `templates/bundle.md` is a pattern for making one. Both are content and both stay lowercase. **The rule excludes them rather than exempting them**, which is why it needs no list of exceptions.

Where a name is shared with an outside convention, that convention's casing wins. `README.md` and `LICENSE` arrive uppercase anyway.

**The casing is a gate on participation, not decoration.** Nobody types all caps by accident, so a file joins the format's machinery only when somebody deliberately made it so — and getting it wrong fails in the safe direction: a `bundle.md` is an ordinary Document, ignored rather than silently treated as a manifest. Somebody who writes lowercase either is not using these tools or does not want that file wired up, and both are cases where wiring it up anyway would be wrong.

**It is the inverse of why `README.md` can carry no rules.** People edit a README without knowing any exist — universal permission semantics that no specification overturns. An all-caps name cannot be opted into by accident, so its rules bind only somebody who went looking for them. **A consumer MUST NOT depend on the content or structure of `README.md`.**

Directories are outside the rule and keep their own convention. `_types/` already says *structural rather than content* with its underscore, and a second signal for one meaning is worse than one.

- **`BUNDLE.md`** — the Bundle's own Document, at its root, with `type: bundle`. It is how a Bundle describes itself; see below. Recommended.
- **`LOG.md`** — append-only history for a directory, newest first. Creating it is optional, but when it exists writers MUST append rather than rewrite.
- **`_types/`** — Type Definitions ([Type extensions](#type-extensions)).

> **Withdrawn:** `index.md` previously reserved *derived navigation for a directory; a rebuildable cache, not a source of truth.* Its structure was never specified, nothing implemented it, and the name is one static site generators resolve in lowercase — so keeping it would have been this rule's only exception. The name is released. A future reservation for derived navigation should take a name of its own.

### `BUNDLE.md`
A Bundle SHOULD describe itself in a `BUNDLE.md` at its root — an ordinary Document with `type: bundle`, carrying the Bundle's own metadata rather than any file's:

```yaml
---
type: bundle
version: 1.2.0
published: 2026-08-17
consumers: [patient, clinic]
description: Health knowledge — lab results, medications, and conditions.
---
```

| Field | Presence | Field type | Meaning |
|---|---|---|---|
| `type` | required | text | `bundle` |
| `version` | required | semver | this Bundle's version ([Field declarations](#field-declarations)) |
| `published` | recommended | date | when this version was published |
| `consumers` | optional | list of text | the kinds of consumer that may adopt this Bundle |
| `entrypoint` | optional | text | the Document ID ([Document ID](#document-id)) of where a reader should start |
| `description` | *inherited* | text | one line on what the Bundle holds — a core field ([Core fields](#core-fields)), so `optional`; a Bundle SHOULD still carry one |

`version` is required because a Bundle without one cannot be pinned, compared, or reported as outdated — a consumer can say nothing honest about it. It is the Bundle's *content* version, distinct from `lkf_version` ([Versioning](#versioning)), which is the format-grammar version.

**`consumers` is an open vocabulary, and LKF defines no values for it.** Where a distribution model has more than one kind of consumer — a repository and an organization, a workstation and a server, a patient and a clinic — a Bundle may say which of them it is for. LKF does not know what those kinds are and does not enumerate them; the values belong to whoever is distributing, exactly as `tags` ([Core fields](#core-fields)) is `list of text` and left loose.

**The values are kinds, never instances.** `consumers` names what may adopt this Bundle, not what has. It is a permission the publisher grants, and no consumer writes itself into a Bundle it adopts.

It is a list because a Bundle may legitimately apply to more than one kind, and that is the whole reason it is a field. A distributor sorting Bundles into directories by consumer kind can express only one, which forces the *publisher* to answer a question that often belongs to the *adopter*. Omitting `consumers` says nothing — not "no consumers" and not "all consumers" — and consumers MUST NOT reject a Bundle for its absence ([Frontmatter layout and conformance](#frontmatter-layout-and-conformance)).

**`entrypoint` names where to start reading.** A Bundle of any size gives a newcomer no way to tell which Document is the way in, and every consumer otherwise invents its own answer — first alphabetically, the longest one, the one matching the directory name. It carries a **Document ID** ([Document ID](#document-id)), not a link: `entrypoint: recording-decisions`.

It is a claim about **reading order** — *start here* — and nothing else. Whether that Document, or any other, is placed in front of a reader is a consumption question the format leaves alone.

`description` is inherited rather than declared: it is already a core field, and the built-in `bundle` Type Definition adds only `version`, `published`, `consumers` and `entrypoint`. A type *may* restate an inherited field to raise its presence ([Inheritance](#inheritance)); `bundle` has no need to, since `description` at `optional` is already what it wants.

Consistent with [Frontmatter layout and conformance](#frontmatter-layout-and-conformance), a Bundle missing `BUNDLE.md` is not thereby invalid — nothing in LKF rejects. Tools that distribute Bundles will reasonably require one.

> **Not yet fully specified:** the exact structure of `LOG.md`.

## Versioning
- Scheme: **semver `major.minor.patch`**, starting at **0.0.1** (the earliest, most-unstable tier — breaking changes are expected in `0.0.z`). patch = clarifications/errata; minor = backward-compatible additions; major = breaking. Fields are `deprecated` before removal.
- Published versions are **git tags**; the newest tag is the current version.
- A Bundle MAY declare an `lkf_version` on its root `BUNDLE.md` ([`BUNDLE.md`](#bundlemd)); a Document MAY override with its own (file-level wins). This is the *format-grammar* version — not the Bundle's content version (`version`, [`BUNDLE.md`](#bundlemd)), not a file's content version (git's job), and not a Type Definition's own `version`.

## Released names
**Names this specification once defined and no longer does.** A name here is
free: nothing reserves it, no consumer reads it, and a producer may use it for
unrelated domain data ([Frontmatter layout and conformance](#frontmatter-layout-and-conformance)).

| name | was | released in | replaced by |
|---|---|---|---|
| `preload` | a core field: when a Document should be placed in front of a reader | `v0.0.12` | nothing — delivery is a consumer's decision, derived ([Field presence](#field-presence)) |
| `compliance` | a field grading how strongly a rule obliged compliance | never specified; invented and withdrawn in the estate during `v0.0.13` | nothing — a `policy` binds by being one, and `on_violation` says what happens when it does not |
| `applies_to` | the field naming what makes a Document surface | `v0.0.15` | `matches` ([`matches`, declared by `policy` and `workflow`](#matches-declared-by-policy-and-workflow)) |
| `index.md` | a reserved file: derived per-directory navigation | `v0.0.14` | nothing — see [Reserved files](#reserved-files) |
| `concept` | a Document type for background | `v0.0.10` | `document` |
| `entry_point` | the Bundle field naming where a reader should start | `v0.0.17` | `entrypoint` ([`BUNDLE.md`](#bundlemd)) |

**Why this list exists rather than the same fact scattered through the
sections that once defined each name.** Three of these were recorded only in
the prose of the section that removed them, in three different phrasings, and
one — `compliance` — was never recorded here at all, because it never reached
the specification. **A reader asking "is `preload` still a thing?" had to read
three paragraphs and know about a fourth name that is absent.**

**It is also the list a tool can read.** A retired name appearing in a published
Document's *prose* is not a conformance question — [Frontmatter layout and conformance](#frontmatter-layout-and-conformance) stands, and such a Document
is valid — but it is usually a rule still instructing authors to declare
something nothing reads. **A consumer MAY report that; it MUST NOT reject the
Document for it.**

**A released name may be reserved again**, and doing so is a breaking change
even though nothing currently uses it — a Bundle that adopted the free name for
its own purposes would silently acquire the specification's meaning.

> Known gaps and deferred features are tracked in [`ROADMAP.md`](docs/ROADMAP.md).

## Survival
**How much you should expect this to last.** Not *how long* — that is far harder
to answer, and nobody can — and not *will it continue indefinitely*, which is
harder still. Default (when absent): `intended`.

| Value | Meaning |
|---|---|
| `experimental` | **No intentions.** It is out in the world to find out whether it earns its keep, and many experiments do not. Do not fall in love with it. |
| `intended` | **It is meant to exist and to stick around. Nothing is promised.** (Default.) |
| `promised` | **Committed to.** Withdrawing it is an event rather than an edit. |

**`intended` is the ordinary case, and a producer MAY leave it unwritten.** A
value that is honest for nineteen things in twenty is one nobody should have to
write nineteen times, which is what the default is for. Declaring it is equally
valid, and a producer who wants the record explicit SHOULD feel free.

**Where the field tends to earn its keep is at the two ends** — a warning, or an
undertaking. Both are deliberate, and both say something silence does not.

*That is an observation about how the values fall in practice, not a rule. This
specification does not say when a producer should declare a field.*

**Being used does not change it.** Somebody relying on an experiment has taken a
risk they were warned about, and their use creates no undertaking nobody gave.
**Only the publisher moves this value**, and only deliberately.

**Neither this field nor `lifecycle_status` ([Lifecycle](#lifecycle)) is an input to the other**, and
each is useful alone: a Document may declare `survival` and no lifecycle, or the
reverse, and a consumer reading one need not look for the other.

They are orthogonal in the sense [`verified` and trust tiers](#verified-and-trust-tiers) gives the word for trust. A Document can
be `stable` and not expected to last — reliable now, and dying soon — or
`provisional` with its survival `promised`, because the undertaking was given
early and the journey is not far along.

### Why it has a default at all

**Future intent is frequently and legitimately undecided**, and *we mean to keep
this and have promised nothing* is both the honest answer and the common one. A
default that states what a reader already infers from silence costs nothing and
hides nothing.

**The name was chosen for how it degrades.** A neglected Document saying
`intended` is out of date rather than untrue — where `maintained` or `supported`
would be making a false claim to everybody who reads it, and neglect is exactly
the state nobody returns to correct.

### Moving between values is an act, not a feeling

| | |
|---|---|
| `experimental` → `intended` | the moment you would be reluctant to delete it. `experimental` means *no intentions*, so the transition is when intentions form |
| `intended` → `promised` | the moment somebody else's work breaks if you withdraw it |

Neither is a matter of degree, and both are answerable by asking the publisher
one question.

**A Document that has been retired did not survive**, whatever it last intended.
That is a fact about what happened rather than a contradiction of the field.
