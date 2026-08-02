# LKF Guidelines

How the Luma Knowledge Format project is run. This document governs **anyone working on the format — human or agent alike.** It is written to be executable by an agent, not just read by a maintainer.

> **MVP draft — steer these:** two assumptions are baked in and open for change.
> 1. **Scope:** covers *how a change lands*, *versioning*, and *who decides*. Formal contribution/PR norms and a proposals process are deferred until there are outside contributors.
> 2. **Audience:** written as imperative rules an agent can follow, mirroring the format's own trust model (agents propose and draft; a human ratifies).

## Who decides

The project has a **human maintainer** with final say over the specification. Agents and contributors may propose and draft anything; ratification is the maintainer's.

## The core rule

**An agent MAY propose and draft a change to the specification. A human MUST ratify it before it is released (tagged).**

This is the format's own trust model applied to the project itself: drafting is machine work; sign-off is human. An unratified change is a `draft` proposal, not the spec.

## How a change lands

1. **Propose** — state the change and why. (An issue, or a short note; a `proposals/` process may come later.)
2. **Decide** — the maintainer accepts, revises, or rejects. The decision and its rationale are recorded in the commit message that lands the change.
3. **Edit** — apply the change to `SPEC.md` (and `PRINCIPLES.md` if affected).
4. **Version** — bump the `Version` in `SPEC.md`'s header per the versioning rules.
5. **Release** — tag the commit (`vX.Y.Z`). The tag is the published version.

## Versioning & release policy

- Scheme is **semver `major.minor.patch`**.
  - **patch** — clarifications, wording, errata; no grammar change.
  - **minor** — backward-compatible additions (new optional field, new reserved convention).
  - **major** — breaking changes.
- **Deprecate before removing.** A field or rule is marked `deprecated` (which validators warn on) for at least one minor version before a major removes it.
- Published versions are **git tags**; the spec at `HEAD` is the current version. No versioned filenames or directories.
- While in `v0.0.z`, breaking changes MAY ship in a patch — the pre-1.0 tier is explicitly unstable.

## Rules for agents working on the format

An agent working in this repository MUST:
- **Not change `SPEC.md` normatively without a ratified decision.** Drafting a proposal is fine; merging it into the spec as settled is not.
- **Record the rationale in the commit message** for every accepted change, so the "why" is never lost. (When commit-log spelunking gets painful, graduate to an append-only `DECISIONS.md`.)
- **Keep changes additive within a minor version;** anything breaking requires a major bump and maintainer sign-off.
- **Honor the principles** in `PRINCIPLES.md`; if a change appears to violate one, stop and surface the conflict rather than proceeding.
- **Bump the version and note the change** when preparing a release.

## Deferred (post-MVP)

Formal contribution guide (`CONTRIBUTING.md`), a `proposals/` RFC process, issue/PR templates, `CODEOWNERS`, and a `CHANGELOG.md` — possibly added when necessary.
