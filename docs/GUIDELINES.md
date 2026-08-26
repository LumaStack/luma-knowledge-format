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

- **Work toward the next release accumulates on a branch named for it** — `v0.0.14-draft`. It is created when the first change toward that release starts and **deleted when it ships**. Between releases there may be no draft branch at all, which correctly says nothing is pending.
- **The branch name decides the version, at the start.** You cannot name it without answering *is this breaking?* — which is the question most often decided wrongly, and it is easier to answer while the change is fresh than at release time when you have stopped thinking about it. It also means **the version is decided once** rather than per pull request, so two changes in flight cannot both claim the next number.
- **Feature branches fork off the draft and merge back.** A small change may go straight to the draft.
- **Merge, never squash or rebase.** Every merge is a merge commit. This project requires the rationale for a change to live in its commit message, and squashing collapses a branch's messages into one — losing exactly what the rule exists to preserve. Rebasing rewrites them. Squash and rebase merging are disabled on the repository, so the buttons are absent rather than merely discouraged.
- **A fix that changes no meaning may go straight to `main`** — a typo, a broken link, anything under `docs/`. No version moves and no tag is cut. The invariant is that `main` is a coherent released specification, not that it is byte-identical to a tag.

> **`develop` is retired.** It was a long-lived second branch accumulating anything, and it rotted: thirty-one commits behind `main` and unused since `v0.0.10`. A branch named for the release it is building is finite, says what it is for, and disappears when it is done. `develop` said only *later*.

## Changelog

`CHANGELOG.md` lets a reader see **what changed between versions at a glance** — without diffing commits — and lets them skip minor edits that don't affect behavior.

- **Include** behavior-affecting spec changes: additions, renames/migrations, deprecations, and removals of field types, core fields, and type mechanics, plus other notable highlights.
- **Omit** anything that doesn't change behavior — wording, typos, formatting, example tweaks, and project-process changes.
- **As changes land,** add an entry under `## [Unreleased]` in the right category. [Keep a Changelog](https://keepachangelog.com) defines six, used in this order: **Added** (new features) / **Changed** (changes in existing functionality) / **Deprecated** (soon-to-be removed) / **Removed** (now removed) / **Fixed** (bug fixes) / **Security** (vulnerabilities). A **breaking** change carries an italic *Migration:* note saying exactly what a bundle or type author must do.
- **`Security` is never filed as `Fixed`.** It exists so a reader scanning for *must I upgrade urgently* finds the answer in one place, and burying a vulnerability among typo corrections defeats the only thing that group is for.
- **At release,** rename `## [Unreleased]` to `## [x.y.z] — YYYY-MM-DD` and open a fresh empty `## [Unreleased]` above it. Newest version on top.

## Cutting a release

**Releasing is merging the draft branch into `main` and tagging it.** The version is already correct everywhere, because the branch name decided it when the work started.

1. **Confirm the draft is what you mean to ship.** Clean, pushed, and its `CHANGELOG` section describes everything in it. A breaking change carries its italic *Migration:* note (see [Changelog](#changelog)).
2. **Date the changelog section.** `## [Unreleased]` becomes `## [x.y.z] — YYYY-MM-DD`, with a fresh empty `## [Unreleased]` above it.
   ⚠️ **Add the version's link definition at the foot of the file** and repoint `[Unreleased]` at the new tag — versions are meant to be linkable, and a heading in brackets with no definition renders as literal brackets:
   ```
   [Unreleased]: …/compare/vX.Y.Z...HEAD
   [X.Y.Z]:      …/compare/vPREV...vX.Y.Z
   ```
3. **Open a pull request** from the draft to `main`, and **merge it with a merge commit.**
4. **Tag `main`** with an annotated tag: `git tag -a vX.Y.Z -m "…"`. Lightweight tags carry no message and no author.
5. **Push the tag separately.** ⚠️ `git push origin main` does not push tags.
6. **Verify the invariant:** `main` and the newest tag point at the same commit. `git rev-parse --short main` and `git rev-parse --short vX.Y.Z^{commit}` must agree.
7. **Publish the GitHub Release** against the tag. ⚠️ A pushed tag is not a release — the tag is the mechanism, the Release is what people read, and the repository has no other release notes.
8. **Delete the draft branch.**

### A release is titled with its version and nothing else

`v0.0.13`. **Never** `v0.0.13 — some summary of the changes`.

The tag, the release commit and the GitHub Release all carry the same bare version. The changelog says what changed; a title that also says it is **a second summary, free to disagree with the first** — and it will, because one gets edited and the other does not. The place for *what this release contains* is the Release body and the changelog, both of which can hold the whole answer instead of a clause.

### What the GitHub Release body carries

- the changes, grouped **Added** / **Changed** / **Deprecated** / **Removed** / **Fixed** / **Security**, each with the reasoning rather than only the outcome;
- an **Upgrading from vX.Y.Z** section saying what a bundle or type author must actually do — and saying so plainly when the answer is *nothing*, which is usually the most useful sentence in the notes;
- the version-category note, if a breaking change shipped as a patch;
- a pointer to `CHANGELOG.md` for the full history.

The upgrade section is not a duplicate of the changelog's *Migration:* notes. Those are per-change and written as the change lands; this is the whole upgrade in one place, written once the release is known.

### The version lives in two files, and both have to

- `SPEC.md` — the `Version` header. The specification's version.
- `bundle/BUNDLE.md` — the `version` field. The version of the Bundle that ships the built-in Type Definitions.

**Neither is redundant.** The built-in types are a rendering of what the specification says, so a Bundle claiming a different version than the spec it renders is lying about which spec it implements — and a Bundle manifest has to carry a version whatever this project prefers, because that is what makes it pinnable. Continuous integration refuses a mismatch between them.

**`README.md` used to be a third, and was removed on 2026-08-26.** It stated the version in its Status line, which was the only one of the three a reader looked at rather than a tool parsed — and it went stale three times: it sat at `v0.0.9` through two releases, and then `v0.0.14` shipped and was tagged while it still said `v0.0.13`. Each time the response was a better check rather than one fewer place to forget.

**The lesson is the general one, so it is written as a rule rather than an anecdote: a number nothing parses is not worth duplicating.** The README now says *unstable* and points at `SPEC.md` and the tags. A reader who wants the current version gets it from the newest release, which cannot go stale because it is not written down anywhere.

**Resist a third.** A new file needing the version should read it from one of these, or go without.

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
- **Merge, never squash or rebase.** Squashing collapses the per-commit rationale this project requires be preserved.
- **Add a `CHANGELOG.md` entry** under `## [Unreleased]` for behavior-affecting changes; omit non-behavioral edits.
- **Keep changes additive within a minor version;** anything breaking requires a major bump and maintainer sign-off.
- **Honor the principles** in `PRINCIPLES.md`; if a change appears to violate one, stop and surface the conflict rather than proceeding.
- **Follow [Cutting a release](#cutting-a-release) in full** when preparing one. Two versions get bumped, not one, and three of the steps fail silently.

## Deferred (post-MVP)

Formal contribution guide (`CONTRIBUTING.md`), a `proposals/` RFC process, issue/PR templates, and `CODEOWNERS` — possibly added when necessary.
