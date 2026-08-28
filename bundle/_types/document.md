---
type: type_definition
defines: document
fields:
  type:             { field_presence: required,   field_type: text,               desc: "What kind of Document this is — the one hard conformance requirement (§4)." }
  title:            { field_presence: recommended, field_type: text,               desc: "Human label; may fall back to the filename." }
  description:      { field_presence: optional,    field_type: text,               desc: "One-sentence summary; used by indexes and search." }
  tags:             { field_presence: optional,    field_type: list of text,       desc: "Categorization, typically nested via / (e.g. ml/generative)." }
  lifecycle_status: { field_presence: optional,    field_type: enum,               values: [draft, provisional, stable, archived], desc: "§6." }
  survival:         { field_presence: optional,    field_type: enum,               values: [experimental, intended, promised], desc: "Whether this is expected to last. Default intended, usually unwritten. §14." }
  created:          { field_presence: optional,    field_type: actor_event,        desc: "Original author and creation time. Immutable. §7.1." }
  modified:         { field_presence: recommended, field_type: actor_event,        desc: "Last editor and last meaningful change. Advances on edit. §7.1." }
  verified:         { field_presence: optional,    field_type: list of actor_event, desc: "Independent confirmation events. §7.2." }
  sources:          { field_presence: optional,    field_type: list,               desc: "Materials the content derives from (bespoke shape). §7.3." }
  stale_after:      { field_presence: optional,    field_type: date,               desc: "The content SHOULD be re-checked after this date." }
---

# document

The root type. Every Document is one, and every other type extends it —
implicitly, so a Type Definition never restates these fields (§10.3).

This is the one Type Definition that declares core fields rather than domain
fields, because it is where the core fields come from.

`document` is a real type as well as the root: a file with nothing more specific
to say may declare `type: document` rather than inventing a name for it.

**The root declares nothing about when a Document is wanted.** `matches` is
declared by `policy` and `workflow` and by nothing else (§10.7) — the two kinds
that act on a consumer. Background does not act; it is reached through the
things that do.
