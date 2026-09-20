# Changelog — `document`

The history of the `document` Type Definition, newest first, keyed by the
type's own `version`. History from before it declared one is recorded in
the format's own
[`CHANGELOG.md`](https://github.com/LumaStack/luma-knowledge-format/blob/main/CHANGELOG.md).

## 0.0.1

- Versioning begins: the type declares its own `version`, independent of
  the format's `lkf_version`. Shipped with format release `0.0.21`, which
  moved every Type Definition to the folder shape — this definition now
  lives at `type_definitions/document/DEFINITION.md`, with this changelog
  beside it.
- The contract gains `type_version` (`recommended`, `semver`): every
  Document may record which version of its type's Type Definition it was
  written against, so migration becomes a query rather than a belief. On a
  Type Definition the field tracks the `type_definition` contract itself.
