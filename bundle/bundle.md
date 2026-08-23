---
type: bundle
version: 0.0.10
description: The Luma Knowledge Format — its specification, and the built-in types as real Type Definitions.
---

# Luma Knowledge Format — built-in types

This directory is a Bundle: the built-in Type Definitions, expressed in the
format they define. It is what §10.4 means by vendoring — copy the
`_types/*.md` you want into your own bundle.

It sits in its own directory so that it *is* the unit of distribution. The
repository around it — the specification, the changelog, the project
documentation — maintains this Bundle rather than being part of it, and a
Bundle that dragged its own governance along would not be self-contained in any
useful sense.

The specification itself stays at the repository root. This Bundle is the
built-in types; a Bundle carrying the spec as loadable knowledge would be a
different one, with a different purpose.

`version` tracks the specification version and is bumped with it at release.
