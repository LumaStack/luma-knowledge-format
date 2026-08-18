---
type: type_definition
defines: document
fields:
  type:             { obligation: mandatory,   field_type: text,               desc: "What kind of Document this is — the one hard conformance requirement (§4)." }
  title:            { obligation: recommended, field_type: text,               desc: "Human label; may fall back to the filename." }
  description:      { obligation: optional,    field_type: text,               desc: "One-sentence summary; used by indexes and search." }
  tags:             { obligation: optional,    field_type: list of text,       desc: "Categorization, typically nested via / (e.g. ml/generative)." }
  lifecycle_status: { obligation: optional,    field_type: enum,               values: [draft, provisional, stable, archived], desc: "§6." }
  created:          { obligation: optional,    field_type: actor_event,        desc: "Original author and creation time. Immutable. §7.1." }
  modified:         { obligation: recommended, field_type: actor_event,        desc: "Last editor and last meaningful change. Advances on edit. §7.1." }
  verified:         { obligation: optional,    field_type: list of actor_event, desc: "Independent confirmation events. §7.2." }
  sources:          { obligation: optional,    field_type: list,               desc: "Materials the content derives from (bespoke shape). §7.3." }
  stale_after:      { obligation: optional,    field_type: date,               desc: "The content SHOULD be re-checked after this date." }
  preload:          { obligation: optional,    field_type: enum,               values: [mandatory, recommended, optional], desc: "How strongly this Document should be loaded before working with its Bundle. Absent means optional. §5.2." }
---

# document

The root type. Every Document is one, and every other type extends it —
implicitly, so a Type Definition never restates these fields (§10.3).

This is the one Type Definition that declares core fields rather than domain
fields, because it is where the core fields come from.

`document` is a real type as well as the root: a file with nothing more specific
to say may declare `type: document` rather than inventing a name for it.

`preload` is the one field here that describes how a Document should be
*consumed* rather than what it is or where it came from. It lives on the root
because any Document may carry it, not only a Bundle's manifest — see §5.2 for
what each level obliges a consumer to do.
