# LKF Design Principles

The durable goals and values behind the Luma Knowledge Format (LKF). When a design question is unclear, it is settled by appeal to these — the specification bends to honor them, not the other way around.

LKF assumes a world where a body of knowledge is never simply written once and filed away: it is maintained and revised **continuously**, by people and agents working in parallel. The format's job is to make that kind of living corpus trustworthy while holding **as little opinion as it can**.

## Plain text, no lock-in

A knowledge base is just markdown files in a git repository — the files themselves are the whole system, and the authoritative source of truth. Any index, cache, or database is *derived* from them, and can be rebuilt or discarded without loss.

- **Readable by a human with no tools.** Open any file in any editor and it stands on its own.
- **Parseable by an agent with no special SDK.** It is plain markdown and YAML — nothing proprietary sits between an agent and the content.
- **Diffable in version control.** Every change lands as a clean, reviewable diff.
- **Portable.** No file is bound to a single tool, organization, or moment in time — which is what lets knowledge move freely across projects, systems, teams, and organizations.

Because the substrate is this plain, producers (people, agents, export pipelines) and consumers (agents, interfaces, search indexes, deterministic code) all work from the same files, with nothing to install between them.

## Trust is legible

A largely machine-written corpus is only useful if a reader can judge what they are looking at — from the file itself, without external context. LKF makes these answerable in-band:

- **Provenance** — where did this come from, and who checked it?
- **Trust** — how much should I rely on it?
- **Freshness** — is it still true?
- **Lifecycle** — is this the current version?

## Small core, open edge

LKF standardizes only the **small set of frontmatter conventions** that make a corpus self-describing and trustworthy — and prescribes no particular runtime or tooling beyond that. Everything domain-specific is left to producers and consumers.

That small core is paired with a **predictable pattern for extension**: any `type` can declare its own rules — the fields it requires, recommends, or allows — so a domain can specialize LKF without touching the core. The core stays small precisely so the edges can grow.

## Permissive by default

LKF tolerates imperfect input rather than rejecting it. A consumer MUST NOT refuse a file for a missing optional field, an unknown type, an extra key, or a link whose target doesn't exist — unknown things are preserved, and a broken link may simply be knowledge not yet written. Strictness is opt-in (a validator), never the default posture.

## Out of Scope

LKF does not:
- **Fix a taxonomy of Document types.** The `type` vocabulary is open — yours to define and extend.
- **Prescribe how a bundle is stored, served, or searched.** LKF describes files on disk, not the systems built around them.
- **Replace domain schemas.** Where a domain already defines one — a data model, an API contract — LKF points at it rather than absorbing or replacing it.
