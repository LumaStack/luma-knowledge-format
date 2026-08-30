---
type: type_definition
defines: document
fields:
  type:             { field_presence: required,   field_type: text,               desc: "What kind of Document this is — the one hard conformance requirement." }
  title:            { field_presence: recommended, field_type: text,               desc: "Human label; may fall back to the filename." }
  description:      { field_presence: optional,    field_type: text,               desc: "One-sentence summary; used by indexes and search." }
  tags:             { field_presence: optional,    field_type: list of text,       desc: "Categorization, typically nested via / (e.g. ml/generative)." }
  lifecycle: { field_presence: optional,    field_type: enum,               values: [draft, provisional, stable, archived], desc: "." }
  survival:         { field_presence: optional,    field_type: enum,               values: [experimental, intended, promised], desc: "How much you should expect this to last. Default intended, and often left unwritten." }
  created:          { field_presence: optional,    field_type: actor_event,        desc: "Original author and creation time. Immutable.." }
  modified:         { field_presence: recommended, field_type: actor_event,        desc: "Last editor and last meaningful change. Advances on edit.." }
  verified:         { field_presence: optional,    field_type: list of actor_event, desc: "Independent confirmation events.." }
  sources:          { field_presence: optional,    field_type: list,               desc: "Materials the content derives from (bespoke shape).." }
  stale_after:      { field_presence: optional,    field_type: date,               desc: "The content SHOULD be re-checked after this date." }
---

# document

The root type. Every Document is one, and every other type extends it —
implicitly, so a Type Definition never restates these fields.

This is the one Type Definition that declares core fields rather than domain
fields, because it is where the core fields come from.

`document` is a real type as well as the root: a file with nothing more specific
to say may declare `type: document` rather than inventing a name for it.

**The root declares nothing about when a Document is wanted.** `matches` is
declared by `policy` and `workflow` and by nothing else — the two kinds
that act on a consumer. Background does not act; it is reached through the
things that do.
