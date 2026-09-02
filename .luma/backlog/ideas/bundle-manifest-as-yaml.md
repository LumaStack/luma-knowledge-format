---
type: luma/idea
title: Should bundle.md be YAML rather than markdown?
created: { by: human:benlinton, at: 2026-08-18T00:00:00Z }
contributors: [human:benlinton, agent:claude-opus-5]
horizon: later
scope: project
stage: draft
---

# Should bundle.md be YAML rather than markdown?

**Unsettled, and worth revisiting deliberately rather than by drift.**

The doubt is about files that are almost entirely frontmatter. A markdown
document with a manifest at the top is the format's best property when the body
carries something; it is a YAML file wearing a costume when the body does not.

What YAML would buy: a JSON Schema, which brings editor validation and
completion that frontmatter never gets; no ambiguity about whether the body is
normative; and one obvious parse. What markdown buys: one parser across every
document foreman reads, a `type` that makes the file discoverable by the same
tooling that reads bundles, and the self-describing property the whole format
rests on.

## Why not now

**The timing matters more than the answer.** `bundle.md` is a reserved file in
the specification and `bundle` is a built-in type, so moving it is a breaking
change — but the format is in the `0.0.z` tier it declares unstable, which is
exactly the window such a change exists for. That window closes at `1.0`, and
this is a cheap edit now and an expensive migration later. Same one-directional
cost curve as nesting the catalog content.

## What would settle it

Whether real bundles turn out to have bodies worth reading. A bundle is
knowledge-adjacent, and its body has a real job: what this bundle does, when to
reach for it, what it assumes. If bundles genuinely carry that, `bundle.md` earns
itself. If they do not, it is a manifest and the markdown wrapper is costing a
schema for nothing.

## Notes

Migrated from `luma-foreman/docs/IDEAS.md` on 2026-08-21. `created.at` is a
day-level estimate from git history.

**Split at migration** from the same entry as *should `catalog.md` be
`catalog.yaml`*, now filed in `luma-catalog`. The original treated them as one
question while observing that *"they may deserve different answers, and forcing
symmetry is part of what makes this feel wrong"* — which is the symmetry the
single entry was imposing. They are separated because they are independently
decidable and only one of them has a deadline: `bundle.md` is the format's,
`catalog.md` is defined nowhere in the specification and is luma-catalog's own
call.

**Checked at migration.** `SPEC.md` states version `v0.0.9`, *"Released. Pre-1.0
— the `0.0.z` tier is unstable; breaking changes may still ship until
`1.0.0`."* The window this idea depends on is open and closes where the entry
says.

**One nuance in favour of moving.** §11 lists `bundle.md` under *Reserved files*
but as **Recommended** — *"a Bundle SHOULD describe itself in a `bundle.md`"*. A
SHOULD is a slightly cheaper thing to change than a MUST.

**Early evidence leans towards keeping markdown.** The bundles read during this
migration — `git-secrets`, `backlog-ideas`, `session-manager`, `bundle-manager`,
`git-worktrees` — all have substantial bodies that a reader would want. That is
the entry's own test, and on current evidence `bundle.md` is passing it.
