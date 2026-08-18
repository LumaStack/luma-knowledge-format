---
type: bundle
version: 0.0.4
description: The Luma Knowledge Format — its specification, and the built-in types as real Type Definitions.
---

# Luma Knowledge Format

This repository is itself a Bundle. Its Documents are the built-in Type
Definitions under `_types/`; its Assets are the specification, the readme, the
licence, and the project documentation under `docs/` — files with no
frontmatter, which is exactly what an Asset is (§2).

The format describing itself in its own terms is the point: `_types/` is a
normative rendering of the built-in types and a worked example at the same time.

`version` tracks the specification version and is bumped with it at release.
