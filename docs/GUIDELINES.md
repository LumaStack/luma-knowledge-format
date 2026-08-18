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

`main` always reflects the **latest released specification**. Checking out `main` gives a coherent release, never half-finished normative work.

- **All changes are made on a branch off `main`**, never committed directly to `main`. Unreleased work accumulates on a **`develop`** branch (feature branches may fork off it and merge back).
- **Normative work reaches `main` only at release time**, by merging the release-ready work in and tagging it.
- **Merge, never squash or rebase.** Every merge is a merge commit. This project requires the rationale for a change to live in its commit message, and squashing collapses a branch's messages into one — losing exactly what the rule exists to preserve. Rebasing rewrites them. Squash and rebase merging are disabled on the repository, so the buttons are absent rather than merely discouraged.
- **Exception — critical hotfix:** a fix that genuinely can't wait for the next release may go straight to `main` and ship as a patch; `develop` then picks it up.

### Non-normative documentation may land on `main` between releases

**The test: can this change make a reader wrong about what the format requires?** If not, it does not need a release.

| Gated — release only | May land between releases |
|---|---|
| `SPEC.md` | `docs/EXPLANATION.md` |
| `PRINCIPLES.md` | `docs/ROADMAP.md` |
| `CHANGELOG.md` | `docs/GUIDELINES.md` |
| `bundle/**` | `README.md`, except the version in its Status line |
| the version in `README.md`'s Status line | |

`PRINCIPLES.md` is gated despite being prose: the specification bends to honor it, so a principle on `main` that the released spec does not yet reflect describes a format nobody can use. `bundle/**` is gated because the built-in types are a rendering of what the specification says, and the two must agree at every commit on `main`.

**Why this exception exists.** The roadmap is the file that says what is open, the explanation is the file a newcomer reads first, and GitHub shows `main`. Gating them on the release cycle guarantees the two most-read documents in the repository are the two most likely to be stale — and neither can mislead anyone about a rule, because neither states one.

**The rules are unchanged otherwise.** Still a branch and a pull request, never a direct commit to `main`. Still a merge commit. **And `develop` must be fast-forwarded to `main` immediately afterward** — the same trap as step 10 of a release, reached by a different route: skip it and the next release pull request shows the documentation as a change.

**The invariant, restated precisely.** `main` and the newest tag are in lockstep **at the moment a release is cut**, and between releases `main` may be ahead by non-normative documentation only. That is checkable rather than a matter of trust:

```sh
git diff --name-only "$(git describe --tags --abbrev=0)"..main
```

Anything from the gated column in that output is unreleased normative work sitting on `main`, and is a mistake to fix rather than a state to leave.

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
4. **Merge it with a merge commit.** Squash and rebase merging are disabled on
   the repository (see [Branching](#branching)), so this should be the only
   option offered.
5. **Bump the version in all three places.** ⚠️ Three files. Only the first is
   obvious, and a release that misses one is not detectable by reading any single
   file:
   - `SPEC.md` — the `Version` header. The specification's version.
   - `bundle/bundle.md` — the `version` field. The version of the Bundle that
     ships the built-in Type Definitions.
   - `README.md` — the version in the **Status** line.

   Then verify rather than trust, because all three are one-line edits that look
   done at a glance:

   ```sh
   grep -n '^- \*\*Version' SPEC.md
   grep -n '^version:' bundle/bundle.md
   grep -n '^> \*\*Status' README.md
   ```

   They are kept in lockstep deliberately, and each states something different
   that a stale number makes false. The built-in types are a rendering of what
   the specification says, so a Bundle claiming a different version than the spec
   it renders is lying about which spec it implements. The README's Status line
   is what a reader sees before deciding whether to adopt an unstable format at
   all, so a stale one understates how much has moved.

   **Resist a fourth.** Three is already more than the design wants — each is a
   place to forget. A new file needing the version should read it from one of
   these or go without; a number that exists to be looked at by a human is worth
   the duplication, and one that exists to be parsed is not.

   *History, so an old release does not read as a missed step:*
   `bundle/bundle.md` did not exist before `v0.0.4`, and the README carried no
   version before `v0.0.7`. Releases up to `v0.0.3` touched `SPEC.md` alone.
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
12. **Publish the GitHub Release** against the tag. ⚠️ A pushed tag is not a
    release — the tag is the mechanism, the Release is what people read, and the
    repository has no other release notes. It carries:
    - the changes, grouped **Added** / **Changed** / **Deprecated** / **Removed**,
      each with the reasoning rather than only the outcome;
    - an **Upgrading from vX.Y.Z** section saying what a bundle or type author
      must actually do — and saying so plainly when the answer is *nothing*,
      which is usually the most useful sentence in the notes;
    - the version-category note, if a breaking change shipped as a patch;
    - a pointer to `CHANGELOG.md` for the full history.

    The upgrade section is not a duplicate of the changelog's *Migration:* notes.
    Those are per-change and written as the change lands; this is the whole
    upgrade in one place, written once the release is known.

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
