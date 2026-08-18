# LKF Roadmap

Open questions and deferred features for the Luma Knowledge Format. `SPEC.md` describes only what is settled; this file tracks what is not.

## Next steps

The three questions to answer next, in no particular order. Two are detailed
under *Undecided* below; the third is new.

1. **`extends: source`** — needs a ruling. §10.1's example Type Definition
   inherits from `source`, which is not a built-in and is defined nowhere.

   *Reading of the evidence, not yet a decision.* A third possibility beats the
   two originally recorded: the example is **vestigial**. `source` looks like a
   parent type from an earlier design that carried provenance — `author`,
   `last_modified`, where a thing came from — which is exactly what a
   `lab_result` would have wanted to inherit. Those fields are now **core**
   (§5.1 `created`, `modified`, `verified`, `sources`) and arrive through the
   root, so a `source` parent has nothing left to supply.

   Note also that §7.3's `sources` is a *field* — a list of `{id, resource,
   title, author, last_modified}` — not a type. Semantically a lab result is not
   a source: it is a measurement, and the thing it derives from is the source.
   So `extends: source` reads backwards even on its own terms.

   If that holds, the fix is to **delete `extends: source` from the example**
   rather than define a `source` type to justify it — the example would then
   show a type extending the root implicitly, which is the ordinary case and a
   better teaching example anyway. Creating the type to make the example work
   was declined deliberately: that is inventing specification to resolve errata.

   What would settle it: whether `source` was ever intended as a type, or only
   ever the `sources` field. Check the v0.0.1 history before deciding.
2. **Type Definition `version`** — §12 refers to "a Type Definition's own
   `version`", which §10.1 never declares. The `semver` field type now exists to
   hold it, so the remaining questions are whether a Type Definition carries one
   at all and what a bump means for copies already vendored elsewhere.
3. **Whether `concept` should carry fields of its own.** It currently declares
   none, so `type: concept` and `type: document` are structurally identical and
   differ only in what the name claims. That may be right — a type whose whole
   content is its name is worth having when the name is what a reader needs —
   but it has not been tested against a real knowledge base.

## What `v0.1.0` would mean

Not a promise that the format is finished, or safe to lean on. Only that it has stopped being a guess.

`v0.0.z` says the format has been reasoned about and barely used. `v0.1.0` says it has been run for real in more than one place and held up there. That is a smaller claim than it sounds, and it is the honest one — all of this was designed before it met any data.

Three things would have to be true.

### It has been exercised, more than once

Two or three separate codebases writing and reading real Documents — not examples, and not a corner chosen to avoid the awkward parts — reporting back that it worked for what they were doing.

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
- **`concept` fields** — the built-in `concept` extends `document` and adds
  nothing, making it structurally identical to the root and distinct only in
  meaning. Decide whether that is the intent or whether it should carry fields
  a knowledge-base entry always has.
- **Reserved-file formats** — the exact structure of `index.md` and `log.md` (§11).
- **How many names the format claims at a Bundle root.** §11 reserves four — `bundle.md`, `index.md`, `log.md`, `_types/` — and each one is a name no Bundle author may use for anything else. *The shape has stopped moving* above states the hazard exactly: a reserved name gets no deprecation courtesy, so claiming one later breaks anyone already using it as an ordinary name, without warning.

  The alternative is to claim **one** name and nest everything reserved beneath it:

  ```
  _lkf/
    types/
    index.md
    log.md
  bundle.md        ← arguably stays at root, since it names the thing itself
  ```

  One namespace to defend rather than four, and future reserved names cost nothing to add. It is the same move made twice elsewhere for the same reason — a catalog's content under one subtree, a project's store under `.hq/`.

  **What blocks it is the name.** `_lkf/` stamps the format's own initials into every Bundle's directory structure, which bets the format will always be the thing reading them; a format meant to outlive its origin should not announce whose idea it was in every path. No generic single word has survived: `_meta/` names a category rather than a job, `_reserved/` is honest and ugly. That is the open part.

  **Timing:** this is a Bundle-layout change, which the `v0.1.0` criteria above name specifically as something that must have stopped moving — and it is a migration for every Bundle in existence once any exist. Cheap now.

  *Settled in passing, recorded so it is not re-argued:* `_types/` keeps its name if consolidation does not happen. `.types/` was rejected because hidden directories are skipped by `ls`, default-ignored by search tools, and read as "tooling artifact" — all of which fight §10.6, where discovery is the entire point. `lkf-types/` and `luma-types/` were rejected on the vendor-name argument above. `_schema/` was rejected because *schema* means validate-or-reject, which is precisely what §10.5 refuses. The leading underscore stays because it has real prior art for framework-reserved directories, sorts ahead of letters, remains visible, and avoids colliding with the `types/` that TypeScript projects genuinely use.
- **Where `_types/` resolves.** §10.4 looks in exactly two places: the built-ins, and *a Bundle's* `_types/`. That ties type resolution to Bundles, and the first real consumer has already outgrown it — `luma-catalog` publishes a `type: catalog` document at the root of a directory that is deliberately not a Bundle (no version, never copied wholesale, and it contains Bundles), and that type needs somewhere to live.

  Working around it means either declaring the directory a Bundle, which makes Bundles-inside-Bundles a concept the format then owes an answer for, or letting the consumer invent its own lookup — which is how two tools end up disagreeing about where a type lives, and resolution fails quietly.

  The likely fix is to stop keying resolution on *Bundle* and key it on the directory root a Document is found under, whatever that root is. Decide whether `_types/` is Bundle-specific or root-specific, and if the latter, what constitutes a root.
- **Whether reserved manifests should be markdown at all.** §11.1 makes `bundle.md` a markdown Document carrying a frontmatter manifest. That is right when the body carries something a reader wants and it is a YAML file with a misleading extension when the body is empty — which is the state a pure manifest tends toward.

  Evidence from the first consumer, and it cuts both ways. A Bundle's body has real work to do: what this Bundle is, when to reach for it, what it assumes. `luma-catalog`'s equivalent manifest ended up a short instance note over frontmatter once its general prose moved into the Type Definition where it belonged. **The two may deserve different answers**, and assuming one format for every manifest is what makes that hard to see.

  A plain YAML file would buy a JSON Schema, and with it the editor validation and completion frontmatter never gets, plus no ambiguity about whether a body is normative. Markdown buys one parser for every file a consumer reads, a `type` that makes the file discoverable by the same tooling that reads Documents, and the self-describing property the format rests on.

  **Timing is the sharp part.** `bundle.md` is a reserved name and `bundle` is a built-in, so changing it is breaking — and `0.0.z` is precisely the window for that. It closes at `v0.1.0`, which *The shape has stopped moving* above defines partly as Bundle layout settling. Cheap now; a migration for every Bundle in existence later.

  What would settle it is observable rather than a matter of taste: whether real Bundles turn out to have bodies worth reading. Nobody has written enough of them to know.

## Deferred features — postponed, may return in a later version

- **`obligation: conditional`** — a field that is mandatory *only when* a stated condition holds, carrying a `when:` predicate (ISO 19115-style). Deferred from v0.0.1; `obligation` was chosen as the field-declaration key partly so this can be added later without a rename.
- **User-defined composite field types** — LKF ships the built-in `actor` and `actor_event` field types; arbitrary/user-defined nested object shapes are deferred.

  **No longer hypothetical.** The first consumer hit it immediately: `luma-catalog` declares a `type: catalog` whose `requires` is a list of five-key records (`bundle`, `obligation`, `version`, `by`, `tags`) and whose `starters` is a map of named lists of records. Neither is expressible, so both are declared with an obligation and a description and **no `field_type`** — legal, since §10.2 permits up to four keys and requires only `values` for enums, but it means the two fields carrying all the meaning are the two the contract says nothing about.

  Worth noting what the gap does and does not cost. Discovery still works: a reader finds the definition, sees the fields exist, and reads the shapes from the body prose. What is lost is machine checking of exactly the parts most likely to be got wrong. That is a tolerable trade for one consumer and a poor one at ten.

  Three directions, none evaluated: a nested `fields` block; `list of <type_name>` referencing another Type Definition, which would reuse the mechanism already there; or deciding deliberately that frontmatter stays flat and structured payloads belong in the body. **The third is defensible and would be worth stating outright** rather than leaving as an absence a consumer discovers by hitting it.
- **Hierarchical slug/path field type** — a validated `field_type` for `/`-separated slug hierarchies (tags, categories, taxonomies). For now `tags` is `list of text`, kept loose; the type — and a good name (`path`/`breadcrumb`/`slug-path` all had drawbacks) — can come later.
- **Multiple inheritance for types** — v0.0.1 allows a single `extends` parent; multiple parents (with conflict-resolution rules) are deferred — wanted eventually, not needed now.
- **Domain-field override in inheritance** — v0.0.1 is add-only (a type may only *add* fields, never redefine inherited ones); letting a child override an inherited domain field is deferred, and may never be needed.
- **Stable identifiers** — opaque ids decoupled from path; additive when added (links stay name-based). See also the hidden-id link idea under [Ideas](#ideas--raised-for-consideration-not-evaluated-and-nothing-here-is-decided).
- **`aliases`** — ships together with link resolution (above).
- **`confidence`** and **`volatility`** — trust/freshness fields considered but held.
- **Document-level `owner`** — accountability/stewardship, distinct from a source's `author`.
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
