# Luma Knowledge Format (LKF)

**A contract in plain files, read the same way by anyone or any tool.**<br>
Standardize what matters. Extend the rest however you like.

## Overview

LKF is an open markdown format for representing knowledge, designed to be equally friendly to humans and agents. Its core is small by design — flexible and made to be extended — so it adapts to whatever you need to capture.

Knowledge lives in plain files with YAML frontmatter at the top, which you can create and maintain however you like. LKF is built to share knowledge across teams, tools, or organizations, and it is designed to require minimal tooling.

> **Status: unstable.** Breaking changes may ship in a patch release until `v0.1.0`, which is a claim the format has been run for real in more than one place and has not earned yet. Expect field renames. **This file deliberately states no version number** — `SPEC.md` is authoritative, and the current release is whatever the newest tag says. See [Releases](https://github.com/LumaStack/luma-knowledge-format/releases).

## Example

```markdown
---
type: document
title: Diffusion Models
tags: [diffusion, model, ml/generative]
modified: { by: human:fsmith, at: 2026-08-01T10:00:00Z }
sources:
  - resource: https://arxiv.org/abs/2006.11239
    title: Denoising Diffusion Probabilistic Models
    author: team:foobar
    last_modified: 2020-12-16
---

A class of generative models that learn to reverse a gradual noising process.
See [[score-based-models]] for the continuous-time formulation.
```

## Contents

**Using the format**

- [`explanation.md`](docs/explanation.md) — **start here.** How this works and why it matters.
- [`SPEC.md`](SPEC.md) — the specification. Short by design.
- [`CHANGELOG.md`](CHANGELOG.md) — what changed between versions, and why.

**Working on the format**

- [`principles.md`](docs/principles.md) — the design principles behind LKF.
- [`roadmap.md`](docs/roadmap.md) — open questions and deferred features.
- [`guidelines.md`](docs/guidelines.md) — how the project is run and released. Read it before cutting a release.

## License

Licensed under the [Apache License, Version 2.0](LICENSE).
