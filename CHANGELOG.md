# Changelog

Notable **specification** changes to the Luma Knowledge Format, newest first. The point of this file: see what changed *between versions* at a glance, without reading every commit. It records behavior-affecting changes and omits edits that don't change behavior (wording, typos, formatting, examples).

Format follows [Keep a Changelog](https://keepachangelog.com); versions follow [semver](https://semver.org). See [`GUIDELINES.md`](docs/GUIDELINES.md) for how this file is maintained.

## [Unreleased]

## [0.0.10] — 2026-08-22

### Removed
- **`concept` is removed as a built-in type** *(breaking).* It extended `document`, declared no fields, and no consumer ever treated it differently from the root — which §10.4's own test calls *falsified rather than merely unused*. It was marked *under review* in `0.0.9`; this finishes the job.
  **What kept it was its retrieval mode, and that did not need a type.** *Retrieved when relevant* is what a plain `document` already is, so §10.4 now describes **two** base types rather than three: `workflow` is invoked, `policy` stands, and the third way is the root itself. **A type that names the default dispatches on nothing** — every consumer already treats anything without a more specific type exactly that way. The set is still closed: a further base type would have to name a way of engaging that is neither invoked, nor standing, nor the default.
  **The name is released rather than reserved.** `ROADMAP.md` had held the type on the grounds that removing a name and re-adding it later collides with every Bundle that defined it privately in between. That cost is real and was accepted, because the deferral had begun costing more: *under review* does not stop adoption, and five Documents across two published Bundles had declared it — none for a reason `document` could not serve. A name that is noise stops being harmless once people use it.
  *Migration:* two forms, and the second is easy to miss.
  **`type: concept` becomes `type: document`.** Nothing else changes — the two were structurally identical, which is why the type went.
  **`extends: concept` in a Type Definition becomes `extends: document`.** `0.0.6` said outright that *"`extends: concept` keeps working unchanged"*, so anyone who read that has it in their type definitions; this release retracts it. **The contract is unaffected either way**: `concept` extended `document` and added nothing, so a type that inherited through it inherits exactly the same fields directly. Dropping the line entirely is equally correct, since every type implicitly extends `document` (§10.3).
  A Bundle that genuinely dispatches on the distinction may define `concept` in its own `_types/`, subject to the usual bar in §10.4: name the consumer, and name what it does differently.

### Added
- **`vendored_from` on a Type Definition** (§10.1) — `resource`, `version` and `at`, recording where a copied type came from. §10.4 makes vendoring the only way to share a type, and a vendored copy was previously anonymous: nothing could tell a current copy from a stale or edited one.
  **It is provenance, never a lookup.** Nothing fetches it, and a consumer that cannot reach `resource` reads the Document exactly as before — the contract is the local file and always was. This is deliberately not a search path, an environment variable, or a locator embedded in `type:`; each of those makes a Document's meaning depend on where it is read rather than on what it says.
  **It answers two questions, and the second is easy to miss.** *Is my copy current?* — compare against `resource` at `version`, on demand. And *do two copies in one place agree?* — two Bundles that vendored one type at different versions hold two contracts under one name, which the resolution scope permits and nothing else would surface.

### Changed
- **Self-containment is now defined as a property of lookup, not of relationships** (§2). Previously *"nothing it needs lives outside it"*, which a Bundle naming another Bundle would falsify — and which said nothing about the thing actually at stake. It now reads: **nothing is fetched in order to read it**, so a Bundle may be moved, archived or copied whole and still be read offline.
  The distinction matters because the old wording defended the wrong perimeter. What must never happen is a *lookup* while reading — a remote fetch, a search up the directory tree, a path from the environment. What is harmless is a Bundle *naming* another it expects alongside it, which is a claim about what should be adopted.
- **The bar for a built-in type is sharpened, and it is a narrowing** (§10.4). Four additions, no removals:
  **A built-in is the format's only mandatory surface.** Everything else is permissive by law (§4) — unknown types tolerated, missing fields tolerated, nothing rejected. So the question is never whether a type is *important*; it is whether a consumer that ignored it would fail to read a conformant Document, or engage with one in a way this specification says is wrong. ***My tooling would break* is explicitly the wrong kind of broken** — true of every domain type ever written, and a consumer ignoring one still reads the Document correctly as a plain `document`.
  **The cost of a built-in is a word taken from everyone, permanently**, which is why *important to us* is not an argument — **importance is what a namespace is for**. A cheap further check: **does it change at the format's rate?** A type that gains fields as somebody's tooling matures would drag the format's releases behind that roadmap.
  And the removal-is-cheaper-than-addition asymmetry is now **a tiebreaker rather than an entry route** — it applies to a candidate that already cleared the checks and is still balanced, not to one that failed them.
- **Namespacing is stated as a convention rather than a suggestion** (§10.4). **Unprefixed means the format defines it; a prefix means somebody else does** — so a reader can tell a type's origin from its name without a lookup. A `type` published beyond the Bundle that wrote it SHOULD be namespaced.
- **The Bundle is named as the resolution scope, with its consequence** (§10.4). Because a contract is found in *this* Bundle's `_types/`, **two Bundles may hold different versions of one type without contradiction** — each Bundle's Documents are checked against the copy that travelled with them. That is the scoping mechanism prose lacks, and it is why vendoring a type is safe where duplicating a policy would not be.
  **The exception is now stated too:** a Document living outside every Bundle has no such scope, the format offers no rule for where its contract is found, and whoever puts a Document there owes it an answer.
- **§10.6 no longer names a particular vendor's command.** Discovery is reading `_types/<type>.md`; tooling may wrap that and needs no index to do it.
- **A subtype may strengthen an inherited field's obligation** (§10.3) — `optional` → `recommended` → `mandatory`, by redeclaring the field with a higher `obligation` and nothing else changed. Weakening stays forbidden, and where a field is declared at several points in a chain the strongest obligation wins.
  **The gap it closes: a type whose semantics rest on inherited fields could not state them.** Where a type's growth stage *is* `lifecycle_status` and its age *is* `created` — both `optional` on the root — add-only left no way to say a Document missing either is incomplete. The type's own contract called unremarkable exactly the omissions that break it, and the workaround in practice was a sentence of prose telling readers to treat a field as required despite what the contract said.
  **Consistent with add-only, because obligation is not meaning.** That rule exists to keep an inherited field's meaning stable everywhere it is used; the field still means what its declaring type said, and a subtype only states how strongly *it* expects it. Removing a field, or changing its `field_type`, `values` or meaning, is still forbidden — §10.3 now says so in those terms rather than by the broader word *redefine*.
  **Nothing becomes non-conformant.** Obligation describes intent (§5) and the sole hard requirement is still a non-empty `type` (§4), so a consumer that knows only the parent type and one that knows the subtype may disagree about whether a file is complete, and each is right at its own level.
  **`deprecated` is not on the ladder** and is not reachable this way — it states something about a field's future rather than its strength, so a type inheriting a deprecated field may not mandate it.
  *Migration:* none. This permits what was previously forbidden, so every existing Type Definition and Document stays valid. Validators gain a rule; documents do not change.

## [0.0.9] — 2026-08-19

### Added
- **`workflow` and `policy` as built-in types** (§10.4) — both field-free, both extending `document`. A `workflow` is a procedure that gets *transformed*: consumers project it into whatever form their harness expects, selected by the type rather than by where the file sits. A `policy` is a course of action that a consumer keeps as *standing context*, where a workflow is loaded on invocation — the type is what tells otherwise-identical prose apart.
  **They complete a three-way partition rather than adding two labels.** `concept` is *retrieved when relevant*, `workflow` is *invoked*, `policy` is *standing* — three ways prose reaches a consumer, and the set is closed at three because a fourth would have to name a fourth way of engaging rather than a fourth subject.
  `policy` must be built in rather than defined per Bundle for a specific reason: tooling that makes a policy hard to ignore can only be written against a name the format guarantees. A type each Bundle defines privately is one no consumer can rely on finding. Its dispatch difference is *intended* rather than shipped, which §10.4 permits provided the consumer and the difference are named — and if it never acquires them it has been falsified, not merely unused.
  Corroborating count, offered as corroboration only: across the first seven Bundles written against this format, `workflow` was defined independently in 7 of 7 and `policy` in 6 of 7 — thirteen byte-identical files. **That sample is biased** and the spec now says so: all seven are governance Bundles, so finding governance vocabulary throughout them proves less than it appears to.
- **The test for declaring a type at all** (§10.4) — *name the consumer, and name what it does differently.* If you cannot name both, the `type` is a label, and a label costs a name every other Bundle must avoid. Three forms are given: **checked** (a validator has a contract), **transformed** (something converts it into another form), **consulted** (the format's own machinery reads it to handle other Documents). Wanting to *enumerate* Documents of a kind is explicitly not enough, since that works for any type and would grow the list without limit.
- **The further test for being built in** (§10.4) — earning a name is not enough; a built-in must also be **unavoidable**, either *structural* (the format's machinery depends on it) or *ubiquitous* (nearly every Bundle defines it independently, counted rather than asserted — **and then check what you counted**, since a sample drawn from one domain finds that domain's vocabulary everywhere). Records that removal is cheaper than late addition, and that the asymmetry favours admitting a balanced case now.

### Deprecated
- **`concept` is marked *under review*** and should be a deliberate choice rather than a default. It has existed since `v0.0.1` with no consumer treating it differently from `document`, which §10.4's own test calls *falsified rather than merely unused*. Its retrieval mode — pulled in when relevant — is the case for keeping it, and that case is a claim nothing yet implements. **What would settle it is a durable knowledge base**, the thing `concept` was written for and which nobody has built with this format. Held rather than removed, because a removed name re-added later collides with every Bundle that defined it privately in between.

## [0.0.8] — 2026-08-18

### Added
- **`preload` as a core field** (§5.1, §5.2) — `optional`, an enum of `mandatory | recommended | optional`, saying how strongly a Document should be loaded before working with its Bundle. A Bundle is usually larger than any one task needs, and nothing let a Document say it was the spine rather than reference material.
  It sits on the root rather than on `bundle` because any Document may carry it. **`mandatory` is a hard requirement**: a consumer that cannot load such a Document fails and names it, rather than starting diminished — a level that degrades quietly is a hint, and hints are ignored. The cost falls on authors, which is what keeps the level meaning anything.
  Absent means `optional`, and §5.2 states why that is a genuine default here while absence of `consumers` means nothing: the weakest value is also the safe one.
- **`entry_point` on the built-in `bundle` type** (§11.1) — `optional`, a Document ID (§3) naming where a reader should start. Without it every consumer invents its own answer — first alphabetically, longest file, name matching the directory. It is deliberately distinct from `preload: mandatory`: entry point is reading order, `preload` is context presence, and a Bundle may need several Documents loaded while still having one place to begin.
  Both are additive and non-breaking; existing Bundles remain valid unchanged.

## [0.0.7] — 2026-08-18

### Changed
- **`applies_to` on the built-in `bundle` type is renamed `consumers`** (§11.1). Same obligation, same field type, same open vocabulary — only the name changes. Two reasons, both structural rather than stylistic. Every other field on `bundle` states a *property* of the Bundle (`version`, `published`, `description`) while `applies_to` stated a *relation*, and in policy languages `applies_to` conventionally means enforcement scope — "this rule applies to these targets" — whereas this field is about eligibility. `consumers` also matches how the format names every other collection: `tags` holds tags, `sources` holds sources.
  The spec now says outright that the values are **kinds, never instances** — the one ambiguity a plural noun introduces, and cheaper to close in a sentence than to name around.
  *Migration:* rename the key. Nothing published declares `applies_to`, so in practice there is nothing to migrate — the field existed for a matter of hours.
  Breaking, shipped as a patch under the pre-1.0 clause (`v0.0.z` is explicitly unstable). No deprecation cycle, because deprecating a field with no users would leave the specification carrying a dead name to protect nobody.

## [0.0.6] — 2026-08-18

### Added
- **`applies_to` on the built-in `bundle` type** (§11.1) — `optional`, `list of text`, naming the kinds of consumer that may adopt a Bundle. **The vocabulary is open and LKF defines no values for it.** Where a distribution model has more than one kind of consumer — a repository and an organization, a workstation and a server — a Bundle may say which of them it is for. The format does not know what those kinds are, so the values belong to whoever is distributing, exactly as `tags` is `list of text` and left loose.
  It is a list because a Bundle may legitimately apply at more than one, and that is precisely why it has to be a field. The first consumer had sorted Bundles into directories by consumer kind; a directory can express only one, so it forced the *publisher* to answer a question that often belongs to the *adopter*, permanently and with no override. Absence says nothing — not "none", not "all" — and no consumer rejects a Bundle for it (§4).
  Additive and non-breaking: existing Bundles remain valid unchanged.

## [0.0.5] — 2026-08-17

### Changed
- **The built-in types moved to a `bundle/` directory** (§10.4). They previously sat at the repository root, which made the repository itself the Bundle — and therefore made the "unit of distribution" include the changelog, the guidelines and the roadmap. Project apparatus is not knowledge, and a Bundle that drags its own governance along is not self-contained in any useful sense. `bundle/` is now exactly the unit: `bundle.md` and `_types/`.
  `SPEC.md` deliberately stays at the repository root. It is the primary artifact and the first thing a reader looks for, and burying it to tidy the secondary files would be the wrong trade. This Bundle is the built-in types; a Bundle carrying the specification as loadable knowledge would be a different one.
  *Migration:* if you vendored built-in Type Definitions from this repository, they are now under `bundle/_types/` rather than `_types/`. The file contents are unchanged.

## [0.0.4] — 2026-08-17

### Added
- **The built-in types ship as real Type Definitions** in this repository's `_types/` — `document`, `concept`, `bundle` and `type_definition`. They were previously prose tables only, which meant the format claimed to be self-hosting while its own types were the one thing not expressed in it. The repository is now itself a Bundle: the Type Definitions are its Documents, and the specification, readme, licence and `docs/` are its Assets — legal only since Assets exist. A tool MAY supply the built-ins itself rather than requiring every bundle to vendor them.
- **Assets** (§2, §8). A Bundle's files are now partitioned: every file is either a Document (frontmatter with a `type`) or an **Asset** (no frontmatter, no type, outside Type Definition validation). An **Attachment** is an Asset a Document links to — a relationship rather than a category, so the same Asset may be an Attachment of several Documents, and one nothing links to is nobody's Attachment. Scripts, templates and binaries previously had no standing in a Bundle at all.
- **Asset links** (§8). `[[…]]` links a Document; `[…](…)` links anything else. No new syntax, no change to ID or slug rules, and the two forms are distinguishable on sight. An Asset link MUST point inside the Bundle — reaching outside breaks self-containment — though as with Document links, an unresolved one stays legal.
- **`bundle` as a built-in type, and `bundle.md`** (§10.4, §11.1). A Bundle describes itself in a root `bundle.md` with `type: bundle`, carrying `version` (mandatory, `semver`), `published` and `description` (both recommended). `version` is mandatory because a Bundle without one cannot be pinned, compared, or reported as outdated — a consumer can say nothing honest about it.
- **`semver` field type** (§10.2). A version string as [semver.org](https://semver.org) defines it, including pre-release and build metadata (`1.0.0-alpha.1+build.5`). The `v` prefix is excluded deliberately — `v1.2.3` is a tag convention rather than a version, and accepting both spellings would mean two bundles could write the same version two ways and every consumer would have to normalize.

### Changed
- **`lkf_version` moved from the root `index.md` to `bundle.md`** (§12). §11 defines `index.md` as derived navigation — "a rebuildable cache, not a source of truth" — so the format-grammar version was living on a file any tool is entitled to regenerate, and a navigation rebuild could discard it. `bundle.md` is a source of truth.
  *Migration:* if a Bundle declares `lkf_version` on its root `index.md`, move it to `bundle.md`. Bundles that never declared one are unaffected.

- **BREAKING: the base object is a Document, not a Concept** (§2, §3, and throughout). `Concept` named the abstraction after one of its instances: a `task` is not a concept, nor is a `lab_result`, and the definition had to stretch to "a tangible asset, an abstract idea, or anything in between" to cover them. The spec's own prose already said the right word, describing a bundle as "a collection of knowledge documents". `document` is now the root type every type implicitly extends, and `concept` becomes an ordinary built-in type for knowledge-base entries, adding no fields of its own.

  *Migration:* **No file has to change.** "Concept" was the specification's word for the object, never a value anyone wrote in frontmatter — existing files are already conformant Documents. Three things to update, all outside your content: rename `Concept ID` to `Document ID` in tooling and prose; where your documentation says "Concept" meaning *any LKF file*, it now says Document; and where it means *a knowledge-base entry specifically*, that is now the `concept` type. `extends: concept` keeps working unchanged — `concept` extends `document` and adds nothing, so anything that inherited the core fields through it still does.

- **Redefining a built-in type is now `SHOULD NOT` rather than `MUST NOT`** (§10.4). The prohibition was the format's only hard "you may not", sitting inside a specification that is otherwise permissive by default and never rejects. It also over-applied: redefining `type_definition` is genuinely dangerous, while redefining the root type to *add* a field is useful and has no other mechanism, since §10.3 lets a type add fields but nothing lets you add to the root. One blanket rule forbade the useful case to prevent the harmful one. It is now discouraged rather than forbidden. Not a migration — this permits strictly more than before. If it later needs teeth, the shape is already in the format: §10.3 is add-only, so the rule would be that a redefinition may *add* to a built-in but never contradict it.
  The heading changed from "Reserved built-ins" to "Built-in names", freeing *reserved* for a possible future mechanism that reserves type names on behalf of others.

## [0.0.3] — 2026-08-15

### Added
- **`unknown` as an actor kind and as a producer** (§7.4). `unknown:unknown` records that the actor was not captured; `agent:unknown` records the kind without the identity. The grammar previously had no way to say *not recorded*, so a tool that could not tell who invoked it had to guess a plausible actor or omit `by` — and omitting it discards the `at` timestamp in the same `actor_event`. **Supported but an anti-pattern:** a tool that can identify its actor should, and `unknown:unknown` is for genuine ignorance rather than a default for tools that never asked.

## [0.0.2] — 2026-08-02

### Changed
- **Field type `concept-link` → `wikilink`** *(breaking).* The internal-link field type is now type-agnostic — it links to any Concept, not only `concept`-typed ones — and named by form to pair with `uri`.
  - *Migration:* replace `field_type: concept-link` with `field_type: wikilink` in Type Definitions.
- **Composite field type `actor-event` → `actor_event`** *(breaking).* snake_case, per the identifier-casing convention.
  - *Migration:* replace `actor-event` with `actor_event` in Type Definitions.
- **Reserved type `type-definition` → `type_definition`** *(breaking).* Same casing convention; the `_types/` directory and the reserved-name rule are otherwise unchanged.
  - *Migration:* rename `type: type-definition` to `type: type_definition`.
- **Identifier casing standardized.** Field names, `type` names, and `field_type` values prefer snake_case; Concept slugs and IDs prefer kebab-case (path/URI-like). A recommendation, not a hard rule.

## [0.0.1] — 2026-08-02

Initial release.

### Added
- **Core model** — files as Concepts, path-based identity, and permissive conformance (a non-empty `type` is the only hard requirement; consumers never reject).
- **Core fields** — `type`, `title`, `description`, `tags`, `lifecycle_status`, `created`, `modified`, `verified`, `sources`, `stale_after`.
- **Provenance & trust** — `created`/`modified` (author + timestamp), `verified` with derived trust tiers, structured `sources`, and the actor convention `<kind>:<producer>/<version>`.
- **Type extensions** — Type Definitions in `_types/`, the field-type vocabulary, field `obligation` (`mandatory`/`recommended`/`optional`/`deprecated`), single/add-only inheritance, vendored resolution, and validation as a *suggested framework — not a contract*.

[Unreleased]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.9...HEAD
[0.0.9]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.8...v0.0.9
[0.0.8]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.7...v0.0.8
[0.0.7]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.6...v0.0.7
[0.0.6]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.5...v0.0.6
[0.0.5]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.4...v0.0.5
[0.0.4]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.3...v0.0.4
[0.0.3]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.2...v0.0.3
[0.0.2]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.1...v0.0.2
[0.0.1]: https://github.com/LumaStack/luma-knowledge-format/releases/tag/v0.0.1
