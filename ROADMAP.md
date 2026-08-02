# LKF Roadmap

Open questions and deferred features for the Luma Knowledge Format. `SPEC.md` describes only what is settled; this file tracks what is not.

## Undecided — needs a decision before it can be specified

- **Field-level ratification** — confirm the working-default levels in `SPEC.md` §5.1 (`title`, `description`, `tags`, `verified`, `sources`).
- **Type-extension rules** (§10) — property-type vocabulary, `extends`/inheritance and conflict resolution, tool-default vs. bundle precedence, validator severities.
- **Link resolution** — the algorithm and slug rules (uniqueness scope within a bundle, ambiguity handling). Reintroduce `aliases` here; alternate-name resolution is meaningless without the resolution rules.
- **Reserved-file formats** — the exact structure of `index.md` and `log.md` (§11).

## Deferred features — postponed, may return in a later version

- **Stable identifiers** — opaque ids decoupled from path; additive when added (links stay name-based).
- **`aliases`** — ships together with link resolution (above).
- **`confidence`** and **`volatility`** — trust/freshness fields considered but held.
- **Concept-level `owner`** — accountability/stewardship, distinct from a source's `author`.
- **Attestation** — verifying that a computed value was produced by a sanctioned method (OKF's "Attested Computation"). Out of scope for MVP and adoption is uncertain, but worth revisiting in a later version.
