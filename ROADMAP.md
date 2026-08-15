# LKF Roadmap

Open questions and deferred features for the Luma Knowledge Format. `SPEC.md` describes only what is settled; this file tracks what is not.

## What `v0.1.0` would mean

Not a promise that the format is finished, or safe to lean on. Only that it has stopped being a guess.

`v0.0.z` says the format has been reasoned about and barely used. `v0.1.0` says it has been run for real in more than one place and held up there. That is a smaller claim than it sounds, and it is the honest one — all of this was designed before it met any data.

Three things would have to be true.

### It has been exercised, more than once

Two or three separate codebases writing and reading real Concepts — not examples, and not a corner chosen to avoid the awkward parts — reporting back that it worked for what they were doing.

The signal is not that nobody complained. It is that newcomers stop hitting the wall the first one hit. One consumer's opening review turned up four holes, three of them the same missing capability and one closed in `v0.0.3`. That rate is ordinary for a format meeting real data. The bump asserts the rate has come down, and only later consumers can show that.

### Type lookup has been run, not just written down

How a Type Definition gets found — the chain, where it ends, which copy wins when two disagree, what a vendored copy remembers about its origin — is open on several fronts (see *Undecided*). Writing the rule is the easy half. **Resolution fails quietly, because the wrong definition is still a definition**, so a rule nobody has run against two bundles that disagree has not been tested at all.

### The shape has stopped moving

Bundle layout, how types get copied between bundles, and which names the format claims for itself.

These cost the most to revisit, because they live in every consumer's files rather than in a document. A field can be deprecated for a version and then dropped — the format already allows for that. A reserved name gets no such courtesy: claim one at `v0.2` and anyone already using it as an ordinary name breaks without warning. Changing the layout is a migration for every bundle in existence.

### What wouldn't count

Reading without writing. Using a fraction of it. Elapsed time.

**Who decides:** the maintainer, weighing what consumers report. Nothing here trips on its own.

## Undecided — needs a decision before it can be specified

- **Field-level ratification** — confirm the working-default levels in `SPEC.md` §5.1 (`title`, `description`, `tags`, `verified`, `sources`).
- **Type-extension rules** (§10) — property-type vocabulary, `extends`/inheritance and conflict resolution, tool-default vs. bundle precedence, validator severities.
- **Type Definition `version`** — §12 refers to "a Type Definition's own `version`", but §10.1 never declares it. Decide whether a Type Definition carries one, whether it is semver, and what a bump means for copies already vendored elsewhere.
- **Vendored-type provenance** — §10.4 makes vendoring the only sharing mechanism, but a vendored `_types/*.md` records nothing about where it came from, so copies drift silently with no signal. Decide whether a vendored Type Definition SHOULD carry upstream provenance (`sources`, §7.3, alongside a version) so tooling can offer an opt-in staleness check without reintroducing remote resolution.
- **Link resolution** — the algorithm and slug rules (uniqueness scope within a bundle, ambiguity handling). Reintroduce `aliases` here; alternate-name resolution is meaningless without the resolution rules.
- **`extends: source` in §10.1** — the example Type Definition inherits from `source`, which is neither a reserved built-in (§10.4) nor defined anywhere in the spec. Either the built-ins list is incomplete, or the example is showing a bundle-local parent and should say so. Errata either way, but the two readings differ in what they commit the format to.
- **Asset links** — §8 specifies links only between Concepts; a bundle's non-Concept files (PDFs, images, attachments) have no link form at all, and `sources[].resource` (§7.3) covers attribution rather than body-prose linking. Candidate rule, needing no new syntax: `[[…]]` links Concepts, `[…](…)` links everything else.
- **Built-in types as files** — §10.3 calls the format self-hosting, but `concept` and `type_definition` exist only as prose tables (§5.1, §10.3). Decide whether the format ships them as real `_types/concept.md` and `_types/type_definition.md` — a normative reference rendering and a worked example of the format describing itself — and if so, whether a bundle vendors them or a tool supplies them.
- **Reserved-file formats** — the exact structure of `index.md` and `log.md` (§11).

## Deferred features — postponed, may return in a later version

- **`obligation: conditional`** — a field that is mandatory *only when* a stated condition holds, carrying a `when:` predicate (ISO 19115-style). Deferred from v0.0.1; `obligation` was chosen as the field-declaration key partly so this can be added later without a rename.
- **User-defined composite field types** — LKF ships the built-in `actor` and `actor_event` field types; arbitrary/user-defined nested object shapes are deferred.
- **Hierarchical slug/path field type** — a validated `field_type` for `/`-separated slug hierarchies (tags, categories, taxonomies). For now `tags` is `list of text`, kept loose; the type — and a good name (`path`/`breadcrumb`/`slug-path` all had drawbacks) — can come later.
- **Multiple inheritance for types** — v0.0.1 allows a single `extends` parent; multiple parents (with conflict-resolution rules) are deferred — wanted eventually, not needed now.
- **Domain-field override in inheritance** — v0.0.1 is add-only (a type may only *add* fields, never redefine inherited ones); letting a child override an inherited domain field is deferred, and may never be needed.
- **Stable identifiers** — opaque ids decoupled from path; additive when added (links stay name-based). See also the hidden-id link idea under [Ideas](#ideas--raised-for-consideration-not-evaluated-and-nothing-here-is-decided).
- **`aliases`** — ships together with link resolution (above).
- **`confidence`** and **`volatility`** — trust/freshness fields considered but held.
- **Concept-level `owner`** — accountability/stewardship, distinct from a source's `author`.
- **Attestation** — verifying that a computed value was produced by a sanctioned method (OKF's "Attested Computation"). Out of scope for MVP and adoption is uncertain, but worth revisiting in a later version.

## Ideas — raised for consideration, not evaluated, and nothing here is decided

- **A link that carries a hidden id** — a markdown link whose target is a path by default, but which also carries an identifier no reader sees, so a rename cannot break it. Recorded to be evaluated; neither adopted nor dismissed.

  A first look at possible carriers, as observations rather than a shortlist:
  - A **reference-link definition** (`[text][wp]` with `[wp]: /path "lkf:…"`) produces no rendered output at all under CommonMark, and its label binds the id to the link.
  - A **URL fragment** (`/path/file.pdf#lkf=…`) rides in the href and survives every renderer.
  - A **frontmatter link table** keyed by label keeps the id out of the prose entirely.
  - A **link title** is portable but visible on hover; an **adjacent HTML comment** is invisible but tied to the link only by adjacency.
  - Raw-HTML tags are stripped by sanitizers such as GitHub's; Pandoc-style `{#…}` attributes render literally outside the flavors that support them; zero-width Unicode breaks diffs and human readability.

  Evaluating it would have to settle: whether targets gain ids at all (an id on a link does nothing without one on the target — see **Stable identifiers**), how that sits with §3's requirement that block ids be human-readable rather than generated hashes, and whether the problem is pressing given that renames are already handled by tooling rewrites (§3) and unresolved links are legal (§8).
