---
type: type_definition
defines: workflow
extends: document
fields: {}
---

# workflow

A procedure a person or an agent follows. The body is the instructions.

It adds no fields of its own — a workflow needs a name, a line on when it
applies, and a body, all of which the root already provides.

## How it reaches a consumer: **invoked**

Loaded while it is being followed, absent otherwise. That is its place among the
three ways a Document reaches a consumer (§10.4) — a `policy` is kept standing,
and a plain `document` is retrieved when relevant.

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
