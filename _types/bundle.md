---
type: type_definition
defines: bundle
extends: document
fields:
  version:   { obligation: mandatory,   field_type: semver, desc: "This Bundle's content version. §11.1." }
  published: { obligation: recommended, field_type: date,   desc: "When this version was published." }
---

# bundle

A Bundle's own Document, living at the Bundle root as `bundle.md` (§11.1). It
describes the Bundle rather than any file inside it.

`version` is mandatory because a Bundle without one cannot be pinned, compared,
or reported as outdated — a consumer can say nothing honest about it. It is the
Bundle's *content* version, distinct from `lkf_version` (§12), which is the
version of the format grammar.

`description` is not declared here: it is a core field already (§5.1), and
inheritance is add-only (§10.3), so a type may not restate an inherited field to
strengthen its obligation. A Bundle SHOULD still carry one.
