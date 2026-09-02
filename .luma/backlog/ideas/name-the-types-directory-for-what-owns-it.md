---
type: luma/idea
title: Should `_types/` be named for the thing that reserves it?
created: { by: human:benlinton, at: 2026-08-25T00:00:00Z }
contributors: [human:benlinton, agent:claude-opus-5]
horizon: later
scope: project
stage: draft
---

# Should `_types/` be named for the thing that reserves it?

**Raised while settling the ALL CAPS rule for reserved *files*, and deliberately
not taken then.** `format_types/` was the shape suggested. Recorded because the
question is real and the answer is not obvious.

## What the underscore is doing

`_types/` uses a leading underscore to say *structural, not content* — a
convention borrowed from a dozen ecosystems where it means *internal, do not
treat as ordinary*. It works, and it is the reason directories were left out of
the casing rule: **a second signal for one meaning is worse than one.**

What it does **not** say is **who reserved it.** A reader meeting `_types/` in
somebody's bundle cannot tell whether the format claims that name or the bundle's
author invented it. `format_types/` would answer that, and would leave `types/`
free for anybody who wants an ordinary directory of that name.

## What it would cost

**A reserved name is taken from everyone, permanently**, and this trades one
reserved name for another rather than releasing anything. The cost is a rename
across every bundle that has a `_types/`, plus the tooling that resolves types,
plus §10 and §11 of the specification.

**And it may be solving a problem nobody has.** No bundle has yet wanted a
directory called `types` for something else. Until one does, the collision the
prefix guards against is hypothetical.

## The wider question it belongs to

**Does the format want a namespace for the names it claims, or a signal per
name?** `_types/` is a signal. `format_types/` is a namespace with one member.
If a second reserved directory ever appears the namespace pays off; if none
does, the prefix is noise on a single name.

Worth deciding *with* that second directory rather than before it.

## Re-open trigger

A second reserved directory is proposed, **or** a bundle author wants `types/`
for content and cannot have it.
