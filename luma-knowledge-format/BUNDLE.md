---
type: bundle
version: 0.0.18
entrypoint: specification/lkf
description: The Luma Knowledge Format — its specification, and the built-in types as real Type Definitions.
---

# Luma Knowledge Format

This directory is a Bundle: the specification, and the built-in Type
Definitions expressed in the format they define. `entrypoint` names the
specification, which is where a reader starts.

It is what *Resolution and namespacing* means by vendoring — copy the
`_types/*.md` you want into your own bundle.

**The specification travels with the types because they are one claim.** The
built-in Type Definitions are a rendering of what the specification says, and a
consumer that has the types without the prose has the shape of the contract and
none of its meaning. Distributing them together is what makes the Bundle answer
a question rather than pose one.

**It sits in its own directory so that it *is* the unit of distribution.** The
repository around it — the changelog, the project documentation, the release
guidelines — maintains this Bundle rather than being part of it, and a Bundle
that dragged its own governance along would not be self-contained in any useful
sense.

**`specification/` is an ordinary directory, not a reserved one.** The format
claims no name there; a Bundle that carries a specification is a Bundle with a
Document in it, which is why the file is lowercase and typed `document` like
any other.

`version` tracks the specification version and is bumped with it at release.
