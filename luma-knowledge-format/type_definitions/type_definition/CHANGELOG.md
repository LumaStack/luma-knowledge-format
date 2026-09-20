# Changelog — `type_definition`

The history of the `type_definition` Type Definition, newest first, keyed by the
type's own `version`. History from before it declared one is recorded in
the format's own
[`CHANGELOG.md`](https://github.com/LumaStack/luma-knowledge-format/blob/main/CHANGELOG.md).

## 0.1.0

- Versioning begins: the type declares its own `version`, independent of
  the format's `lkf_version`. Shipped with format release `0.0.21`, which
  moved every Type Definition to the folder shape — this definition now
  lives at `type_definitions/type_definition/DEFINITION.md`, with this changelog
  beside it.
- The contract gains the `version` field declaration the specification
  already documents (`optional`, `semver`), so the field every built-in now
  carries is one this type's own contract names.
