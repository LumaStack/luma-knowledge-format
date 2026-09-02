# What LKF is, and why it exists

An introduction for someone who has just arrived. [The specification](../luma-knowledge-format/specification/lkf.md) is the
rules; this is the reasoning, with enough examples to see the shape.

## The problem

A repository's knowledge used to be written by a handful of people who mostly
remembered writing it. That is no longer true. Notes, decisions, runbooks and
summaries are increasingly written by agents, revised by other agents, and read
months later by someone who was not there for any of it.

Three things go wrong, and none of them are solved by writing more markdown.

**You cannot tell what you are looking at.** Open a file in a knowledge base and
try to answer: who wrote this, when, has anyone checked it, and is it still
true? Usually nothing in the file says. So you either trust everything, which is
how a confident hallucination becomes institutional fact, or you trust nothing,
which makes the corpus worthless. Neither is a knowledge base.

**Metadata dies when a file moves.** Every tool invents its own frontmatter.
Move a file from one system to another and the keys mean nothing — or worse,
mean something else. Knowledge that cannot travel is knowledge locked to
whatever tool happened to be fashionable when it was written.

**Schemas are usually all-or-nothing.** Most formats let you describe structure
and then reject anything that does not match. Real knowledge bases are
half-finished by nature: a file gets written before its links exist, a type gets
used before anyone defines it. A format that rejects those is a format people
route around.

## What LKF is

Markdown files with YAML frontmatter, in a git repository. That is the whole
substrate — no database, no server, no SDK. Any index or cache is derived and
can be thrown away.

On top of that, three things:

**One hard rule.** A Document declares a `type`. That is the only requirement
the format makes.

**A small core of provenance fields**, so a file can answer the questions above
in-band: who created it, who last changed it, who has independently confirmed
it, what it derives from, and whether it is still meant to be relied on.

**An open extension mechanism.** Any `type` may declare its own fields by
writing a Type Definition — itself an ordinary Document. Your domain
specialises the format without asking anyone and without touching the core.

And one posture that runs through all of it: **consumers tolerate rather than
reject.** An unknown type, a missing optional field, an extra key, a link
pointing at a file nobody has written yet — none of those make a file invalid.
Strictness is something you opt into with a validator, never the default.

## What LKF is not

- **Not a taxonomy.** It does not tell you what types to have. `meeting`,
  `lab_result`, `incident` — yours to define.
- **Not a storage or search system.** It describes files on disk, not the
  machinery around them.
- **Not a replacement for a domain schema.** Where you already have a data model
  or an API contract, LKF points at it rather than absorbing it.
- **Not a validator.** Nothing rejects your files. Tools may choose to check
  them; the format never requires it.

## Examples

**The smallest legal Document.** One field.

```markdown
---
type: note
---

Grafana dashboards are provisioned from `ops/grafana/`, not edited in the UI.
```

Nothing else is required, and this is a complete, conformant file. Everything
below is added because it earns its place, never because the format demands it.

**A knowledge-base entry that answers "should I trust this?"**

```markdown
---
type: document
title: Diffusion Models
tags: [ml/generative]
lifecycle: stable
created:  { by: agent:opus-5, at: 2026-05-02T14:20:00Z }
modified: { by: human:fsmith, at: 2026-08-01T10:00:00Z }
verified:
  - { by: human:fsmith, at: 2026-08-01T10:05:00Z }
sources:
  - id: ddpm
    resource: https://arxiv.org/abs/2006.11239
    title: Denoising Diffusion Probabilistic Models
    last_modified: 2020-12-16
---

A class of generative models that learn to reverse a gradual noising process.[^ddpm]
See [[score-based-models]] for the continuous-time formulation.
```

Written by an agent, edited and then confirmed by a person. `verified` is a
*list* because confirmation is an event that can happen repeatedly and by
different actors — and the trust tier is **derived** from it rather than stored,
so nobody can label a file trustworthy without leaving a trace of who said so.

`[[score-based-models]]` may not exist yet. That is legal: an unresolved link is
often just knowledge not written down yet.

**Recording that nobody knows who wrote something.**

```markdown
---
type: runbook
title: Rotating the deploy key
created: { by: unknown:unknown, at: 2024-11-03 }
stale_after: 2027-01-01
---
```

`unknown:unknown` is a supported value, and using it is better than omitting the
field. A missing author reads as an oversight; a recorded unknown reads as a
fact. A tool that *can* identify its actor should — this exists for genuine
ignorance, not as a default for tools that never asked.

**Adding your own type.** No permission required, no central registry.

```markdown
---
type: type_definition
defines: incident
fields:
  severity:  { field_presence: required,   field_type: enum, values: [sev1, sev2, sev3] }
  detected:  { field_presence: required,   field_type: datetime }
  resolved:  { field_presence: optional,    field_type: datetime }
  responder: { field_presence: recommended, field_type: wikilink }
---

# Incident

One file per incident. Opened when detected, closed when `resolved` is set.
```

Drop that in `_types/incident.md` and `type: incident` now has a published
contract. Note what it does *not* declare: `created`, `modified`, `verified`,
`title`. Those arrive automatically from the root type, so a Type Definition
only ever describes what is specific to the domain.

## The built-in types

The built-in types ship with the format, [written in the format
itself](../luma-knowledge-format/_types/). Reasons to care, rather than a restatement of each
file:

**[`document`](../luma-knowledge-format/_types/document.md)** — the root every type implicitly
extends. You will rarely write `type: document` on a real file, and that is
fine: its value is that every Document you *do* write already has somewhere to
put provenance, without you declaring anything.

**[`procedure`](../luma-knowledge-format/_types/procedure.md)** — a procedure a
consumer *runs* rather than reads, projected into whatever form does the
running. Nothing else in a Bundle makes that claim, which is why the type
exists rather than a tag.

**[`policy`](../luma-knowledge-format/_types/policy.md)** — a rule that
constrains the consumer's own behaviour instead of informing it. Two Documents
can be identical prose with identical fields, and one belongs in permanent
context while the other belongs behind an invocation; nothing but the `type`
can say which.

**[`bundle`](../luma-knowledge-format/_types/bundle.md)** — makes a directory into something
you can hand to someone else: a self-contained unit with a version they can pin
and compare. Matters the moment your knowledge stops being only yours.

**[`type_definition`](../luma-knowledge-format/_types/type_definition.md)** — the extension
point, and the reason the core can stay small. It is also self-hosting: the
thing that defines types is itself a type, defined the same way.

## Where to go next

- [`specification/lkf.md`](../luma-knowledge-format/specification/lkf.md) — the rules, in full. Short by design.
- [`principles.md`](principles.md) — the values a design question is settled by
  when the specification is silent.
- [`roadmap.md`](roadmap.md) — what is still open, and what `v0.1.0` would mean.

A warning worth repeating from the readme: this format is in its `0.0.z` tier
and is explicitly unstable. Breaking changes may ship in a patch release. It has
been reasoned about carefully and used barely at all — which is exactly the
moment when telling us it does not fit is most useful.
