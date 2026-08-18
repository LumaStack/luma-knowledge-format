# LKF Guidelines

How the Luma Knowledge Format project is run. This document governs **anyone working on the format — human or agent alike.** It is written to be executable by an agent, not just read by a maintainer.

> **MVP draft — steer these:** two assumptions are baked in and open for change.
> 1. **Scope:** covers *how a change lands*, *branching*, *the changelog*, *versioning*, and *who decides*. Formal contribution/PR norms and a proposals process are deferred until there are outside contributors.
> 2. **Audience:** written as imperative rules an agent can follow, mirroring the format's own trust model (agents propose and draft; a human ratifies).

## Who decides

The project has a **human maintainer** with final say over the specification. Agents and contributors may propose and draft anything; ratification is the maintainer's.

## The core rule

**An agent MAY propose and draft a change to the specification. A human MUST ratify it before it is released (tagged).**

This is the format's own trust model applied to the project itself: drafting is machine work; sign-off is human. An unratified change is a `draft` proposal, not the spec.

## How a change lands

1. **Propose** — state the change and why. (An issue, or a short note; a `proposals/` process may come later.)
2. **Decide** — the maintainer accepts, revises, or rejects. The decision and its rationale are recorded in the commit message that lands the change.
3. **Branch** — make the change on a branch off `main`, never on `main` directly (see [Branching](#branching)).
4. **Edit** — apply it to `SPEC.md` (and `PRINCIPLES.md` if a principle is affected), and add a `CHANGELOG.md` entry under `## [Unreleased]` for any behavior-affecting change (see [Changelog](#changelog)).
5. **Release** — see [Cutting a release](#cutting-a-release). It is a checklist rather than a sentence because several of its steps are silent when missed.

## Branching

`main` always reflects the **latest released version** — it advances only when a release is cut, so `main` and the newest tag stay in lockstep. Checking out `main` always gives a coherent release, never half-finished work.

- **All changes are made on a branch off `main`**, never committed directly to `main`. Unreleased work accumulates on a **`develop`** branch (feature branches may fork off it and merge back).
- **`main` advances only at release time**, by merging the release-ready work in and tagging it.
- **Exception — critical hotfix:** a fix that genuinely can't wait for the next release may go straight to `main` and ship as a patch; `develop` then picks it up.

## Changelog

`CHANGELOG.md` lets a reader see **what changed between versions at a glance** — without diffing commits — and lets them skip minor edits that don't affect behavior.

- **Include** behavior-affecting spec changes: additions, renames/migrations, deprecations, and removals of field types, core fields, and type mechanics, plus other notable highlights.
- **Omit** anything that doesn't change behavior — wording, typos, formatting, example tweaks, and project-process changes.
- **As changes land,** add an entry under `## [Unreleased]` in the right category (**Added** / **Changed** / **Deprecated** / **Removed**). A **breaking** change carries an italic *Migration:* note saying exactly what a bundle or type author must do.
- **At release,** rename `## [Unreleased]` to `## [x.y.z] — YYYY-MM-DD` and open a fresh empty `## [Unreleased]` above it. Newest version on top.

## Cutting a release

A release is the only thing that advances `main`. Work through this in order; the
steps that get skipped are marked.

1. **Confirm `develop` is what you mean to ship.** It is clean, pushed, and
   `## [Unreleased]` in `CHANGELOG.md` describes everything in it. A breaking
   change carries its italic *Migration:* note (see [Changelog](#changelog)).
2. **Decide the version**, by the categories below. If a breaking change is
   shipping as a patch under the pre-1.0 clause, say so in the release commit —
   it otherwise reads as a miscategorisation later.
3. **Open a pull request** from `develop` to `main`.
4. **Merge it with a merge commit — never a squash.** ⚠️ The rationale for each
   change lives in its own commit message, and this project requires that
   rationale be preserved. Squashing destroys it.
5. **Bump both versions.** ⚠️ Two files, and the second is the easy miss:
   - `SPEC.md` — the `Version` header. The specification's version.
   - `bundle/bundle.md` — the `version` field. The version of the Bundle that
     ships the built-in Type Definitions.

   They are kept in lockstep deliberately: the built-in types are a rendering of
   what the specification says, so a Bundle claiming a different version than the
   spec it renders would be lying about which spec it implements.
   `bundle/bundle.md` did not exist before `v0.0.4`, which is why releases up to
   `v0.0.3` touched `SPEC.md` alone.
6. **Promote the changelog.** Rename `## [Unreleased]` to `## [x.y.z] — YYYY-MM-DD`
   and open a fresh empty `## [Unreleased]` above it. Newest version on top.
7. **Commit as `Release vX.Y.Z`**, with the rationale — what it contains, and why
   that version number.
8. **Tag `main`** with an annotated tag: `git tag -a vX.Y.Z -m "…"`. Lightweight
   tags carry no message and no author.
9. **Push `main` and the tag.** ⚠️ Two pushes. `git push origin main` does not
   push tags; `git push origin vX.Y.Z` is a separate command.
10. **Fast-forward `develop` to `main`.** ⚠️ Easily forgotten, and the next cycle
    silently starts from stale state — the following release PR would show the
    previous release commit as a change.
11. **Verify the invariant:** `main` and the newest tag point at the same commit.
    `git rev-parse --short main` and `git rev-parse --short vX.Y.Z^{commit}`
    must agree.

## Versioning & release policy

- Scheme is **semver `major.minor.patch`**.
  - **patch** — clarifications, wording, errata; no grammar change.
  - **minor** — backward-compatible additions (new optional field, new reserved convention).
  - **major** — breaking changes.
- **Deprecate before removing.** A field or rule is marked `deprecated` (which validators warn on) for at least one minor version before a major removes it.
- Published versions are **git tags**; `main` equals the newest tag (the released spec). No versioned filenames or directories.
- While in `v0.0.z`, breaking changes MAY ship in a patch — the pre-1.0 tier is explicitly unstable.

## Rules for agents working on the format

An agent working in this repository MUST:
- **Not change `SPEC.md` normatively without a ratified decision.** Drafting a proposal is fine; merging it into the spec as settled is not.
- **Record the rationale in the commit message** for every accepted change, so the "why" is never lost. (When commit-log spelunking gets painful, graduate to an append-only `DECISIONS.md`.)
- **Work on a branch off `main`, never commit directly to `main`** (except a ratified critical hotfix) — `main` stays equal to the latest release.
- **Add a `CHANGELOG.md` entry** under `## [Unreleased]` for behavior-affecting changes; omit non-behavioral edits.
- **Keep changes additive within a minor version;** anything breaking requires a major bump and maintainer sign-off.
- **Honor the principles** in `PRINCIPLES.md`; if a change appears to violate one, stop and surface the conflict rather than proceeding.
- **Follow [Cutting a release](#cutting-a-release) in full** when preparing one. Two versions get bumped, not one, and three of the steps fail silently.

## Deferred (post-MVP)

Formal contribution guide (`CONTRIBUTING.md`), a `proposals/` RFC process, issue/PR templates, and `CODEOWNERS` — possibly added when necessary.
