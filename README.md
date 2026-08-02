# Luma Knowledge Format (LKF)

LKF is a format for representing knowledge, designed to be equally friendly to humans and agents. Its core is small by design — flexible and made to be extended — so it adapts to whatever you need to capture.

Knowledge lives in plain files with YAML frontmatter at the top, which you can create and maintain however you like. LKF is built to share knowledge across teams, systems, or organizations, and it is designed to require minimal tooling.

> **Status:** Draft — `v0.0.1-draft`. Not yet ratified; breaking changes expected. Versions are published as git tags (`v0.0.1`).

## Example

```markdown
---
type: concept
title: Diffusion Models
tags: [ml/generative]
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

- [`SPEC.md`](SPEC.md) — the specification.
- [`PRINCIPLES.md`](PRINCIPLES.md) — the design principles behind LKF.
- [`GUIDELINES.md`](GUIDELINES.md) — how the project is run.
- [`ROADMAP.md`](ROADMAP.md) — open questions and deferred features.
