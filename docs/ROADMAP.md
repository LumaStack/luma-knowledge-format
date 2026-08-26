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
3. **~~Whether `concept` should carry fields of its own.~~** Resolved by removing
   the type — see the `0.0.10` entry in `CHANGELOG.md`. The observation that
   made it a question (`type: concept` and `type: document` were structurally
   identical) turned out to be the answer.

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
- **Type Definition `version` — declared in `0.0.11`, semantics still open.** §10.1 now permits one and §12's dangling reference has something to point at. **What a bump *means* is deliberately undefined**: it is a label, not a promise, so a consumer compares for equality and infers nothing from the tier. Still to settle — whether the tiers in `bundle-versioning` carry over to types, and whether bumping a type forces a bump of the Bundle shipping it.

  **This was the pressure behind wanting one repository per shared type**, and splitting would never have fixed it — a repository has one version too. Declaring the version on the type is what removes it: a consumer that vendored one type out of six no longer sees a bump caused by the other five.
- **~~Should `lifecycle_status` carry `unknown`, as its default?~~ Yes.** Shipped in `0.0.11`. `unknown` is not a stage — it says the value was not filled in — and it is the default because both real defaults would be wrong guesses.
- **~~`policy` and `preload` answer overlapping questions.~~ Resolved in `0.0.11`.** They were on the same axis: all three engagement modes were written in loading vocabulary, so two collided with a loading field. Redefined by what a consumer *does* — run it, be bound by it, read it — they are orthogonal, and a `policy` with `preload: optional` stops reading as a contradiction.

  **What is not solved is reachability.** A rule nobody loads still governs nothing. The answer is something always present naming the rules that exist, and nothing does that yet.

- **Vendored-type provenance** — §10.4 makes vendoring the only sharing mechanism, but a vendored `_types/*.md` records nothing about where it came from, so copies drift silently with no signal. Decide whether a vendored Type Definition SHOULD carry upstream provenance (`sources`, §7.3, alongside a version) so tooling can offer an opt-in staleness check without reintroducing remote resolution.

  **This now has a real consumer and is the highest-value item here.** A shared type library — the thing §10.4 already contemplates when it says types are shared *by vendoring* — is only safe if drift is loud. Without provenance, the choice for a widely-used type is between an unprefixed built-in the format did not want and copies that disagree silently. **Provenance is what makes the namespaced-and-vendored path viable**, and it is what keeps the built-in list short.
- **Link resolution** — the algorithm and slug rules (uniqueness scope within a bundle, ambiguity handling). Reintroduce `aliases` here; alternate-name resolution is meaningless without the resolution rules.
- **`extends: source` in §10.1** — the example Type Definition inherits from `source`, which is neither a reserved built-in (§10.4) nor defined anywhere in the spec. Either the built-ins list is incomplete, or the example is showing a bundle-local parent and should say so. Errata either way, but the two readings differ in what they commit the format to.
- **~~Whether `concept` survives.~~ It does not.** Removed in `0.0.10`.

  The argument that had held it here was that removing a name and re-adding it
  later collides with every Bundle that defined it privately in between. **That
  cost is real and was accepted**, because the deferral had started costing more:
  a type marked *under review* still gets adopted, and five Documents across two
  published Bundles had declared it, none of them for a reason a `document` could
  not serve. **A name that is noise is not harmless once people begin using it.**

  Its retrieval mode did not need rescuing. *Retrieved when relevant* is what a
  plain `document` already is, so the mode survives the type — it simply lives on
  the root, where it always was.

  *Re-open only if a durable knowledge base turns out to need fields a `document`
  cannot give it, which is what would have justified the type in the first place.*
- **Reserved-file formats** — the exact structure of `index.md` and `LOG.md` (§11).
- **How many names the format claims at a Bundle root.** §11 reserves four — `BUNDLE.md`, `index.md`, `LOG.md`, `_types/` — and each one is a name no Bundle author may use for anything else. *The shape has stopped moving* above states the hazard exactly: a reserved name gets no deprecation courtesy, so claiming one later breaks anyone already using it as an ordinary name, without warning.

  The alternative is to claim **one** name and nest everything reserved beneath it:

  ```
  _lkf/
    types/
    index.md
    log.md
  bundle.md        ← arguably stays at root, since it names the thing itself
  ```

  One namespace to defend rather than four, and future reserved names cost nothing to add. It is the same move made twice elsewhere for the same reason — a catalog's content under one subtree, a project's store under `.luma/`.

  **What blocks it is the name.** `_lkf/` stamps the format's own initials into every Bundle's directory structure, which bets the format will always be the thing reading them; a format meant to outlive its origin should not announce whose idea it was in every path. No generic single word has survived: `_meta/` names a category rather than a job, `_reserved/` is honest and ugly. That is the open part.

  **Timing:** this is a Bundle-layout change, which the `v0.1.0` criteria above name specifically as something that must have stopped moving — and it is a migration for every Bundle in existence once any exist. Cheap now.

  *Settled in passing, recorded so it is not re-argued:* `_types/` keeps its name if consolidation does not happen. `.types/` was rejected because hidden directories are skipped by `ls`, default-ignored by search tools, and read as "tooling artifact" — all of which fight §10.6, where discovery is the entire point. `lkf-types/` and `luma-types/` were rejected on the vendor-name argument above. `_schema/` was rejected because *schema* means validate-or-reject, which is precisely what §10.5 refuses. The leading underscore stays because it has real prior art for framework-reserved directories, sorts ahead of letters, remains visible, and avoids colliding with the `types/` that TypeScript projects genuinely use.
