# Changelog

Notable **specification** changes to the Luma Knowledge Format, newest first. The point of this file: see what changed *between versions* at a glance, without reading every commit. It records behavior-affecting changes and omits edits that don't change behavior (wording, typos, formatting, examples).

Format follows [Keep a Changelog](https://keepachangelog.com); versions follow [semver](https://semver.org). See [`GUIDELINES.md`](docs/GUIDELINES.md) for how this file is maintained.

## [Unreleased]

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
