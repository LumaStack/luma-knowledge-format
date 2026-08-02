# LKF Roadmap

Open questions and deferred features for the Luma Knowledge Format. `SPEC.md` describes only what is settled; this file tracks what is not.

## Undecided — needs a decision before it can be specified

- **Field-level ratification** — confirm the working-default levels in `SPEC.md` §5.1 (`title`, `description`, `tags`, `verified`, `sources`).
- **Type-extension rules** (§10) — property-type vocabulary, `extends`/inheritance and conflict resolution, tool-default vs. bundle precedence, validator severities.
- **Link resolution** — the algorithm and slug rules (uniqueness scope within a bundle, ambiguity handling). Reintroduce `aliases` here; alternate-name resolution is meaningless without the resolution rules.
- **Reserved-file formats** — the exact structure of `index.md` and `log.md` (§11).

## Deferred features — postponed, may return in a later version

- **`obligation: conditional`** — a field that is mandatory *only when* a stated condition holds, carrying a `when:` predicate (ISO 19115-style). Deferred from v0.0.1; `obligation` was chosen as the field-declaration key partly so this can be added later without a rename.
- **User-defined composite field types** — LKF ships the built-in `actor` and `actor_event` field types; arbitrary/user-defined nested object shapes are deferred.
- **Hierarchical slug/path field type** — a validated `field_type` for `/`-separated slug hierarchies (tags, categories, taxonomies). For now `tags` is `list of text`, kept loose; the type — and a good name (`path`/`breadcrumb`/`slug-path` all had drawbacks) — can come later.
- **Multiple inheritance for types** — v0.0.1 allows a single `extends` parent; multiple parents (with conflict-resolution rules) are deferred — wanted eventually, not needed now.
- **Domain-field override in inheritance** — v0.0.1 is add-only (a type may only *add* fields, never redefine inherited ones); letting a child override an inherited domain field is deferred, and may never be needed.
- **Stable identifiers** — opaque ids decoupled from path; additive when added (links stay name-based).
- **`aliases`** — ships together with link resolution (above).
- **`confidence`** and **`volatility`** — trust/freshness fields considered but held.
- **Concept-level `owner`** — accountability/stewardship, distinct from a source's `author`.
- **Attestation** — verifying that a computed value was produced by a sanctioned method (OKF's "Attested Computation"). Out of scope for MVP and adoption is uncertain, but worth revisiting in a later version.
