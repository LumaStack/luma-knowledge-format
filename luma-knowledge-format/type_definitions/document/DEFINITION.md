---
type: type_definition
defines: document
fields:
  type:             { field_presence: required,   field_type: text,               desc: "What kind of Document this is — the one hard conformance requirement." }
  title:            { field_presence: recommended, field_type: text,               desc: "Human label; may fall back to the filename." }
  description:      { field_presence: optional,    field_type: text,               desc: "One-sentence summary; used by indexes and search." }
  tags:             { field_presence: optional,    field_type: list of text,       desc: "Categorization, typically nested via / (e.g. ml/generative)." }
  stage:            { field_presence: optional,    field_type: enum,               values: [draft, provisional, stable, archived, unknown], desc: "What a change owes a reader — nothing, notice, or a path. Default unknown." }
  survival:         { field_presence: optional,    field_type: enum,               values: [temporary, probationary, intended, promised], desc: "How much you should expect this to last. Default intended, and often left unwritten." }
  created:          { field_presence: optional,    field_type: actor_event,        desc: "Original author and creation time. Immutable.." }
  modified:         { field_presence: recommended, field_type: actor_event,        desc: "Last editor and last meaningful change. Advances on edit.." }
  verified:         { field_presence: optional,    field_type: list of actor_event, desc: "Independent confirmation events.." }
  sources:          { field_presence: optional,    field_type: list,               desc: "Materials the content derives from (bespoke shape).." }
  stale_after:      { field_presence: optional,    field_type: date,               desc: "The content SHOULD be re-checked after this date." }
  matches:          { field_presence: optional,    field_type: list_or_keyword,    values: [eager, nothing], desc: "What makes this Document surface — eager, nothing, or a list of conditions. Absent means nothing." }
---

# document

The root type. Every Document is one, and every other type extends it —
implicitly, so a Type Definition never restates these fields.

This is the one Type Definition that declares core fields rather than domain
fields, because it is where the core fields come from.

`document` is a real type as well as the root: a file with nothing more specific
to say may declare `type: document` rather than inventing a name for it.

**The root declares `matches`, so any Document may say what surfaces it.**
It mostly earns its keep on `policy` and `procedure` — the two kinds that
act on a consumer. Background usually does not act; it is normally reached
through the things that do, so a Document that neither binds nor runs
declares it only where its subject genuinely arises on its own.
