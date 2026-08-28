---
type: type_definition
defines: bundle
extends: document
fields:
  version:     { field_presence: required,   field_type: semver,       desc: "This Bundle's content version. §11.1." }
  published:   { field_presence: recommended, field_type: date,         desc: "When this version was published." }
  consumers:   { field_presence: optional,    field_type: list of text, desc: "Kinds of consumer that may adopt this Bundle. Open vocabulary. §11.1." }
  entrypoint: { field_presence: optional,    field_type: text,         desc: "Document ID (§3) of where a reader should start. §11.1." }
---

# bundle

A Bundle's own Document, living at the Bundle root as `BUNDLE.md` (§11.1). It
describes the Bundle rather than any file inside it.

`version` is mandatory because a Bundle without one cannot be pinned, compared,
or reported as outdated — a consumer can say nothing honest about it. It is the
Bundle's *content* version, distinct from `lkf_version` (§12), which is the
version of the format grammar.

`consumers` is an **open vocabulary and LKF defines no values for it.** Where a
distribution model has more than one kind of consumer — a repository and an
organization, a workstation and a server, a patient and a clinic — a Bundle may
say which of them it is for. The format does not know what those kinds are; the
values belong to whoever is distributing, as `tags` does.

**The values are kinds, never instances.** This says what may adopt the Bundle,
not what has — a permission the publisher grants, not a record anything writes
back into.

It is a list because a Bundle may legitimately apply to more than one kind, which
is the whole reason it is a field rather than a directory a distributor sorts
into. A directory can express only one, forcing the *publisher* to answer a
question that often belongs to the *adopter*. Absence says nothing — not "none"
and not "all" — and no consumer rejects a Bundle for it (§4).

`description` is not declared here: it is a core field already (§5.1), and
inheritance is add-only (§10.3), so a type may not restate an inherited field to
strengthen its obligation. A Bundle SHOULD still carry one.
