# Changelog

Notable **specification** changes to the Luma Knowledge Format, newest first. The point of this file: see what changed *between versions* at a glance, without reading every commit. It records behavior-affecting changes and omits edits that don't change behavior (wording, typos, formatting, examples).

Format follows [Keep a Changelog](https://keepachangelog.com); versions follow [semver](https://semver.org). See [`GUIDELINES.md`](GUIDELINES.md) for how this file is maintained.

## [Unreleased]

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
