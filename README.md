# Luma Knowledge Format (LKF)

LKF is a format for representing knowledge, designed to be equally friendly to humans and agents. Its core is small by design — flexible and made to be extended — so it adapts to whatever you need to capture.

Knowledge lives in plain files with YAML frontmatter at the top, which you can create and maintain however you like. LKF is built to share knowledge across teams, systems, or organizations, and it is designed to require minimal tooling.

> **Status: `v0.0.9` — unstable.** Breaking changes may ship in a patch release until `v0.1.0`, which is a claim the format has been run for real in more than one place and has not earned yet. Expect field renames. `SPEC.md` is authoritative for the version; releases are published as git tags — see [Releases](https://github.com/LumaStack/luma-knowledge-format/releases).

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

- [`EXPLANATION.md`](docs/EXPLANATION.md) — **start here.** How this works and why it matters.
- [`SPEC.md`](SPEC.md) — the specification. Short by design.
- [`CHANGELOG.md`](CHANGELOG.md) — what changed between versions, and why.

**Working on the format**

- [`PRINCIPLES.md`](docs/PRINCIPLES.md) — the design principles behind LKF.
- [`ROADMAP.md`](docs/ROADMAP.md) — open questions and deferred features.
- [`GUIDELINES.md`](docs/GUIDELINES.md) — how the project is run.

## License

Licensed under the [Apache License, Version 2.0](LICENSE).
