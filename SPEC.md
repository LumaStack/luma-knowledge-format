# Luma Knowledge Format — Specification

- **Version:** `v0.0.10`
- **Status:** Released. Pre-1.0 — the `0.0.z` tier is unstable; breaking changes may still ship until `1.0.0`.

## Abstract

The Luma Knowledge Format (LKF) is a format for representing knowledge, designed to be equally friendly to humans and agents. Its core is small by design — flexible and made to be extended — so it adapts to whatever you need to capture.

Knowledge lives in plain markdown files with YAML frontmatter at the top, which you can create and maintain however you like. LKF is built to share knowledge across teams, tools, or organizations, and is designed to require minimal tooling.

## 1. Conventions

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as in RFC 2119.

Two roles are referenced throughout:
- A **producer** writes LKF — a person, an agent, or an export pipeline.
- A **consumer** reads LKF — a human, an agent, a UI, a search index, or tooling.

## 2. Terminology

- **Knowledge Bundle (or Bundle)**: A self-contained, hierarchical collection of files — the unit of distribution. Every file in a Bundle is either a Document or an Asset. **Self-contained means nothing is fetched in order to read it:** its Documents, its Assets and every Type Definition its Documents use are present in the Bundle, so it may be moved, archived or copied whole and still be read offline. **It is a statement about lookup, not about relationships** — a Bundle may *name* another it expects alongside it, which is a claim about what should be adopted, never a lookup performed while reading.
- **Asset**: A file in a Bundle that is not a Document — a script, a template, an image, a binary. It carries no frontmatter and no `type`, and Type Definitions (§10) do not apply to it.
- **Attachment**: An Asset that a Document links to (§8). A relationship, not a category: the same Asset may be an Attachment of several Documents, and an Asset that nothing links to is nobody's Attachment.
- **Document**: A single unit of knowledge within a Bundle, represented as one markdown file with YAML frontmatter. Every Document declares a `type`; what it describes — a table, an API, a metric, a task, a lab result — is that type's business, not the format's.
- **Document ID**: The path of the Document's file within the Bundle, with the `.md` suffix removed.
- **Slug**: A Document's filename without its directory path or `.md` extension (e.g. `diffusion-models` for `wiki/concepts/diffusion-models.md`).
- **Document Type** (or **Type**): The value of a Document's `type` field — a short string naming the kind of Document (e.g. `task`, `note`, `lab_result`). An open vocabulary; consumers tolerate unknown types.
- **Type Definition**: A Document (with `type: type_definition`) that declares a type's contract — its fields, their obligations, and their field types (§10).
- **Field type**: The shape of a field's value (e.g. `text`, `number`, `wikilink`), declared in a Type Definition (§10.2). Distinct from a Document's `type`.
- **Frontmatter**: A YAML metadata block delimited by `---` at the top of a markdown file.
- **Body**: Everything in the file after the frontmatter.
- **Link**: A markdown link from one Document to another, expressing a relationship between them.
- **Source**: A material a Document derives from, external or internal to the Bundle, recorded in the `sources` frontmatter field.

## 3. Document ID

A Document's **ID** is its file path within the Bundle, with the `.md` suffix removed. For example, `wiki/concepts/diffusion-models.md` has the ID `wiki/concepts/diffusion-models`.

LKF does not define a separate identifier field; the ID is path-based. Renaming or moving a Document changes its ID, so producers SHOULD perform renames through tooling that rewrites inbound links (§8). Consumers MUST tolerate links whose target does not resolve (§8).

> Stable opaque identifiers were considered and deferred; see [`PRINCIPLES.md`](docs/PRINCIPLES.md) and the project rationale. Because links are name-based, introducing ids later is additive and optional.

## 4. Frontmatter layout and conformance

Core fields defined by this specification appear at the **top level** of the frontmatter, alongside any domain-specific fields (a flat layout — no nesting under a reserved map).

- The field names defined in §5–§7 are **reserved**; producers MUST NOT reuse them for unrelated domain data. The prefix `lkf_` is reserved for future core fields.
- Consumers MUST preserve unrecognized keys when rewriting a file, and MUST NOT reject a file for containing them.
- **Identifier casing (a recommendation).** Field names, `type` names, and `field_type` values prefer snake_case (lowercase words joined by `_`); Document slugs and IDs prefer kebab-case (`-`), since they are path- and URI-like. Like nearly everything here, this is a strong recommendation, not a hard rule — the only hard requirement is a non-empty `type` (Conformance, below).

