---
type: luma/idea
title: Should every Document record which type version it was written against?
created: { by: agent:claude-fable-5, at: 2026-09-19T00:00:00Z }
contributors: [human:benlinton, agent:claude-fable-5]
horizon: later
scope: project
stage: provisional
---

# Should every Document record which type version it was written against?

**The surviving third of the versioned-folders proposal.** Its other two thirds
— `type_definitions/` as the directory's name, and every Type Definition
becoming a folder carrying its changelog, versions and migrations — ship in
`v0.0.21`. This
part was deliberately not carried along, because it is the expensive one: it
touches every Document in every Bundle in every adopted project, and as a
*required* field it would make Documents written before it incomplete rather
than merely old.

## What it would buy

A Document that names the type version it conforms to is a Document that can be
**migrated by query rather than by belief**. Today every change to a type is
absorbed by readers being tolerant — accept the old spelling beside the new
one, indefinitely — and a tolerance can only be retired by believing every
document everywhere has been rewritten, which nothing can check.

The engine repository's `frontmatter-needs-a-version-and-types-need-migrations`
worked the design through and its analysis stands: a **sibling field**
(`type_version`) beats an in-band spelling (`type: idea@1.0`) because old
readers ignore unknown keys for free, while in-band changes the one value every
tool dispatches on; and if in-band ever wins, `@` is the spelling, on YAML
safety and npm/Go/pip familiarity. The value would be **copied from the
`version` the Type Definition already declares**, so nothing new is minted.

## The name, and the two-versions corner (worked 2026-09-19)

**Self-hosting means a Type Definition will carry two version facts**: its own
`version` (the contract it publishes to its instances) and this field (the
version of the `type_definition` contract it was written against — because its
`type` is `type_definition`). The rule that keeps them apart is one the format
already follows without stating it: **bare `version` is what a document
publishes; a qualified `*_version` is what it conforms to.** `BUNDLE.md` is
exactly this shape today — `version` beside `lkf_version` — and Type
Definitions now match the publishing half.

**`type_version` beats `type_definition_version`** because of that corner, not
despite it. It qualifies the field `type`, which every document carries, so it
means the same thing everywhere — on a Type Definition it names the
`type_definition` contract version, with no special case. The longer spelling
is the one that invites the misreading: on a Type Definition, "the type
definition's version" is ambiguous between its own and the one it follows.
The self-hosting fixed point stays coherent: `type_definition`'s own file has
`defines: type_definition` and its `type_version` would reference itself, the
fixed point `defines` already has.

## What has to be answered before it could ship

- **What a Document without the field means.** Everything written to date lacks
  it, and *unversioned* and *version 1* are different claims. The choice is
  permanent.
- **A version or a range.** Conforming to both 1.0 and 1.1 is the common state
  during a migration, and a single value cannot say so.
- **Who writes it.** If a tool, hand-written Documents are second-class until a
  tool touches them, which contradicts the format being hand-authorable.
- **Field disagreeing with content.** A Document claiming 1.1 while missing
  what 1.1 requires must be detectable, and *report, never refuse* says it must
  still be readable.
- **It is not provenance.** A conformance claim is what a migration exists to
  change; a provenance stamp (the estate's `created_using`) is written once and
  never changed, including by a migration. Two fields, opposite rules — easy to
  merge by accident, hard to unpick.

## Why it is parked rather than rejected

The tolerance workaround is written down and functioning, so there is no
forcing event — and a version scheme chosen without one is the kind of
confidently-written rule that overreaches. The folder shape shipping first is
deliberate sequencing, not a substitute: it gives migrations a home so that
when a version field arrives it has something to key on.

## Re-open trigger

**A second version of any Type Definition ships with a migration beside it** —
the moment a Document's version determines what must be done to it. **Or** a
tolerance-retirement gate needs to be *checked* rather than believed — the
engine's `retire-the-migration-tolerances` names exactly this.
