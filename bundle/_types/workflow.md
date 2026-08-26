---
type: type_definition
defines: workflow
extends: document
fields:
  matches:      { field_presence: optional, field_type: list_or_keyword, values: [always, nothing], desc: "What makes this Document surface — always, nothing, or a list of triggers. Absent means nothing. §10.7." }
  applies_to:   { field_presence: deprecated, field_type: list, desc: "The former name of `matches`, through v0.0.13. Read where `matches` is absent; report each use. §10.7." }
---

# workflow

A procedure a person or an agent follows. The body is the instructions.

It adds no fields of its own — a workflow needs a name, a line on when it
applies, and a body, all of which the root already provides.

## What a consumer does with it: **runs it**

A procedure, executed rather than read. That is its place among the three things
a consumer can do with a Document (§10.4) — a `policy` **binds** you, and a plain
`document` you **read**.

**This says nothing about when it loads.** `applies_to` (§10.7) says when its subject arises, and a consumer derives the rest. A workflow
is usually fetched when it is being followed and absent otherwise, but that is a
sensible default rather than part of what the type means.

## What dispatches on it

**A workflow is transformed; no other Document in a Bundle is.** Consumers
project it into whatever form their harness expects — an agent skill, a
generated command, an entry in a task runner — and that projection is selected
by the type, not by where the file sits. A path-based scan misses a file
somebody moved, and a capability that silently fails to be generated is the
worst outcome available: everything looks correct and the thing is simply not
there.

That is the difference the name buys, and it is why the type exists without
declaring a single field.

## `description` carries unusual weight

For most Documents `description` is a convenience. For a workflow it is what a
consumer reads to decide **whether this procedure applies at all** — the text
that selects it, before its body is read.

The root declares `description` as `optional` and inheritance is add-only
(§10.3), so this type cannot strengthen it. Treat it as required in practice: a
workflow without one is a procedure nothing will ever choose to run.

## What it does not say

A workflow says what to do. It does not say where it should be installed, in
what format, or for which tool — those belong to whatever consumes it. A
procedure naming its harness has bound vendor-neutral knowledge to whichever
assistant happened to be current when it was written.

## `applies_to`, and not `on_violation`

A workflow declares **when its subject arises** — *before a release*, *at the
first commit in a fresh repository* — so a consumer can reach for it then rather
than waiting to be asked.

**It does not declare `on_violation`, and that is the difference between the two
kinds that carry triggers.** A `policy` can be *broken*, at a moment, by an
action somebody takes — and something can act on that. The only way to fail a
workflow is **not to run it**, which is the absence of an action. Detecting
absence needs state no consumer is obliged to keep, so the format does not ask
for it.

The consequence, stated plainly because it is a real gap rather than an
oversight: **there is no way to say *you must run this when that moment
arrives*.** A workflow's body may say so, and nothing mechanical follows.
