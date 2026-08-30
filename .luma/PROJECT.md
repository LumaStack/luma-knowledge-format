---
type: luma/project
title: luma-knowledge-format
disclosure_level: public
description: The LKF specification itself — open it for frontmatter fields, the type system, conformance rules, or what a Bundle is. Not for anything written in the format.
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

**Changes here are ratified before a tag.** `luma-knowledge-format/specification/lkf.md`
is authoritative for the version, and `docs/guidelines.md` binds anybody working on it — read it in full
before cutting a release. Work accumulates on a branch named for the release it
is building; never commit to `main`, never squash or rebase.