**Conformance.** A file is a conformant Document if it has a parseable YAML frontmatter block containing a non-empty `type`. **This is the only hard requirement.** Consumers **MUST NOT** reject a Document for: missing recommended or optional fields, an unrecognized `type`, unknown extra keys, or unresolved links. Everything a type declares (§10) is *published intent*, not an enforced rule — validation is a **suggested framework** (§10.5), never a conformance gate, and it never rejects by default.

## 5. Field obligation

Every field — a core field here, or a domain field declared by a Type Definition (§10) — carries an **`obligation`**: how strongly it should be present. Values are stored as full lowercase words:

| Obligation | Meaning |
|---|---|
| `mandatory` | Expected on every Document of this type. |
| `recommended` | Not mandatory, but include it whenever the information is available; omit only when it genuinely doesn't apply or isn't known. |
| `optional` | May be present; its absence is unremarkable. |
| `deprecated` | Still accepted and read, but on its way out; migrate off it. |

Obligation describes *intent*. Whether and how a tool checks it is a suggested validation framework, not a rule (§10.5), and nothing about obligations changes whether a file is conformant (§4). The sole hard requirement remains a non-empty `type`.

### 5.1 Core fields

| Field | Obligation | Field type | Meaning |
|---|---|---|---|
| `type` | mandatory | text | What kind of Document this is. **The one hard conformance requirement (§4).** Consumers tolerate unknown types. |
| `title` | recommended | text | Human label; may fall back to the filename. |
| `description` | optional | text | One-sentence summary; used by indexes and search. |
| `tags` | optional | list of text | Categorization; typically nested via `/` (e.g. `ml/generative`). Kept intentionally loose — organizations define their own tag conventions. |
| `lifecycle_status` | optional | enum | `draft \| provisional \| stable \| archived`. §6. |
| `created` | optional | actor_event | Original author + creation time. **Immutable.** §7.1. |
| `modified` | recommended | actor_event | Last editor + last meaningful change. **Advances on edit.** §7.1. |
| `verified` | optional | list of actor_event | Independent confirmation events. §7.2. |
| `sources` | optional | list | Materials the content derives from (bespoke shape). §7.3. |
| `stale_after` | optional | date | The content SHOULD be re-checked after this date. |
| `preload` | optional | enum | `mandatory \| recommended \| optional`. How strongly this Document should be loaded before working with its Bundle. §5.2. |

> Some obligations above (`title`, `description`, `tags`, `verified`, `sources`) are working defaults pending final ratification.

### 5.2 `preload`

A Bundle is usually larger than any one task needs. `preload` lets a Document say how strongly it should be in front of a reader — human or agent — *before* work with its Bundle begins, so a consumer working to a budget knows what it may leave until later.

| Value | A consumer SHOULD | If it cannot |
|---|---|---|
| `mandatory` | load it before working with the Bundle at all | **fail, naming the Document.** Not a diminished start |
| `recommended` | load it upfront when able | proceed, and report that it did not |
| `optional` | leave it until something references it or a need arises | nothing — it was never going to be loaded unprompted |

**Absent means `optional`.** A genuine default rather than a meaning assigned to silence: here the weakest value is also the safe one, since failing to load something is recoverable while loading everything by default is not. (Contrast `consumers` in §11.1, where absence says *nothing* — there, both possible defaults would be wrong guesses.)

**`mandatory` is a hard requirement, not a strong preference.** A level that degrades quietly is a hint, and hints are ignored. A consumer that cannot load a `mandatory` Document refuses rather than proceeding without it. The cost of that falls on the author: marking too much `mandatory` makes a Bundle unusable in a constrained context, which is what keeps the level meaning anything. A Bundle's total `mandatory` weight is a requirement it imposes on every consumer, and is better surfaced where the Bundle is published than discovered when something fails.

**It says nothing about importance.** All three values are about *timing*. An `optional` Document may be the most valuable thing in a Bundle and simply not be needed until something asks for it. Retirement is `lifecycle_status` (§6), not this.

**Preload is always relative to what contains the thing.** A Document's `preload` is relative to its Bundle — *of this Bundle's Documents, which do I need ahead of the work* — and says nothing about whether the Bundle should be in play at all. That is a question for whoever adopted it, and this field does not answer it.

*Ahead of the work* is deliberately not pinned to a moment. In current practice it means what a consumer loads at the start of a session, and that is the clearest way to explain it — but the loading model is a property of the consumer, and a field carried by every Document that uses it has to outlive whatever the current one turns out to be.