- **Where `_types/` resolves.** §10.4 looks in exactly two places: the built-ins, and *a Bundle's* `_types/`. That ties type resolution to Bundles, and the first real consumer has already outgrown it — `luma-catalog` publishes a `type: catalog` document at the root of a directory that is deliberately not a Bundle (no version, never copied wholesale, and it contains Bundles), and that type needs somewhere to live.

  Working around it means either declaring the directory a Bundle, which makes Bundles-inside-Bundles a concept the format then owes an answer for, or letting the consumer invent its own lookup — which is how two tools end up disagreeing about where a type lives, and resolution fails quietly.

  The likely fix is to stop keying resolution on *Bundle* and key it on the directory root a Document is found under, whatever that root is. Decide whether `_types/` is Bundle-specific or root-specific, and if the latter, what constitutes a root.
- **Whether reserved manifests should be markdown at all.** §11.1 makes `BUNDLE.md` a markdown Document carrying a frontmatter manifest. That is right when the body carries something a reader wants and it is a YAML file with a misleading extension when the body is empty — which is the state a pure manifest tends toward.

  Evidence from the first consumer, and it cuts both ways. A Bundle's body has real work to do: what this Bundle is, when to reach for it, what it assumes. `luma-catalog`'s equivalent manifest ended up a short instance note over frontmatter once its general prose moved into the Type Definition where it belonged. **The two may deserve different answers**, and assuming one format for every manifest is what makes that hard to see.

  A plain YAML file would buy a JSON Schema, and with it the editor validation and completion frontmatter never gets, plus no ambiguity about whether a body is normative. Markdown buys one parser for every file a consumer reads, a `type` that makes the file discoverable by the same tooling that reads Documents, and the self-describing property the format rests on.

  **Timing is the sharp part.** `BUNDLE.md` is a reserved name and `bundle` is a built-in, so changing it is breaking — and `0.0.z` is precisely the window for that. It closes at `v0.1.0`, which *The shape has stopped moving* above defines partly as Bundle layout settling. Cheap now; a migration for every Bundle in existence later.

  What would settle it is observable rather than a matter of taste: whether real Bundles turn out to have bodies worth reading. Nobody has written enough of them to know.

## Deferred features — postponed, may return in a later version

- **`obligation: conditional`** — a field that is mandatory *only when* a stated condition holds, carrying a `when:` predicate (ISO 19115-style). Deferred from v0.0.1; `field_presence` was chosen as the field-declaration key partly so this can be added later without a rename.
- **User-defined composite field types** — LKF ships the built-in `actor` and `actor_event` field types; arbitrary/user-defined nested object shapes are deferred.

  **No longer hypothetical.** The first consumer hit it immediately: `luma-catalog` declares a `type: catalog` whose `requires` is a list of five-key records (`bundle`, `field_presence`, `version`, `by`, `tags`) and whose `starters` is a map of named lists of records. Neither is expressible, so both are declared with an obligation and a description and **no `field_type`** — legal, since §10.2 permits up to four keys and requires only `values` for enums, but it means the two fields carrying all the meaning are the two the contract says nothing about.

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

