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

**There is one version.** Until a second has to be readable alongside it, both
shapes are speculative and the folder is an empty level that costs a slug
collision for a companion file that does not exist.

## Re-open trigger

A second specification version must be readable alongside the current one,
**or** the first version-scoped companion file is proposed — a grammar, a
schema, or conformance tests.

**If it is the companion file that arrives first, take the folder shape and fix
slug resolution with it**, rather than shipping the collision and hoping the
roadmap catches up.