## 6. Lifecycle: `lifecycle_status`

A Document's lifecycle stage — nascent to active, with `archived` as the retired terminal. Default (when absent): `provisional`.

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

Trust tier is **orthogonal** to `lifecycle_status`: a Document can be `provisional` yet human-reviewed, or `stable` yet only machine-confirmed.

### 7.3 `sources`

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

**Supported, and an anti-pattern.** A tool that *can* identify its actor should. `unknown:unknown` exists for genuine ignorance — a command invoked through one interface by both people and agents, with nothing to tell them apart — not as a default for tools that never asked. A body of Documents where most authors are `unknown:unknown` has thrown away provenance it could have kept, and no reader can tell which of those were unavoidable and which were laziness.

Prefer the most specific value available: `agent:opus-5` over `agent:unknown` over `unknown:unknown`. Where a tool has a way for the caller to say who is acting, the honest default is `unknown:unknown` and the expected practice is to pass the real one.

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

  > **The quotes are load-bearing, and omitting them fails silently.** `[[…]]` is YAML flow-sequence syntax, so an unquoted wikilink parses as a **nested array** rather than a string, with no error from any YAML parser:
  >
  > ```yaml
  > parent: [[topic-ml]]      # → [["topic-ml"]]   a list containing a list
  > parent: "[[topic-ml]]"    # → "[[topic-ml]]"   a string, as intended
  > ```
  >
  > A validator (§10.5) catches this as a value whose shape does not match its declared `field_type`, but nothing else will — the document stays conformant, and a consumer simply never resolves the link. Producers writing frontmatter wikilinks MUST quote them.

**Assets use ordinary markdown links.** `[[…]]` links a Document; `[…](…)` links anything else — an Asset, or an external address:

```markdown
Run [the setup script](scripts/setup.sh) before the first import.
```

An Asset link is a path relative to the linking Document, and MUST point inside the Bundle — a link reaching outside breaks self-containment, which is the property that lets a Bundle be copied whole. Whether the target *exists* is a separate question: as with Document links, an unresolved Asset link is legal (§4), and a consumer MUST NOT reject a Bundle for one.

The two forms are distinguishable on sight and neither needs the other's rules — an Asset has no slug and no ID, only a path.

Both forms resolve through the consuming tool's index. How a bare slug resolves to a full Document ID — and how ties between same-slug Documents are broken — is governed by the link-resolution rules, which are not yet specified (see [`ROADMAP.md`](docs/ROADMAP.md)). **Unresolved links are legal** — a missing target MAY simply represent not-yet-written knowledge. Renames rewrite inbound links atomically via tooling (§3).

## 9. Body conventions

The body is CommonMark. Producers SHOULD favor structural markdown (headings, lists, tables, code fences) over prose. Two portable extensions are supported:

- **Callouts** — `> [!note]`, restricted to the GitHub ∩ Obsidian intersection for maximum portability: `note`, `tip`, `important`, `warning`, `caution`. Unknown types fall back to `note` (they degrade to a plain blockquote anywhere).
- **Footnote attribution** — `[^id]` keyed to a `sources[].id` (§7.3).

## 10. Type extensions

Any `type` MAY declare a **contract** for its Documents — which fields they carry and what shape those fields take — so producers and consumers can discover exactly what, say, a `lab_result` expects, and tools MAY validate against it. This is how LKF stays a small core with an open, extensible edge.

Nothing in this section is a conformance requirement. A Type Definition publishes *intent*; §10.5 describes a suggested way to check it; §4 remains the only hard rule.

### 10.1 Type Definitions

A `type` is declared by a **Type Definition** — an ordinary Document with `type: type_definition`, living in the bundle's reserved `_types/` directory. Because a Type Definition is itself a Document, it is plain markdown, git-committed, and self-documenting (its body carries docs and examples).

```yaml
---
type: type_definition
defines: lab_result
extends: source
fields:
  test_name: { obligation: mandatory,   field_type: text,   desc: "e.g. LDL cholesterol" }
  value:     { obligation: mandatory,   field_type: number }
  unit:      { obligation: mandatory,   field_type: text }
  patient:   { obligation: mandatory,   field_type: wikilink, desc: "→ the person Document" }
  panel:     { obligation: recommended, field_type: list of wikilink }
  status:    { obligation: optional,    field_type: enum, values: [pending, final, corrected] }
---

# Lab Result

A single quantitative lab measurement. One file per result.
```

