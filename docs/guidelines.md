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
4. **Edit** — apply it to the specification (and `principles.md` if a principle is affected), and add a `CHANGELOG.md` entry under `## [Unreleased]` for any behavior-affecting change (see [Changelog](#changelog)).
5. **Release** — see [Cutting a release](#cutting-a-release). It is a checklist rather than a sentence because several of its steps are silent when missed.

## Branching

`main` always reflects the **latest released version** — it advances only when a release is cut, so `main` and the newest tag stay in lockstep. Checking out `main` always gives a coherent release, never half-finished work.

- **Work toward the next release accumulates on a branch named for it** — `v0.0.14-draft`. It is created when the first change toward that release starts and **deleted when it ships**. Between releases there may be no draft branch at all, which correctly says nothing is pending.
- **The branch name decides the version, at the start.** You cannot name it without answering *is this breaking?* — which is the question most often decided wrongly, and it is easier to answer while the change is fresh than at release time when you have stopped thinking about it. It also means **the version is decided once** rather than per pull request, so two changes in flight cannot both claim the next number.
- **Feature branches fork off the draft and merge back.** A small change may go straight to the draft.
- **Merge, never squash or rebase.** Every merge is a merge commit. This project requires the rationale for a change to live in its commit message, and squashing collapses a branch's messages into one — losing exactly what the rule exists to preserve. Rebasing rewrites them. Squash and rebase merging are disabled on the repository, so the buttons are absent rather than merely discouraged.
- **A fix outside the Bundle may go straight to `main`** — a typo, a broken link, anything under `docs/`, `README.md` or `.luma/`. No version moves and no tag is cut. The invariant is that `main` is a coherent released specification, not that it is byte-identical to a tag. **Inside the Bundle nothing goes straight to `main`:** every change there cuts a release, down to a typo (see [What moves the version](#what-moves-the-version)).

> **`develop` is retired.** It was a long-lived second branch accumulating anything, and it rotted: thirty-one commits behind `main` and unused since `v0.0.10`. A branch named for the release it is building is finite, says what it is for, and disappears when it is done. `develop` said only *later*.

## Where history lives

**Two files hold history. Every other file in this repository is current or
forward-looking.**

| what | where |
|---|---|
| what changed in a release, and why | `CHANGELOG.md` |
| a name the format once defined and no longer does | [`retired.md`](retired.md) |

**There is one specification file, `luma-knowledge-format/specification/lkf.md`.
There are no versioned specification filenames, none are planned, and nothing
supersedes anything.** Multi-versioning would follow from adopters depending on
different specification versions at once, and that day has not come; the
question is parked with its re-open trigger in
[`one-folder-per-specification-version.md`](../.luma/backlog/ideas/one-folder-per-specification-version.md),
and taking it up would be a deliberate decision with real obligations
attached rather than a filing change.

**Everywhere else — `README.md`, the rest of `docs/`, `CLAUDE.md`, `.luma/`,
the Bundle — a sentence that is only true in the past is a defect.** Not untidy:
a defect. A reader cannot tell a live rule from an obituary, and an agent acts
on what it found. `concept` was retired in `v0.0.10` and `docs/` was still
teaching it as a shipping built-in eight releases later, linking to a file that
had not existed the whole time.

### A rule may rest on the past and still be current

**History is a record of what is no longer true. A rule that cites what produced
it is not that.** The sentence is doing present work — its claim is live, and
the past is the evidence. **Nothing here is in tension, so nothing needs an
exception**, and a rule that appeared to require one would be the wrong rule.

Three sentences, and only the last is a defect:

| | |
|---|---|
| **history** — belongs in the two files | *`concept` was a Document type for background, retired in `v0.0.10`* |
| **a live rule with its evidence** — belongs wherever the rule does | *`README.md` stated the version and went stale three times, which is why it no longer does* |
| **teaching something dead** — a defect anywhere | *`concept` is the ordinary knowledge-base entry; reach for it when writing a wiki page* |

**The test is the claim, not the tense.** *What is this sentence asking a reader
to do?* Follow a rule in force — keep it, however much past it carries. Reach
for something nothing reads — that is the defect, however briefly it is
mentioned.

**This is why the rule costs nothing.** A document built on its own history is
already current: every rule in this file arrived from something that went wrong,
and saying so is what makes the rules arguable rather than arbitrary. **Strip
that and you get assertions nobody can weigh** — a worse document, in service of
a rule that was never aimed at it.

**Design rationale is the same shape.** *Why this field is optional*, *why this
vocabulary is closed* — the claim is a rule in force, so it stays.

**Two homes that are not files.** Why a decision went the way it did lives in
**the commit message that landed it**; something not decided yet lives in
[`roadmap.md`](roadmap.md), which is forward-looking and therefore not an
exception to any of this.

**Never call it "released".** A *release* in this project is a published, tagged
version. A name the format gave up is **retired** — one word cannot mean
*shipped* and *given up* in the same repository.

## What the specification may contain

**The strictest instance of [Where history lives](#where-history-lives).**
`lkf.md` **states the current specification and nothing else.** No history, no
record of what a field used to be called, no note explaining why something was
removed. A reader — and increasingly a reader is an agent with the whole file in
its context — should be able to take every sentence in it as currently true.

**History muddies exactly the file that can least afford it.** A paragraph
saying *`applies_to` was this field's name through `v0.0.13`* teaches a name
nobody should write, in the one document whose job is to say what to write. An
agent reading the spec to produce a Document cannot reliably tell a live rule
from an obituary, and it will occasionally emit the dead one — which happened:
`organizing-a-bundle` taught `preload` for two releases after its removal.

**Design rationale stays**, on the test in [Where history
lives](#where-history-lives): *why this field is optional*, *why this vocabulary
is closed* explain a rule currently in force.

**What `lkf.md` may not do is name a retired thing at all**, even in service of
a live rule. Elsewhere the evidence for a rule can safely include what it
replaced. Here it cannot: the file is read to *produce* Documents, by readers
holding all of it at once, and a dead field named anywhere in it is a dead field
in the reader's context. **The rule is the same; only the margin for error is
different.** Rationale in `lkf.md` argues from what the rule achieves, never
from what preceded it.

## Changelog

`CHANGELOG.md` lets a reader see **what changed between versions at a glance** — without diffing commits — and lets them skip minor edits that don't affect behavior.

- **Include** behavior-affecting spec changes: additions, renames/migrations, deprecations, and removals of field types, core fields, and type mechanics, plus other notable highlights.
- **Omit** anything that doesn't change behavior — wording, typos, formatting, example tweaks, and project-process changes.
- **Unless it is the whole release.** Every change inside the Bundle cuts a version ([What moves the version](#what-moves-the-version)), so a release may contain nothing but errata. **A dated section with nothing under it reads as a missing entry, not as nothing happening** — give it one line under **Fixed** saying what was corrected and that no behavior changed. That sentence is the most useful thing in the notes for somebody deciding whether to upgrade.
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

**The body never opens with a heading.** ⚠️ **Do not paste the changelog section in whole** — it carries `## [x.y.z] — YYYY-MM-DD` at its top, which GitHub renders directly beneath the release title that already says the version. That is the same second title [A release is titled with its version and nothing else](#a-release-is-titled-with-its-version-and-nothing-else) exists to prevent, moved one line down — and the rule is not satisfied by fixing the title afterwards, which is what has had to happen. **The first `##` in the body is a change group or `Upgrading`,** never the version.

**A paste is also how the `Upgrading` section goes missing** — the changelog has no such section to copy, so pasting produces notes without the one part written for the person deciding whether to upgrade. `v0.0.14`, `v0.0.15` and `v0.0.16` all shipped this way. The body is **written from** the changelog, not **copied from** it.

**Section references must be resolved.** The changelog and the spec cross-reference by section name; a body pasted from an older changelog can carry `§13`-style numbers this specification stopped using when [sections became named rather than numbered](#what-the-specification-may-contain). Read the body once as a stranger would before publishing.

### Where the version lives

**Two files state the current version. Both are parsed, and both have to.**

- `luma-knowledge-format/specification/lkf.md` — the `lkf_version` **frontmatter** field. The specification's version.
- `luma-knowledge-format/BUNDLE.md` — the `version` field. The version of the Bundle that ships the specification and the built-in Type Definitions.

**In `lkf.md` it lives in the frontmatter and never in the body.** The body once
carried a `Version` header as well, which is the same number stated twice in one
file — free to disagree with itself, and it is prose that drifts while the
parsed value stays right. **Frontmatter is where a version is machine-readable,
and `lkf_version` is a field this specification already defines for exactly this
claim**, so the body has nothing left to add.

**Neither is redundant.** The built-in types are a rendering of what the specification says, so a Bundle claiming a different version than the spec it renders is lying about which spec it implements — and a Bundle manifest has to carry a version whatever this project prefers, because that is what makes it pinnable. Continuous integration refuses a mismatch between them.

**`README.md` used to be a third, and was removed on 2026-08-26.** It stated the version in its Status line, which was the only one of the three a reader looked at rather than a tool parsed — and it went stale three times: it sat at `v0.0.9` through two releases, and then `v0.0.14` shipped and was tagged while it still said `v0.0.13`. Each time the response was a better check rather than one fewer place to forget.

**The lesson is the general one, so it is written as a rule rather than an anecdote: a number nothing parses is not worth duplicating.** The README now says *unstable* and points at the specification and the tags. A reader who wants the current version gets it from the newest release, which cannot go stale because it is not written down anywhere.

**`CHANGELOG.md` carries versions too, and it is not a third place to forget.**
It records **versions that shipped** — `## [x.y.z] — YYYY-MM-DD`, written once at
release and never revised. The two files above make a live claim about *what the
version is now*, which is the kind that goes stale; a dated changelog entry is a
fact about the past, which cannot. Adding one is [step 2 of cutting a
release](#cutting-a-release).

**Nowhere else states the current version.** Not `README.md`, not `CLAUDE.md`,
not `docs/`, not `.luma/`, not `.github/`. A number in any of them is one a
person maintains by remembering to, and the record above is what remembering to
looks like over three releases.

**A reference to a past version is not a statement of the current one.**
`retired.md` naming the release that retired a field, `roadmap.md` naming the
one that settled a question, this file naming the releases that shipped badly —
each is a fact about the past, like a changelog entry, and none of them goes
stale. They stay. **The rule bans a second answer to *what version is this
now*,** not the use of version numbers.

**Resist a third live statement.** A new file needing the current version should
read it from one of the two, or go without.

## Versioning & release policy

### What moves the version

**The Bundle is the release.** `luma-knowledge-format/` is the unit of
distribution, and a version exists to say which copy of it somebody holds. So
the question is never how significant a change feels — it is whether the change
is inside the thing being distributed.

**A change to any file in `luma-knowledge-format/` moves the version:**

- the specification, `specification/lkf.md`
- the built-in Type Definitions, `_types/`
- the Bundle's own manifest and behavior, `BUNDLE.md`

**Plus `LICENSE`**, which is the only file beyond the directory that moves the
version. **It stays outside the Bundle and should not be copied in** — a licence
duplicated into the unit of distribution is not self-containment, it is a second
copy to keep in step. Self-containment is about what must be *read* to use the
Bundle, and nothing is read here: the terms are the same terms wherever they sit.

**But changing it changes how the Bundle may be distributed**, and that is a
change to what an adopter may do with the version they hold. A release says
*this is what you get and what you may do with it*, so the second half moves the
number as surely as the first.

**A change anywhere else does not.** `README.md`, `docs/`, `CHANGELOG.md`,
`CLAUDE.md`, `.luma/`, `.github/` — all of it is the project *around* the
Bundle. It maintains the Bundle rather than being part of it, and nobody who
adopts a version receives any of it. **A repository whose version moved because
a roadmap entry was reworded is reporting churn as change**, and a consumer who
upgraded for it got nothing.

**There is no carve-out inside the Bundle. A typo in `lkf.md` cuts a patch.**
The Bundle is what somebody adopted, and a copy differing from the version it
claims is not that version — the size of the difference does not change that.

**The rule is mechanical on purpose.** Whether an edit "really" changed meaning
is a judgment made by the person least placed to make it, at the moment they
most want the answer to be no. *It's only a typo* is how a Bundle comes to have
two contents under one number, and no consumer can tell which one they hold.

**`patch` is where these land**, and the tier already says so: *clarifications,
wording, errata*. A typo is errata. Nothing about the size of a change lets it
skip a version — the size decides the tier, not whether there is one.

### The ladder

- Scheme is **semver `major.minor.patch`**.
  - **patch** — clarifications, wording, errata; no grammar change.
  - **minor** — backward-compatible additions (new optional field, new reserved convention).
  - **major** — breaking changes.
- **Deprecate before removing.** A field or rule is marked `deprecated` (which validators warn on) for at least one minor version before a major removes it.
- Published versions are **git tags**; `main` equals the newest tag (the released spec). No versioned filenames or directories.
- While in `v0.0.z`, breaking changes MAY ship in a patch — the pre-1.0 tier is explicitly unstable.

## Rules for agents working on the format

An agent working in this repository MUST:
- **Read this file in full before cutting a release.** Not skim it, not recall it — open it. The evidence that this is not happening: `v0.0.12` and `v0.0.13` are tagged with no GitHub Release at all (step 7), and `v0.0.14`, `v0.0.15` and `v0.0.16` each shipped a body pasted whole from the changelog — duplicate heading, no `Upgrading` section — leaving the maintainer to correct the titles by hand.
- **Not change the specification normatively without a ratified decision.** Drafting a proposal is fine; merging it into the spec as settled is not.
- **Not put history anywhere but `CHANGELOG.md` and [`retired.md`](retired.md)** — see [Where history lives](#where-history-lives). A retired name goes in `retired.md`, what changed in a release goes in the changelog, a rationale goes in the commit message. A rule may cite what produced it and stays current by doing so; **the specification alone may not name a retired thing even then** ([What the specification may contain](#what-the-specification-may-contain)).
- **Not leave a retired name behind when one is retired.** Removing it from the specification is half the job — sweep `README.md`, `docs/`, `CLAUDE.md` and `.luma/` in the same change. `concept` was taught in `docs/` for eight releases after removal, and `index.md` for four.
- **Record the rationale in the commit message** for every accepted change, so the "why" is never lost. (When commit-log spelunking gets painful, graduate to an append-only `DECISIONS.md`.)
- **Work on a branch off `main`, never commit directly to `main`** (except a ratified critical hotfix) — `main` stays equal to the latest release.
- **Merge, never squash or rebase.** Squashing collapses the per-commit rationale this project requires be preserved.
- **Add a `CHANGELOG.md` entry** under `## [Unreleased]` for behavior-affecting changes; omit non-behavioral edits, unless they are the whole release — then say so in one line.
- **Keep changes additive within a minor version;** anything breaking requires a major bump and maintainer sign-off.
- **Honor the principles** in `principles.md`; if a change appears to violate one, stop and surface the conflict rather than proceeding.
- **Follow [Cutting a release](#cutting-a-release) in full** when preparing one. Two versions get bumped, not one, and three of the steps fail silently.

## Deferred (post-MVP)

Formal contribution guide (`CONTRIBUTING.md`), a `proposals/` RFC process, issue/PR templates, and `CODEOWNERS` — possibly added when necessary.
