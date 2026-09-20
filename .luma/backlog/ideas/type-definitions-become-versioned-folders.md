---
type: luma/idea
title: Should a Type Definition be a versioned folder rather than a file?
created: { by: agent:claude-opus-5, at: 2026-09-19T00:00:00Z }
contributors: [human:luma-foundry, agent:claude-opus-5]
horizon: next
scope: project
stage: draft
---

# Should a Type Definition be a versioned folder rather than a file?

**A Type Definition is one file today. The proposal is that it becomes a folder
that carries its own history, its own migrations, and more than one version of
itself at once --- and that every document in the estate records which version
it was written against.**

## What was proposed

- **Rename `_types/` to `type_definitions/`.**
- **Each Type Definition is its own folder.**
- Inside it: **`index.md`** (or `type_definition.md`, or `definition.md`) ---
  and that file is **the latest version**.
- **`changelog.md`**, per Type Definition.
- **`migrations/`**, per Type Definition.
- **It should be trivial for one to become its own Bundle** if it wants to.
- **It is versioned**, so the version can drive migrations and so a document can
  record which one it follows.
- **Every document carries `type_version`** --- or `type_definition_version` ---
  **required**, so a document says which schema it conforms to and can therefore
  be migrated.
- **Older versions may sit beside the latest** --- `type-v1.0.md` next to
  `type-v1.1.md` --- so consumers can move at their own pace. **Only the latest
  is required; the rest are optional.**

---

*Everything above is the maintainer's. Below is the agent's, added while
capturing.*

## This is three questions wearing one coat

**They can be answered separately and probably should be, because they have
different blast radii and only one of them is cheap.**

**1. The directory name.** `_types/` becomes `type_definitions/`. Already
captured twice --- [[name-the-types-directory-for-what-owns-it]] here, which
asks whether the name should say *who reserved it*, and an idea in the engine
repository which maps the blast radius and concludes *"`_types/` is reserved by
the format, so this is an LKF spec change first."* **This proposal decides that
question by using the name, so the two should be reconciled rather than left to
disagree.**

**2. A Type Definition becomes a folder.** Structure, contained inside a Bundle,
and cheap by comparison --- nothing outside a Bundle has to know.

**3. Every document gains a required field.** **This is the expensive one.** It
touches every document in every Bundle in every adopted project, and it makes
documents written before it invalid rather than merely old. It should be costed
on its own rather than carried along by the other two.

## The cost that is already documented here

**[[one-folder-per-specification-version]] asks this same structural question
about the specification, and names the price: every version's document gets the
same slug.** `specification/0.0.19/lkf.md` and `specification/0.0.20/lkf.md` are
both `lkf`.

**A Type Definition folder has the identical problem and it lands harder**,
because a Type Definition is *addressed* --- documents resolve their type
through it. If `type_definitions/work_item/v1.0.md` and `.../v1.1.md` both slug
to something a reader has to disambiguate, then **the resolution rule becomes
part of the format** rather than a filing convention.

**That idea also names what the folder buys**, and the same argument transfers
intact: companion files that belong to one version and have nowhere
version-scoped to live. Here those are the migrations and the changelog, which
is exactly the proposal.

## What a required version field has to answer

**It is a different field from a provenance stamp, and conflating them is a
trap.** The violation type already carries `created_using` --- the bundle and
version that *created* a record --- with the rule that it is **written once and
never changed, including by a migration.** A `type_version` says what a document
*conforms to now*, which a migration is precisely supposed to change. **Two
fields, opposite rules, easy to merge by accident and hard to unpick
afterwards.**

Open, and each needs an answer before this could ship:

- **What does a document without the field mean?** Everything written to date
  lacks it. *Unversioned* and *version 1* are different claims, and the choice
  is permanent.
- **Does it name a version or a range?** A document conforming to both 1.0 and
  1.1 is the common case during a migration, and a single version cannot say so.
- **Who writes it --- the author or the tool?** If a tool, then hand-written
  documents are invalid until a tool touches them, which contradicts the format
  being hand-authorable.
- **What happens when the field disagrees with the content?** A document
  claiming 1.1 while missing a field 1.1 requires is a state that must be
  detectable, and *report never refuse* says it must still be readable.

## What "each could be its own Bundle" implies

**A Bundle has a `BUNDLE.md`, a version, a stage and a survival.** If a Type
Definition folder is to become one trivially, its layout should be a **subset**
of a Bundle's rather than a different shape --- `changelog.md` and a version are
already Bundle-shaped, which suggests the proposal is reaching for that
deliberately.

**Worth asking whether it is the same thing.** A Bundle that contains exactly
one Type Definition and nothing else may already be the answer, in which case
this is a naming and ergonomics question rather than a structural one.

## What this does not settle

- **Whether migrations belong to the Type Definition or to the engine.** The
  format can define where a migration *lives* without defining what runs it, and
  it must not learn about adoption --- that boundary is load-bearing.
- **Whether the changelog is prose or structured.** A migration that chains one
  version at a time needs machine-readable predecessors; a changelog for readers
  does not.