- `defines` — the `type` name this document governs.
- `extends` — a single parent type to inherit from (§10.3).
- `fields` — the field declarations (§10.2).
- `vendored_from` — where this copy came from, when it is a copy (below).

#### `vendored_from`

A Type Definition copied from elsewhere SHOULD record where it came from:

```yaml
vendored_from:
  resource: https://example.org/shared-types
  version: "0.1.0"
  at: 2026-08-22
```

**It is provenance, never a lookup.** Nothing fetches it, and a consumer that cannot reach `resource` reads the Document exactly as before — the contract is the local file and always was. Without this the copy is anonymous, and §10.4's vendoring model has no way to tell a current copy from a stale or edited one.

**It answers two questions, and the second is easy to miss.** *Is my copy still current?* — compare against `resource` at `version`, on demand. And *do two copies in one place agree?* — two Bundles that vendored the same type at different versions hold two contracts under one name, which §10.4 permits and which nothing else would surface.

### 10.2 Field declarations

Each entry under `fields` declares one field with up to four keys:

| Key | Meaning |
|---|---|
| `obligation` | how strongly the field should be present — `mandatory` / `recommended` / `optional` / `deprecated` (§5) |
| `field_type` | the shape of the field's value (below) |
| `desc` | a one-line human/agent description (surfaced by discovery tooling, §10.6) |
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
| `wikilink` | an internal `[[…]]` link to another Document in the bundle (of any `type`). **Quoted in frontmatter** — see the warning in §8 |
| `uri` | an external address (URL/URI) |
| `actor` | an actor string (§7.4) |
| `actor_event` | `{ by: actor, at: datetime }` |
| `list of <type>` | a list whose items are any of the above (e.g. `list of wikilink`) |

A **relationship** (a typed edge in the Document graph) is simply a field whose `field_type` is `wikilink` or `list of wikilink` — the field's *key* names the relationship (`depends_on`, `parent`, `patient`).

### 10.3 Inheritance

- **`extends`** names a single parent type (single inheritance). A type inherits all of its parent's fields and adds its own.
- Every type implicitly extends the built-in **`document`** root, which supplies the LKF core fields (§5.1). A Type Definition therefore declares only its *domain* fields — never the core fields, except to strengthen an obligation as below. This is self-hosting: `type_definition` is itself a type that extends `document`.
- **Add-only.** A type may only *add* fields. It MUST NOT remove an inherited field, nor redefine its `field_type`, its `values`, or its meaning — core or domain. This keeps every inherited field's meaning stable everywhere the type is used.
- **Obligation may be strengthened, never weakened.** A type MAY raise an inherited field's obligation — `optional` → `recommended` → `mandatory` — by redeclaring the field with a higher `obligation` and nothing else changed. It MUST NOT lower one. Where a field is declared at several points in a chain, the **strongest obligation wins**; an attempted weakening SHOULD be reported rather than honoured.

  `deprecated` is not on that ladder and is not reachable this way. It states something about a field's future rather than its strength, so a type inheriting a `deprecated` field may not mandate it — a field both deprecated and required is a contradiction rather than a precedence question.

  **This is consistent with add-only because obligation is not meaning.** The field means exactly what its declaring type said; a subtype only states how strongly *it* expects the field. Nothing becomes non-conformant either — obligation describes intent (§5) and the sole hard requirement remains a non-empty `type` (§4) — so a consumer that knows only the parent and one that knows the subtype may reach different completeness verdicts, and each is right at its own level.

  **Without this, a type whose semantics rest on inherited fields cannot state them.** Where a type's growth stage *is* `lifecycle_status` and its age *is* `created` — both `optional` on the root — the type has no way to say that a Document missing either is incomplete, and its own contract calls unremarkable exactly the omissions that break it.

### 10.4 Resolution and namespacing

- **The Bundle is the resolution scope, and that has a consequence worth stating.** Because a contract is found in *this* Bundle's `_types/`, two Bundles may hold different versions of the same type without contradiction — each one's Documents are checked against the copy that travelled with them. This is the scoping mechanism prose does not have, and it is why vendoring a type is safe where duplicating a policy would not be.

  **A Document outside every Bundle has no such scope.** Nothing prevents a Document living beside Bundles rather than inside one — describing a repository, or a place Bundles are published from. Such a Document declares a `type` like any other, and the format offers no rule for where its contract is found: there is no Bundle to look in. **Whoever puts a Document there owes it an answer**, and where two Bundles disagree about that type, nothing decides between them.

