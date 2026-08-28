---
type: type_definition
defines: type_definition
extends: document
fields:
  defines: { field_presence: required,   field_type: text, desc: "The type name this document governs.." }
  extends: { field_presence: optional,    field_type: text, desc: "A single parent type to inherit from.." }
  fields:  { field_presence: recommended, field_type: text, desc: "The field declarations. *Field declarations*. See the note below on its field type." }
---

# type_definition

A Type Definition declares a type's contract — the fields its Documents carry,
how strongly each is expected, and what shape each value takes. It is an
ordinary Document, which is what makes the format self-hosting: types are
described in the same form as everything else, in plain markdown, in git.

**Known gap.** `fields` is a map of field declarations, and LKF has no
`field_type` for a map — user-defined composite field types are deferred (see
`docs/ROADMAP.md`). It is declared as `text` here as a placeholder, which is
honest about the gap rather than inventing a vocabulary the spec does not have.