- **A strict mode.** Raised as: *by design and by default LKF is open and accepting, but in cases where allowing faults is not acceptable, there should be a strict mode that gets enabled, where everything needs to be perfect or it fails.* **This does not break the specification or go against it** — it gives people a way to use most of the format while letting producers and consumers know that **this type is not open. It is exact, it fails, it throws errors, and corruption is not tolerated.**

  Whether strictness is set *on* a type, or whether you get it by declaring a different type that is always strict, is open. *"I'm assuming the new-type route is best, but not sure."*

  **The seam it would use already exists, and §4 is not in the way.** §4's `MUST NOT reject` governs **conformance** — whether a file *is* a Document. It says nothing about whether a tool will *act* on one. The specification already relies on that gap: `preload: mandatory` says a consumer that cannot load the Document **refuses rather than proceeding**, which is a consumer failing without rejecting anything as non-conformant. **A strict mode is that same move, generalised** — the file stays a valid Document and the work stops.

  **It may need no new vocabulary.** §5 says obligation *"describes intent. Whether and how a tool checks it is a suggested validation framework, not a rule."* On that reading **strict mode is simply the switch that turns published intent into an enforced contract** — the contract is already fully specified in the Type Definition and merely unbinding. If that is all it is, nothing new is declared and the question becomes who throws the switch.

  **Where it should live is the real question, and there is precedent cutting both ways.** Against putting it on the artifact: obligation is deliberately *not* a property of a bundle, because *"the same bundle is mandatory at one organization and merely available everywhere else — the publisher declares it; the artifact does not carry it."* The same argument says a `decision` type might warrant strictness at a bank and not at a startup. For putting it on the artifact: the idea explicitly wants producers to be able to *signal* exactness, which a consumer-side setting cannot do.

  **The shape that satisfies both is probably the `preload: mandatory` one** — the author declares the claim, the consumer decides whether to honour it, and what is refused is the work rather than the document.

  **The always-strict-type route has one property the flag does not:** strictness travels with the name and cannot be quietly turned off. That is also its cost — no deployment can relax it, and the type vocabulary doubles if every strict thing needs a strict twin. Whether inheritance could carry it instead (`extends` some strict root) runs into single inheritance and into using a field mechanism for a non-field property.

  Evaluating it would have to settle: **what counts as a failure** — unknown keys, unresolved links, a missing `recommended`, a value outside an enum are four very different bars, and §4 currently forgives all of them; whether strictness is all-or-nothing or per-check; and **what a strict consumer does with a non-strict type**, and vice versa, since a Bundle will hold both.

  ### The corruption half may be a second feature, and a smaller one

  *"Corruption is not tolerated"* is not validation. **Validation asks whether a Document satisfies its contract; integrity asks whether it is the file it was.** A perfectly valid Document can be corrupt — somebody rewrote a settled decision's reasoning — and a pristine one can be invalid. The format also already has a third thing easily confused with both: `verified` and `stale_after` are integrity of *claims*, not of bytes.

  **Most of it is already answered, which shrinks the problem to one gap.** `adopted.toml` checksums answer *altered since adoption*; git answers *altered since authoring*, free, for anything committed; `vendored_from` answers *matches what upstream published*. **What none of them answers is which changes are illegitimate** — git reports that a file changed and has no idea it was never supposed to.

  **The obvious design is ruled out by an argument already made here.** An in-document `content_hash` is maintained by whoever edits the file, and the person it guards against is exactly that person — the same reason `adopted.toml`'s checksum *"lives nowhere near a file you are invited to edit"*. It is also self-referential and stale after every legitimate change.

  **So the shape worth considering is a declaration rather than a digest**, because the format already makes immutability claims and enforces none of them: §11 says a `LOG.md` writer **MUST append rather than rewrite**, and a `stable` decision is *frozen*. Rules, stated, unchecked.

  ```yaml
  type: type_definition
  defines: audit_record
  mutability: frozen          # append_only | frozen | open (default)
  ```

  **It detects nothing by itself, and that is the point.** It states the rule so a validator, a review or git can check it, and so a reader knows an edit here is a defect rather than maintenance. No self-reference, no tool required to maintain it, and it makes existing unenforced prose declarable. **It also composes with strict mode rather than duplicating it:** the type says what may change, strict mode says whether a violation stops the work.

  **The heavier alternative** is a sidecar digest — an `adopted.toml` for authored content. It genuinely detects corruption, and it needs a tool to maintain, duplicates git for anything committed, and puts a must-not-be-hand-edited file among files people hand-edit.

  **The honest third option is nothing.** Git plus `adopted.toml` already cover every case anyone has hit, and *corruption is not tolerated* may be a strict-mode posture rather than a mechanism.

  *Prior art worth lifting rather than reinventing, if integrity ever does become a format concern: a consumer of this format has already built the layer — algorithm-tagged content hashes so the algorithm can change without breaking the format, a cheap size-and-mtime pre-check before hashing, and a vocabulary distinguishing output that is thin from output that is false. **That it was built downstream without the format's help is also an argument the format does not need it.***

- **A link that carries a hidden id** — a markdown link whose target is a path by default, but which also carries an identifier no reader sees, so a rename cannot break it. Recorded to be evaluated; neither adopted nor dismissed.

  A first look at possible carriers, as observations rather than a shortlist:
  - A **reference-link definition** (`[text][wp]` with `[wp]: /path "lkf:…"`) produces no rendered output at all under CommonMark, and its label binds the id to the link.
  - A **URL fragment** (`/path/file.pdf#lkf=…`) rides in the href and survives every renderer.
  - A **frontmatter link table** keyed by label keeps the id out of the prose entirely.
  - A **link title** is portable but visible on hover; an **adjacent HTML comment** is invisible but tied to the link only by adjacency.
  - Raw-HTML tags are stripped by sanitizers such as GitHub's; Pandoc-style `{#…}` attributes render literally outside the flavors that support them; zero-width Unicode breaks diffs and human readability.

  Evaluating it would have to settle: whether targets gain ids at all (an id on a link does nothing without one on the target — see **Stable identifiers**), how that sits with §3's requirement that block ids be human-readable rather than generated hashes, and whether the problem is pressing given that renames are already handled by tooling rewrites (§3) and unresolved links are legal (§8).