- **Resolution.** To find a type's contract, a tool looks in exactly two places: the format's **built-in types** (`document`, `workflow`, `policy`, `bundle`, `type_definition`) and the bundle's **`_types/`** directory. The built-ins ship as real Type Definitions in this repository's `bundle/` directory — itself a Bundle, so that the unit of distribution is exactly the types and not the project around them — so they are both a normative rendering and a worked example; a tool MAY supply them itself rather than requiring every bundle to vendor them. There is no remote lookup — a shared type library is used by **vendoring** (copying the `_types/*.md` you want into your own bundle), so a bundle is always self-contained.
- **Built-in names.** The names `document`, `workflow`, `policy`, `bundle` and `type_definition` belong to the format; a bundle SHOULD NOT redefine them. Doing so is legal — the permissive-conformance law (§4) means no consumer rejects a bundle for it — but unwise: a redefinition travels inside the bundle while every tool and every other bundle still assumes the format's meaning.
- **Two base types, because the third way of reaching a consumer is the root itself.** `workflow` and `policy` declare no fields between them. They are not labels for subjects — they name how a Document is *engaged with*:

  | type | how it reaches a consumer |
  |---|---|
  | `workflow` | **invoked** — loaded while it is being followed, absent otherwise |
  | `policy` | **standing** — kept present, because a rule consulted only when somebody thinks to look is not governing anything |

  **The third way — retrieved when relevant, pulled in by need — needs no type, because it is what `document` already is.** A plain Document is the one a consumer fetches when it bears on the work. Naming that mode would be naming the default, and **a type that names the default dispatches on nothing**: every consumer already treats anything without a more specific type exactly that way.

  That is what closes the set rather than leaving it open to any word a Bundle finds useful. **A further base type would have to name a way of engaging that is neither invoked, nor standing, nor the default** — not a further subject matter.

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

  **A built-in is the format's only mandatory surface.** Everything else here is permissive by law (§4) — unknown types tolerated, missing fields tolerated, unresolved links tolerated, and no consumer may reject a Document for any of it. This list is the one place the format *requires* something of every implementation.

  **So the question is never whether a type is important. It is whether a consumer that ignored it would fail to read a conformant Document, or engage with one in a way this specification says is wrong.** Both count, and they fail differently: without `bundle` or `type_definition` there is no Document ID and no way to obtain a contract, so nothing can be read at all; without `workflow` or `policy` a Document parses perfectly and is then kept when it should be invoked, which is a rule this specification states and the consumer got wrong.

  ***My tooling would break* is the wrong kind of broken.** It is true of every domain type ever written. A consumer ignoring one still reads the Document correctly — as a plain `document`, which is what it is — and merely does not participate in something built on top of the format. **That is the format working, not failing.**

  **The cost of a built-in is a word taken from everyone, permanently.** An unprefixed name belongs to the format for good: every Bundle in every domain must then avoid it, and releasing one later collides with whoever defined it privately in the meantime. So *important to us* is not an argument for this list — **importance is what a namespace is for**, and a namespaced type costs nobody anything.

  **A cheap further check: does it change at the format's rate?** A built-in's contract is versioned with the format. A type that gains fields as somebody's tooling matures drags the format's version along with it, and a specification whose releases track one adopter's roadmap has stopped being a specification.

  **Removing a built-in is cheaper than adding one.** Removal costs a deprecation cycle and a frontmatter migration. A late addition costs the same migration *plus* a collision with every Bundle that had already defined the name for itself.

  **That asymmetry is a tiebreaker, not an entry route.** It applies only to a candidate that has already cleared the checks above and is still genuinely balanced — there, admitting now and deprecating later is the cheaper error. It is not a reason to admit something that failed them, because a namespace costs nothing and is available immediately.

- **Namespacing.** **Unprefixed means the format defines it; a prefix means somebody else does.** That is the whole convention, and it lets a reader tell a type's origin from its name without a lookup — `workflow` is LKF's, `acme/deploy_check` is an organization's.

  A `type` published beyond the Bundle that wrote it SHOULD therefore be namespaced — typically by domain (`health/lab_result`, `finance/invoice`) or organization. At larger scale a team or department dimension MAY be added to disambiguate (e.g. `sales/report`, `engineering/report`). These are examples, not a mandated scheme: namespace however fits your context, or not at all.

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

