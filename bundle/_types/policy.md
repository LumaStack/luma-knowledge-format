---
type: type_definition
defines: policy
extends: document
fields:
  on_violation: { field_presence: optional, field_type: enum, values: [allow, audit, warn, require_reason, require_approval, block], desc: "What a consumer SHOULD do when this policy is not complied with. Intent, never a guarantee — see below." }
---

# policy

A course of action adopted, and the reasoning that makes it worth holding.

It adds one field. A policy otherwise needs a name, a line on what it governs,
and a body.

**The range is wide on purpose.** A guardrail nobody may cross, a convention of
house style, what *done* means here, when to stop and ask a person, what this
project owns and must not own — all of these are courses of action somebody
adopted, and all of them must be in force to be worth anything. Nothing narrows
this to rules about risk or security; a naming convention is as much a policy as
a secrets rule, and differs only in what it costs to break.

Strength of obligation is a property of an individual policy, not a reason to
split the type — and not a field either. **A policy binds because it is a
policy**; how strongly is what its body says, in the words its author chose. A
scale would only restate the type on documents that bind and invite a soft tier
for documents that do not bind at all, which are `document`s wearing the wrong
type.

## `on_violation`

**What a consumer SHOULD do at the moment this policy is not complied with**,
ordered by how much reaches the actor:

| | |
| --- | --- |
| `allow` | nothing intercepts. The default, and the honest state of most policies |
| `audit` | detected and recorded, silently |
| `warn` | detected, the actor is told, and it proceeds |
| `require_reason` | proceeds only if a reason is recorded |
| `require_approval` | stops until a third party approves |
| `block` | stops; there is no path through |

**Intent, never a guarantee.** Like everything a type declares (§10.5), this
says what the author asked for. A consumer that cannot intercept SHOULD say so
rather than silently doing the nearest thing it can — a policy that reads as
enforced and is not is worse than one that never claimed to be.

**`audit` before teeth.** Detecting and recording without changing behaviour
tells you the real violation rate before anyone decides a rule deserves
stopping power, and nothing else on the ladder offers that.

## What a consumer does with it: **is bound by it**

A rule that constrains the consumer's own behaviour rather than informing it.
That is its place among the three things a consumer can do with a Document
(§10.4) — a `workflow` you **run**, and a plain `document` you **read**.

**Binding is not presence, and this type claims only the first.** Whether a
policy is in front of anyone is a consumer's decision, derived at most from
`applies_to` (§5.2) — and the two are orthogonal: **a policy binds whether or
not it happens to be loaded.**

## What dispatches on it

**A policy binds; a workflow is run.** A consumer assembling what an agent
should know treats a policy as a constraint on its own behaviour — these are the
rules in force — where a procedure is something it carries out when asked. Same
prose, opposite handling.

**A rule nobody can reach governs nothing**, which is true and is a separate
problem. It argues for making sure a policy is *findable* — something always
present naming the rules that exist — not for defining the type in terms of
being loaded. Conflating the two is what made this type look like a loading
setting wearing a different name.

The type is what tells them apart. Both are prose, and
without the name a consumer has no way to know that one belongs in permanent
context and the other behind an invocation.

**This is the intended difference rather than a shipped one**, and the reason
the type must be built in rather than defined per Bundle: **tooling that makes a
policy hard to ignore can only be written against a name the format guarantees.**
A type each Bundle defines privately is one no consumer can rely on finding.

Named here so it can be built against — and so that a `policy` which never
acquires it is understood to have been falsified rather than merely unused.

## `policy` and `type_definition` both state rules

The line is worth drawing, because content filed on the wrong side is
unfindable.

A **Type Definition** governs the *shape of a Document* — which fields it
carries and what they hold. Its rules are checkable against a file.

A **policy** governs *conduct* — what someone does, when, and to what degree.
Often there is no single file to check it against.

*"A decision record carries a `decided` date"* is a Type Definition. *"Record a
decision before an irreversible change"* is a policy.

## The reasoning is the durable half

A policy without its reasoning is unfinished. The answer is perishable — the
constraint that forced it disappears, or the tooling changes underneath it — and
the argument is what lets someone disagree on the merits later rather than
guessing at intent.

Nothing in the format enforces this. It is the convention that makes the type
worth having.
