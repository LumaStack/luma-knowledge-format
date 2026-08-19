---
type: type_definition
defines: concept
extends: document
fields: {}
---

# concept

A unit of knowledge in a knowledge base — the conceptual material LKF was first
written for. It may describe a tangible asset (a table, an API), an abstract
idea (a metric, a business process), or anything between.

## How it reaches a consumer: **retrieved when relevant**

Progressively disclosed — pulled in by need rather than held permanently. That
is its place in the three-way partition (§10.4), where a `workflow` is invoked
and a `policy` is kept standing.

It adds no fields of its own. Declaring `type: concept` says *this is knowledge*,
as against a task, a record, or a Bundle's own manifest.

## Under review

**`concept` is a candidate for removal**, and using it should be a deliberate
choice rather than a default.

It has existed since `v0.0.1` without a consumer that treats it differently from
a plain `document`. The retrieval mode above is the case for keeping it, and
that case is currently a claim rather than something anything implements — which
is precisely the condition §10.4 describes as *falsified rather than merely
unused*.

**What would settle it is a real knowledge base.** `concept` was the type the
format was first written for, and no durable knowledge base has yet been built
with it. Until one exists there is nothing to check the claim against, so the
name is held rather than removed — removing it and re-adding it later would cost
a collision with every Bundle that had meanwhile defined it privately.