Because Type Definitions are just files, humans and agents discover a type's contract the same way: read `_types/<type>.md`. Tooling may wrap that in a lookup command and needs no index to do so — the file *is* the contract, so reading it directly is always available and never stale.

## 11. Reserved files

- **`bundle.md`** — the Bundle's own Document, at its root, with `type: bundle`. It is how a Bundle describes itself; see below. Recommended.
- **`index.md`** — derived navigation for a directory; a rebuildable cache, not a source of truth. Optional.
- **`log.md`** — append-only history for a directory, newest first. Creating it is optional, but when it exists writers MUST append rather than rewrite.
- **`_types/`** — Type Definitions (§10).

### 11.1 `bundle.md`

A Bundle SHOULD describe itself in a `bundle.md` at its root — an ordinary Document with `type: bundle`, carrying the Bundle's own metadata rather than any file's:

```yaml
---
type: bundle
version: 1.2.0
published: 2026-08-17
consumers: [patient, clinic]
description: Health knowledge — lab results, medications, and conditions.
---
```

| Field | Obligation | Field type | Meaning |
|---|---|---|---|
| `type` | mandatory | text | `bundle` |
| `version` | mandatory | semver | this Bundle's version (§10.2) |
| `published` | recommended | date | when this version was published |
| `consumers` | optional | list of text | the kinds of consumer that may adopt this Bundle |
| `entry_point` | optional | text | the Document ID (§3) of where a reader should start |
| `description` | *inherited* | text | one line on what the Bundle holds — a core field (§5.1), so `optional`; a Bundle SHOULD still carry one |

`version` is mandatory because a Bundle without one cannot be pinned, compared, or reported as outdated — a consumer can say nothing honest about it. It is the Bundle's *content* version, distinct from `lkf_version` (§12), which is the format-grammar version.

**`consumers` is an open vocabulary, and LKF defines no values for it.** Where a distribution model has more than one kind of consumer — a repository and an organization, a workstation and a server, a patient and a clinic — a Bundle may say which of them it is for. LKF does not know what those kinds are and does not enumerate them; the values belong to whoever is distributing, exactly as `tags` (§5.1) is `list of text` and left loose.

**The values are kinds, never instances.** `consumers` names what may adopt this Bundle, not what has. It is a permission the publisher grants, and no consumer writes itself into a Bundle it adopts.

It is a list because a Bundle may legitimately apply to more than one kind, and that is the whole reason it is a field. A distributor sorting Bundles into directories by consumer kind can express only one, which forces the *publisher* to answer a question that often belongs to the *adopter*. Omitting `consumers` says nothing — not "no consumers" and not "all consumers" — and consumers MUST NOT reject a Bundle for its absence (§4).

**`entry_point` names where to start reading.** A Bundle of any size gives a newcomer no way to tell which Document is the way in, and every consumer otherwise invents its own answer — first alphabetically, the longest one, the one matching the directory name. It carries a **Document ID** (§3), not a link: `entry_point: recording-decisions`.

It is deliberately *not* the same claim as `preload: mandatory` (§5.2), though in a small Bundle the same Document usually carries both. Entry point is reading order — *start here* — while `preload` is context presence — *have this available*. A Bundle may need three Documents loaded and still have one place to begin.

`description` is inherited rather than declared: it is already a core field, and the built-in `bundle` Type Definition adds only `version`, `published`, `consumers` and `entry_point`. A type *may* restate an inherited field to raise its obligation (§10.3); `bundle` has no need to, since `description` at `optional` is already what it wants.

Consistent with §4, a Bundle missing `bundle.md` is not thereby invalid — nothing in LKF rejects. Tools that distribute Bundles will reasonably require one.

> **Not yet fully specified:** the exact structure of `index.md` and `log.md`.

## 12. Versioning

- Scheme: **semver `major.minor.patch`**, starting at **0.0.1** (the earliest, most-unstable tier — breaking changes are expected in `0.0.z`). patch = clarifications/errata; minor = backward-compatible additions; major = breaking. Fields are `deprecated` before removal.
- Published versions are **git tags**; the newest tag is the current version.
- A Bundle MAY declare an `lkf_version` on its root `bundle.md` (§11.1); a Document MAY override with its own (file-level wins). This is the *format-grammar* version — not the Bundle's content version (`version`, §11.1), not a file's content version (git's job), and not a Type Definition's own `version`.

> Known gaps and deferred features are tracked in [`ROADMAP.md`](docs/ROADMAP.md).
