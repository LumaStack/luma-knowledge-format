---
type: project
title: luma-knowledge-format
disclosure_level: public
description: The LKF specification itself — open it for frontmatter fields, the type system, conformance rules, or what a Bundle is. Not for anything written in the format.
owns:
  - the specification
  - built-in type definitions
  - conformance rules and the format's version
must_not_own:
  - tooling that reads or writes the format
  - knowledge written in the format
  - any organization's content or conventions
---

## Why it exists

Knowledge that has to cross teams, tools, systems and organizations needs a shape
people and agents can read without a tool in between. LKF is that shape: plain
markdown files with YAML frontmatter, one hard requirement (`type`), and a core
kept small enough to extend rather than fight.

**It is a specification, not an implementation.** Nothing here reads a document
or validates one — that is what a consumer does, and the format is worth having
precisely because several can.

## Boundaries

**The format never knows who is using it.** No luma-specific vocabulary, no
assumption that a consumer is `luma-foreman`, nothing that would make the format
worse for somebody who adopted it alone. When something is useful only to luma,
it belongs in a bundle rather than in the specification.

**Changes here are ratified before a tag.** `SPEC.md` is authoritative for the
version, and `docs/GUIDELINES.md` binds anybody working on it — branch from
`develop`, never commit to `main`, never squash or rebase.

## Status

`v0.0.9`, unstable. Breaking changes may ship in a patch until `v0.1.0`, which
would be a claim the format has been run for real in more than one place — and
has not earned yet.
