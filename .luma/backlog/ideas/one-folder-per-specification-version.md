---
type: luma/idea
title: Should each specification version get its own folder?
created: { by: human:benlinton, at: 2026-08-30T00:00:00Z }
contributors: [human:benlinton, agent:claude-opus-5]
horizon: later
scope: project
lifecycle: draft
---

# Should each specification version get its own folder?

**Raised while moving the specification into the Bundle, and deliberately not
taken then.** `specification/` today holds one file, `lkf.md`, with no version
in its name. The question is what happens when a second version has to be
readable at the same time.

## The two shapes

```
specification/lkf-0.0.19.md          # flat: version in the filename
specification/0.0.19/lkf.md          # folder: version in the path
```

## What the folder buys

**Companion files that belong to one version.** A machine-readable grammar, a
JSON Schema for frontmatter, conformance examples, version-scoped Assets — each
belongs to a specific version and needs somewhere version-scoped to live. Flat
file-per-version has no such place. This is the only real reason; the others
(atomic `git rm -r` of a version) are conveniences.

## What the folder costs

**Every version's specification gets the same slug.** `specification/0.0.19/lkf.md`
and `specification/0.0.20/lkf.md` are both `lkf`. How a bare slug resolves when
two Documents share one is **not specified** — it is in `roadmap.md` under
link resolution. Taking the folder shape now would make this repository's own
Bundle the first thing depending on a rule the format has not written.

Flat filenames keep every slug unique and ask nothing of the roadmap.

## Why neither was chosen

**The condition that would force this has not arrived.** Multi-versioning
becomes real when **adopters depend on different specification versions at
once** — somebody pinned to an older release while somebody else tracks the
newest, and both need the text they are reading to be present. Nobody does yet,
and the maintainer's position on 2026-08-30 is that this is not expected in the
foreseeable future.

**This note is not a plan.** It exists so the two shapes and their costs are on
record if that day comes, and so nobody re-derives them under time pressure.

**There is one version.** Until a second has to be readable alongside it, both
shapes are speculative and the folder is an empty level that costs a slug
collision for a companion file that does not exist.

## Re-open trigger

**Adopters depend on different specification versions at once.** That is the
condition, and everything above is how to answer it when it arrives.

**Or** the first version-scoped companion file is proposed — a grammar, a
schema, conformance tests. **If that arrives first, take the folder shape and
fix slug resolution with it**, rather than shipping the collision and hoping the
roadmap catches up.
