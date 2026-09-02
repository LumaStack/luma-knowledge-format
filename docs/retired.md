# Retired names

**Names this specification once defined and no longer does.** A retired name is
**free**: nothing reserves it, no consumer reads it, and a producer may use it
for unrelated domain data — the open-vocabulary rule in the specification,
[Frontmatter layout and conformance](../luma-knowledge-format/specification/lkf.md#frontmatter-layout-and-conformance),
applies to it like any other unrecognised name.

**This file is history, not specification.** Nothing here is normative and no
rule in the specification depends on it. It exists to answer one question — *is `preload`
still a thing?* — without a reader having to diff releases or search prose for
a name that, by definition, is no longer written anywhere.

**"Retired", not "released".** These names were let go, and this project already
uses *release* for the opposite thing: a published, tagged version. One word
cannot mean *shipped* and *given up* in the same repository.

## The names

| name | was | retired in | replaced by |
|---|---|---|---|
| `concept` | a Document type for background | `v0.0.10` | `document` |
| `preload` | a core field: when a Document should be placed in front of a reader | `v0.0.12` | nothing — delivery is a consumer's decision, derived |
| `compliance` | a field grading how strongly a rule obliged compliance | never specified; invented and withdrawn in the estate during `v0.0.13` | nothing — a `policy` binds by being one, and `on_violation` says what happens when it does not |
| `index.md` | a reserved file: derived per-directory navigation | `v0.0.14` | nothing — see [Reserved files](../luma-knowledge-format/specification/lkf.md#reserved-files) |
| `applies_to` | the field naming what makes a Document surface | `v0.0.15` | [`matches`](../luma-knowledge-format/specification/lkf.md#matches) |
| `entry_point` | the Bundle field naming where a reader should start | `v0.0.17` | [`entrypoint`](../luma-knowledge-format/specification/lkf.md#bundlemd) |
| `workflow` | the built-in type for a procedure a consumer runs | `v0.0.19` | `procedure` — the same contract, named for the invariant across every rendering a consumer projects it into |
| `always` | the `matches` value meaning *nothing gates it* | `v0.0.19` | [`eager`](../luma-knowledge-format/specification/lkf.md#matches) — the same position in the field, named for the reach containment actually gives it |
| `entrypoint` | the Bundle field naming where a reader should start | `v0.0.19` | `matches: eager` on the Document itself — the start-here claim travels on the thing it is about |

## Why each was retired

**`preload`** was the one place this specification described how a Document
should be *consumed* rather than what it is. Consumption belongs to whatever
distributes and loads Bundles. `matches` is not its replacement: it says what
makes a Document *surface*, which is a property of the content, and any
decision about loading is one a consumer derives from it.

**`compliance`** never reached the specification at all. It was invented and
withdrawn in the estate, which is exactly why it is listed here — a name that
was never specified is the one a reader is most likely to find in an old file
and least able to look up.

**`index.md`** reserved *derived navigation for a directory; a rebuildable
cache, not a source of truth.* Its structure was never specified, nothing
implemented it, and it is a name static site generators resolve in lowercase —
so keeping it would have been the reserved-file rule's only exception. A future
reservation for derived navigation should take a name of its own.

**`applies_to`** obliged an author to write a false sentence. `applies_to:
everything` claims a Document governs everything, and none does — what a rule
governs is stated in its body, and no frontmatter value widens or narrows it.
The vocabulary had also outgrown the name: `path` is a target, but `event` is a
moment, and nothing about a moment is a resource a rule scopes over.

**`entry_point`** wrote one idea as two words, and invited the collision it
caused: a consumer building a project-level entrypoint read it as a different
concept that happened to share a word.

## What a retired name means for a tool

A retired name appearing in a published Document is **not a conformance
question.** The specification stands: such a Document is valid, and a consumer
**MUST NOT** reject it. But a retired name in a Document's *prose* is usually a
rule still instructing authors to declare something nothing reads — which has
happened here, where `organizing-a-bundle` taught `preload` for two releases
after its removal. A consumer **MAY** report it.

**A retired name may be reserved again, and doing so is a breaking change** even
though nothing currently uses it — a Bundle that had adopted the free name for
its own purposes would silently acquire the specification's meaning. That rule
is stated in [Versioning](../luma-knowledge-format/specification/lkf.md#versioning); it is repeated here because
this is the list a reader consults before reusing one of these names.
