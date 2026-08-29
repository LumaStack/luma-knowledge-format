# Changelog

Notable **specification** changes to the Luma Knowledge Format, newest first. The point of this file: see what changed *between versions* at a glance, without reading every commit. It records behavior-affecting changes and omits edits that don't change behavior (wording, typos, formatting, examples).

Format follows [Keep a Changelog](https://keepachangelog.com); versions follow [semver](https://semver.org). See [`GUIDELINES.md`](docs/GUIDELINES.md) for how this file is maintained.

## [Unreleased]

### Changed
- **The lifecycle ladder measures what a reader is owed when a Document changes.** The specification said what each stage meant and never said what moved a Document between them, so in practice things got promoted for being used.
  **Audience promotes, not use.** An author exercising their own draft is testing it — that is what a draft is for — and the heaviest possible use by the people who wrote it says nothing about whether anybody else should rely on it. A Document can be published, adopted and used daily by its authors while honestly remaining `draft`. **`provisional` begins when somebody else can rely on it**, at which point the question stops being *is this working for us* and becomes *is this safe for them*. **Availability makes the question live and does not answer it** — nothing promotes itself; ask the author and take their answer, including *no, still a draft*.
  **The stages are now defined by obligation on change**, escalating: none, then notice, then a migration path. `draft` may reverse direction with nobody told; `provisional` may reverse direction and you will hear about it; `stable` may still change direction and owes you a way across. **`stable` does not mean frozen** — it means change carries a duty, which is the reading most people arrive without.
  **A migration path means something different for prose than for an interface.** For an API it is a shim or an overlap; for a Document it may be that the old term still resolves or that a record names what replaced it. The obligation is that somebody thought about the reader, not that a particular mechanism was used.
  **What the ladder deliberately does not say:** how correct the Document is — a buggy `stable` Document is a bug report while a `stable` Document that silently changes shape is a broken promise, and only the second is this field's business; whether it will still exist, which is `survival`; and whether anybody has checked it, which is `verified`.
  **`stable` keeps its name.** `mature` was considered and rejected — it describes the artifact's age and development, which is the producer axis this change moves away from, and a young Document can be fully supported while an old one shifts weekly. **`supported` names the obligation more exactly and bleeds into `survival`**; `ratified` is implied by `provisional`'s own wording. Both are recorded as deferred rather than taken, and renaming a released enum value would break every existing Document for a marginal gain.

- **`lifecycle_status` and `survival` are stated to answer different events.** They look like they overlap and they do not: **`lifecycle_status` governs the shape changing** and `stable` owes a path across; **`survival` governs the thing ending** and `promised` owes a process while `intended` owes nothing.
  **So `stable` with `survival: intended` is precise rather than contradictory** — the shape will not move under you, and nobody promised it will be here next year, which describes most well-run software. **A matrix now gives all nine combinations a plain reading**, because two orthogonal fields are only usable if a reader can see what each pair means without deriving it. **The corners prove the independence**: `stable` with `survival: experimental` is rock solid and may vanish; **`draft` with `survival: promised` is *I am committing to solve this and do not yet know what the answer looks like*** — a real commitment over an unsettled shape, which is the ordinary state of anything hard being taken seriously.
  **That second pair sharpens what `promised` commits to.** It is not *this shape will persist*, which the lifecycle field already governs. **It is that something will be here answering this**, whatever form it takes: a `promised` `draft` may be rewritten beyond recognition without breaking the promise, and withdrawing it with nothing in its place is what would break it.
  **Demoting `promised` to `intended` is itself the announcement.** The promise was never *forever*; it is *you will see this field change before anything happens, and that change begins the process.* It makes the commitment observable, and it means a consumer should re-read the field rather than reading it once.

### Added
- **`survival` — a core field for how much you should expect a Document to last**. `experimental | intended | promised`, optional, default `intended`.
  **It exists so a reader knows how much to lean on something.** Nothing said how much to expect a thing to still be here. A reader could learn how settled a Document was and how much its shape might change, and could not learn the simplest thing: is anybody keeping this, or is it out in the world to find out whether it earns its keep?
  **The question is deliberately the easy one.** Not *how long will this last*, which nobody can answer, and not *will it continue indefinitely*, which is harder still. Just: does it survive? `experimental` says there are no intentions and many experiments do not; `intended` says it is meant to stick around and nothing is promised; `promised` says withdrawing it is an event rather than an edit.
  **`intended` is the default and a producer may leave it unwritten**, which is the point — a value honest for nineteen things in twenty is one nobody should have to write nineteen times. Declaring it is equally valid. **Where the field tends to earn its keep is at the two ends**: a warning, or an undertaking. That is an observation about how the values fall, not a rule — the specification does not say when a producer should declare a field.
  **It has a default because future intent is frequently and legitimately undecided**, and *we mean to keep this and have promised nothing* is the honest common answer. A default that states what a reader already infers from silence costs nothing and hides nothing.
  **The name was chosen for how it degrades.** A neglected Document saying `intended` is out of date rather than untrue — where `maintained` or `supported` would make a false claim to everyone reading it, and neglect is exactly the state nobody returns to correct.
  **Neither it nor `lifecycle_status` is an input to the other**, and each is useful alone — a Document may declare either without the other. They are orthogonal in the sense *`verified` and trust tiers* already gives the word for trust: a Document can be `stable` and not expected to last, or `provisional` with its survival `promised`. Both corners are populated, which is what makes these two axes rather than one ladder.
  **Being used does not change it.** Somebody relying on an experiment took a risk they were warned about, and their use creates no undertaking nobody gave — only the publisher moves this value.
  *Migration:* **none.** The field is optional and its default is the common case, so every existing Document is already correct without being touched.

### Changed
- **`entry_point` is now `entrypoint`** *(breaking)*. One word, so that the same word names the same thing at every level it appears — a Bundle says where to start reading, and a consumer wanting the same idea one level up or down should not have to learn a second spelling for it.
  **The underscore was carrying a distinction that does not exist.** Every other multi-word field here names two different things joined — `field_presence`, `on_violation`, `lkf_version`. *Entrypoint* is one idea with one name, and writing it as two words invited exactly the collision it caused: a consumer building a project-level entrypoint read `entry_point` as *a different concept that happens to share a word*.
  *Migration:* rename the key in every `BUNDLE.md` that declares one. Nothing else changes — same value, same meaning, same `optional` presence.
- ***Released names* records `entry_point` as released.** The name is free; nothing reserves it.

## [0.0.16] — 2026-08-26

### Added
- ***Released names*, Released names** — one list of every name this specification once defined and no longer does: `preload`, `compliance`, `applies_to`, `index.md`, `concept`. Each says what it was, when it was released, and what replaced it.
  **The same facts were already here, in four phrasings and one absence.** Three were recorded only in the prose of the section that removed them — `preload` names itself in its sentence, `applies_to` and `index.md` say only *"The name is released"* with the name a paragraph earlier — and `compliance` appeared nowhere, because it was invented and withdrawn in the estate without ever reaching the specification. **A reader asking whether `preload` is still a thing had to find three paragraphs and know about a fourth name that was missing.**
  **It is also the list a tool can read.** A retired name in a published Document's *prose* is usually a rule still instructing authors to declare something nothing reads. *Frontmatter layout and conformance* is untouched: a consumer MAY report it and MUST NOT reject the Document for it.
  It records one rule that was implicit: **re-reserving a released name is a breaking change** even though nothing uses it, because a Bundle that adopted the free name would silently acquire the specification's meaning.
  *Migration:* none. Nothing changes about any Document.


## [0.0.15] — 2026-08-26

### Changed
- **The version no longer appears in `README.md`.** It lived in three files; it now lives in two, `SPEC.md` and `bundle/BUNDLE.md`, both of which are parsed by something. The README says *unstable* and points at the specification and the tags.
  **It went stale three times** — it sat at `v0.0.9` through two releases, and `v0.0.14` was tagged while it still said `v0.0.13`. Each time the answer was a better check rather than one fewer place to forget. **A number nothing parses is not worth duplicating**, and a reader who wants the current version gets it from the newest release, which cannot go stale because it is not written down.

### Fixed
- **`README.md` said `v0.0.13`.** It is the third place the version lives — the one a reader sees before deciding to adopt — and it was missed at `0.0.14`, which therefore shipped and was tagged claiming a version its own README contradicted. The repository has a check for exactly this; it could not run, because Actions was in a major outage while that release was cut. Corrected here rather than by retagging: `0.0.14` stood for under an hour and `0.0.15` supersedes it.

### Removed
- **`applies_to` is gone** *(breaking)*. `0.0.14` renamed it to `matches` and kept the old name readable so a migration could finish without dropping anybody's triggers. The migration finished the same day, so the fallback is now a second spelling every reader and every consumer has to know about, in exchange for compatibility with nobody.
  **Deprecation is a cost paid for adopters who exist.** This format has none outside the estate that produced it, and carrying a dead name this early buys noise rather than safety. Removing it while the count is zero is free; removing it later would not be.
  *Migration:* rename `applies_to` to `matches`. Consumers stop reading the old name, so a Document still declaring it surfaces nothing — which is the safe direction and is reported rather than silent.


## [0.0.14] — 2026-08-26

### Changed
- **`applies_to` is renamed to `matches`** *(deprecation, not yet breaking)*. `applies_to` is still read where `matches` is absent, and a consumer SHOULD report each use so a migration can be finished rather than assumed. Removal is scheduled and will be its own entry.
  **The old name obliged an author to write a false sentence.** `applies_to: everything` claims a Document governs everything, and none does — what a rule governs is stated in its body, and no frontmatter value widens or narrows it. The field says what makes a Document **surface**, which is a smaller and honest claim, and `matches` reads as a sentence in every form the field takes: *matches `git commit`*, *matches always*, *matches nothing*.
  **The vocabulary had also outgrown the name.** `path` is a target, but `event` is a *moment*, and nothing about a moment is a resource a rule scopes over — so the enforcement-scope convention that chose `applies_to` stopped applying once `event` joined the list.
  *Migration:* rename the field. Nothing else about it changes.

- **`always` leaves the trigger vocabulary and becomes a value of the field** *(breaking)*. `matches: always`, never an entry inside the list.
  As a list member it could sit beside a condition it silently rendered dead — `[always, path: "src/**"]` parses, validates, and ignores the path under OR semantics. It was never a peer of the other kinds either: **every kind narrows, and `always` refuses to.** Making the invalid combination unwritable is cheaper than a rule forbidding it.
  *Migration:* nothing in any known Bundle declares it. The spellings meaning *unconditionally* were silently discarded by consumers, and the one that parsed produced a trigger classing the Document as cheap — so a rule declaring itself ever-present was the one rule that would not be there.

### Added
- **`matches: nothing`, and the absence of `matches` means it** *(breaking — the default reverses)*. A Document declaring nothing is one that nothing surfaces on its own behalf. Consumers MUST NOT read the omission as a claim to be delivered unconditionally, which is what the previous default implied for `policy`.
  **The expensive reading is now the one that has to be asked for.** A Document acquiring a permanent claim on a reader's attention because an author forgot a field is the costliest available default, and it fails in the unrecoverable direction — under-delivering is recoverable, over-delivering is a token bomb.
  *Migration:* a `policy` that genuinely should always be present declares `matches: always`. No Bundle in the universal catalog is affected — all thirty-two of its policies already state what matches them.

- **`field_type: list_or_keyword`**, for a field taking either a list or one of a closed set of keywords. `field_type` is an open vocabulary, so this names a shape rather than adding a mechanism.

## [0.0.13] — 2026-08-25

### Changed
- **`applies_to` is no longer a core field** *(breaking)*. `v0.0.12` put it on the `document` root, which put it on every Document. It is now declared by **`policy` and `workflow` only**, and by nothing else.
  **The two kinds that carry a trigger are the two that act on a consumer** — a rule that binds, and a procedure that is run. Background does not act; it is reached *through* the things that do. Rationale has no moment either: it is wanted when somebody wonders *why*, and wondering is not a trigger. Where a concept's subject genuinely arises somewhere, the policy or workflow that arises there is already advertised at that moment, and announcing the rationale alongside makes it compete with them for attention.
  The section that described it goes with it — the field was never a property of every Document, so describing it inside **Field presence** was the wrong home. It now sits beside the built-in types that declare it.
  *Migration:* a Document other than a `policy` or `workflow` carrying `applies_to` is still conformant and no consumer is obliged to read it. Remove it, or move the claim to the rule or procedure the subject actually belongs to.

- **`workflow` declares `applies_to` and not `on_violation`.** A policy can be broken, at a moment, by an action somebody takes, and something can act on that. The only way to fail a workflow is **not to run it** — the absence of an action, and detecting absence needs state no consumer is obliged to keep.
  **The gap this leaves is real rather than an oversight:** there is no way to say *you must run this when that moment arrives*. A workflow's body may say so, and nothing mechanical follows.

## [0.0.12] — 2026-08-25

> **The version header ran ahead of the releases.** While this work sat unreleased
> on `main`, the header carried `0.0.12` and then `0.0.13` with no tag and no
> dated changelog section behind either. Neither was a release. **A version
> number is consumed by a release, not by a file briefly saying it** — so this is
> `0.0.12`, the successor to `0.0.11`, and no number is skipped.

### Removed
- **`preload` is gone** *(breaking)*. It was the one field here that described how a Document should be **consumed** rather than what it is — and consumption belongs to whatever distributes and loads Bundles, not to the format that defines them. `0.0.11` already found the seam, separating *what a consumer does with a Document* (the type) from *when it loads*; this finishes it by removing the second half rather than keeping a coarser answer than the question deserves.
  **It could only ever say `always`.** With three values on one axis and no way to express a condition, an author who meant *this matters, but only while you are doing X* had to round up. In the estate that produced this format, 26 Documents did exactly that. Given `applies_to`, none does.
  **The name is released** and is no longer reserved.
  *Migration:* replace it with `applies_to` where the Document's subject genuinely arises at particular times, and with nothing at all otherwise. A `policy` with no `applies_to` is one whose subject always arises, which is the honest reading of `preload: mandatory` on a rule.

### Added
- **`applies_to`, a core field** — when a Document's subject arises, as a list of single-key mappings drawn from a closed vocabulary: `always`, `path`, `tool`, `command`, `event`, `topic`.
  **Entries combine with OR and it is not an expression language.** No AND, no negation, no grouping — the moment those exist a consumer needs a parser. Glob syntax absorbs the common compound cases, and anything beyond that belongs in the body.
  **It is a property of the content, not an instruction to a consumer.** *This governs `git commit`* says what the Document is about; whether that means putting it in front of a reader at that moment is a decision a consumer **derives**. That distinction is why it belongs here where `preload` did not.
  `event` is the kind the others cannot reach: a lifecycle point, however it is arrived at. `command: git commit` catches that literal invocation, `event: before-commit` catches the point itself, and declaring both is redundancy in the useful direction.

- **`on_violation` on the built-in `policy` type** — what a consumer SHOULD do at the moment a policy is not complied with: `allow`, `audit`, `warn`, `require_reason`, `require_approval`, `block`.
  **Intent, never a guarantee**, like everything a type declares. A consumer that cannot intercept SHOULD say so rather than silently doing the nearest thing it can, because a policy that reads as enforced and is not is worse than one that never claimed to be.
  It is on `policy` and not on `workflow`: a rule can be broken at a moment, where failing to *run* a procedure is the absence of an action, and detecting that needs state no consumer is obliged to keep.

### Changed
- **`policy` declares no strength scale, deliberately.** A policy binds because it is a policy; how strongly is what its body says. A scale would restate the type on Documents that bind, and invite a soft tier for Documents that do not bind at all — which are `document`s wearing the wrong type. This is the type's existing position that *strength of obligation is a property of an individual policy*, now stated as a reason not to add a field.


### Changed
- **`obligation` is now `field_presence`, and `mandatory` is now `required`**. Two renames, one reason each.
  **`field_presence` says what the field grades.** `obligation` graded *how strongly a field should be present*, which is the JSON Schema question — and every schema language in use calls it `required`, not `obligation`. A reader arriving from any of them stopped to check whether the word meant something subtler. It does not.
  **`required` completes the ladder.** RFC 2119's adjectival set is `REQUIRED / RECOMMENDED / OPTIONAL`, and the scale was using two of those three with a synonym in the top rung. Anybody who has read a specification now recognises all of it at once. `deprecated` still sits beside the ladder rather than on it — it states a field's future rather than its strength, which is why *Inheritance* cannot reach it by strengthening.
  *Migration:* mechanical. Rename the key and the one value; nothing else about the scale moves, and no consumer behaviour changes, since presence never affected conformance.

- **`preload` is marked superseded, and retained**. The estate that produced this format no longer uses it: it conflated *when a Document arrives* with *how strongly its content binds*, which turned out to be two questions with different answers. The replacement is being run outside the specification first, because *Frontmatter layout and conformance*'s tolerance of unknown fields allows it and **a built-in field takes a word from everyone permanently**. The section moves when the replacement has earned it.


### Changed
- ***`created` and `modified`* now says why `actor_event` is nested** rather than a flat `created_by`/`created_at` pair. No behaviour change; the shape is unaltered and this records reasoning that was load-bearing and unwritten.
  Three grounds: **`verified` is a list of them**, so a flat form would be parallel arrays correlated by index — the positional failure *`sources`* avoids by keying footnotes; ***Actor convention*'s argument depends on the pair being atomic**, since omitting `by` *"discards the `at` timestamp sharing the same `actor_event`"*, which only holds if they are one object; and **one `field_type` declaration beats two fields plus an invariant no validator can see**.
  The cost is stated rather than waved away: the flat pair reads better in a diff and greps in one line.

## [0.0.11] — 2026-08-23

### Added
- **A Type Definition may carry its own `version`**, `semver`, optional. A Bundle's version answers the wrong question for a copied type: vendor one type out of a Bundle holding six and a bump caused by any of the other five reports your copy as out of date when it is byte-identical.
  **What a bump means is deliberately not defined yet.** Treat it as a label rather than a promise — compare for equality and difference, and do not infer compatibility from which tier changed. This also gives *Versioning*'s dangling reference to *"a Type Definition's own `version`"* something to point at.
  `vendored_from.version` records the type's own version where it declares one, and the containing Bundle's otherwise.
- **`unknown` on `lifecycle_status`, as the default** *(breaking)*. It is **not a stage** — it says the value was not filled in, so at read time nobody knows it. Whether the fact is lost or was never stated is not the field's business, which is what lets one word serve wherever it is needed; *Actor convention* already uses it that way for actors.
  **It is the default because both real defaults would be wrong guesses.** `provisional` makes a `draft` thing read as more settled than it is, and the other direction makes a `stable` thing read as less. Neither is safe — which is the `consumers` case *the section that described it* contrasts, not the `preload` one.
  **Absent and explicitly `unknown` mean the same thing**, so nothing is ambiguous; writing it is still worth doing, because silence cannot distinguish *considered and undecided* from *never thought about*. Not spelled `none`: `none`, `null` and `nil` are absence words in one language or another, and a value that looks like a null gets conflated with one.
  *Migration:* **no file changes.** Documents that declare a value keep it. Documents that do not now read as `unknown` rather than `provisional`, which is the point. A consumer branching on the enum gains a case.

### Changed
- **`workflow` and `policy` are defined by what a consumer *does* with a Document, not by when it arrives** (*Resolution and namespacing*, *the section that described it*, and both Type Definitions). A `workflow` you **run**, a `policy` **binds** you, a plain `document` you **read**.
  **This closes an overlap with `preload`.** `policy` was defined as *standing — kept present*, which is close enough to `preload: mandatory` that a Document could state both and contradict itself — and adopters had already written a *"`preload` and `type` must agree"* rule to patch around it. All three engagement modes were written in loading vocabulary, so two of them collided with a loading field.
  **On the new axis they are orthogonal.** The type says what the content is to you; `preload` says whether you have it. **A `policy` with `preload: optional` is now an ordinary thing** — a rule that binds when it applies and costs nothing until then — where before it read as a mistake.
  **A rule nobody loads still governs nothing**, and that is a reachability problem rather than a definitional one: the answer is something always present naming the rules that *exist*, not forcing every rule into context.
  *Migration:* **none.** No field changes, no frontmatter changes, and no Document is affected. This is what the two types have always meant, said on the correct axis.
- **The concept migration note now covers `extends: concept`** (0.0.10). It said only to replace `type: concept`, while 0.0.6 had explicitly promised *"`extends: concept` keeps working unchanged"* — so anyone who read that had it in their Type Definitions. Three downstream definitions did.

## [0.0.10] — 2026-08-22

### Removed
- **`concept` is removed as a built-in type** *(breaking).* It extended `document`, declared no fields, and no consumer ever treated it differently from the root — which *Resolution and namespacing*'s own test calls *falsified rather than merely unused*. It was marked *under review* in `0.0.9`; this finishes the job.
  **What kept it was its retrieval mode, and that did not need a type.** *Retrieved when relevant* is what a plain `document` already is, so *Resolution and namespacing* now describes **two** base types rather than three: `workflow` is invoked, `policy` stands, and the third way is the root itself. **A type that names the default dispatches on nothing** — every consumer already treats anything without a more specific type exactly that way. The set is still closed: a further base type would have to name a way of engaging that is neither invoked, nor standing, nor the default.
  **The name is released rather than reserved.** `ROADMAP.md` had held the type on the grounds that removing a name and re-adding it later collides with every Bundle that defined it privately in between. That cost is real and was accepted, because the deferral had begun costing more: *under review* does not stop adoption, and five Documents across two published Bundles had declared it — none for a reason `document` could not serve. A name that is noise stops being harmless once people use it.
  *Migration:* two forms, and the second is easy to miss.
  **`type: concept` becomes `type: document`.** Nothing else changes — the two were structurally identical, which is why the type went.
  **`extends: concept` in a Type Definition becomes `extends: document`.** `0.0.6` said outright that *"`extends: concept` keeps working unchanged"*, so anyone who read that has it in their type definitions; this release retracts it. **The contract is unaffected either way**: `concept` extended `document` and added nothing, so a type that inherited through it inherits exactly the same fields directly. Dropping the line entirely is equally correct, since every type implicitly extends `document`.
  A Bundle that genuinely dispatches on the distinction may define `concept` in its own `_types/`, subject to the usual bar in *Resolution and namespacing*: name the consumer, and name what it does differently.

### Added
- **`vendored_from` on a Type Definition** — `resource`, `version` and `at`, recording where a copied type came from. *Resolution and namespacing* makes vendoring the only way to share a type, and a vendored copy was previously anonymous: nothing could tell a current copy from a stale or edited one.
  **It is provenance, never a lookup.** Nothing fetches it, and a consumer that cannot reach `resource` reads the Document exactly as before — the contract is the local file and always was. This is deliberately not a search path, an environment variable, or a locator embedded in `type:`; each of those makes a Document's meaning depend on where it is read rather than on what it says.
  **It answers two questions, and the second is easy to miss.** *Is my copy current?* — compare against `resource` at `version`, on demand. And *do two copies in one place agree?* — two Bundles that vendored one type at different versions hold two contracts under one name, which the resolution scope permits and nothing else would surface.

### Changed
- **Self-containment is now defined as a property of lookup, not of relationships**. Previously *"nothing it needs lives outside it"*, which a Bundle naming another Bundle would falsify — and which said nothing about the thing actually at stake. It now reads: **nothing is fetched in order to read it**, so a Bundle may be moved, archived or copied whole and still be read offline.
  The distinction matters because the old wording defended the wrong perimeter. What must never happen is a *lookup* while reading — a remote fetch, a search up the directory tree, a path from the environment. What is harmless is a Bundle *naming* another it expects alongside it, which is a claim about what should be adopted.
- **The bar for a built-in type is sharpened, and it is a narrowing**. Four additions, no removals:
  **A built-in is the format's only mandatory surface.** Everything else is permissive by law — unknown types tolerated, missing fields tolerated, nothing rejected. So the question is never whether a type is *important*; it is whether a consumer that ignored it would fail to read a conformant Document, or engage with one in a way this specification says is wrong. ***My tooling would break* is explicitly the wrong kind of broken** — true of every domain type ever written, and a consumer ignoring one still reads the Document correctly as a plain `document`.
  **The cost of a built-in is a word taken from everyone, permanently**, which is why *important to us* is not an argument — **importance is what a namespace is for**. A cheap further check: **does it change at the format's rate?** A type that gains fields as somebody's tooling matures would drag the format's releases behind that roadmap.
  And the removal-is-cheaper-than-addition asymmetry is now **a tiebreaker rather than an entry route** — it applies to a candidate that already cleared the checks and is still balanced, not to one that failed them.
- **Namespacing is stated as a convention rather than a suggestion**. **Unprefixed means the format defines it; a prefix means somebody else does** — so a reader can tell a type's origin from its name without a lookup. A `type` published beyond the Bundle that wrote it SHOULD be namespaced.
- **The Bundle is named as the resolution scope, with its consequence**. Because a contract is found in *this* Bundle's `_types/`, **two Bundles may hold different versions of one type without contradiction** — each Bundle's Documents are checked against the copy that travelled with them. That is the scoping mechanism prose lacks, and it is why vendoring a type is safe where duplicating a policy would not be.
  **The exception is now stated too:** a Document living outside every Bundle has no such scope, the format offers no rule for where its contract is found, and whoever puts a Document there owes it an answer.
- ***Discovery* no longer names a particular vendor's command.** Discovery is reading `_types/<type>.md`; tooling may wrap that and needs no index to do it.
- **A subtype may strengthen an inherited field's obligation** — `optional` → `recommended` → `required`, by redeclaring the field with a higher `field_presence` and nothing else changed. Weakening stays forbidden, and where a field is declared at several points in a chain the strongest obligation wins.
  **The gap it closes: a type whose semantics rest on inherited fields could not state them.** Where a type's growth stage *is* `lifecycle_status` and its age *is* `created` — both `optional` on the root — add-only left no way to say a Document missing either is incomplete. The type's own contract called unremarkable exactly the omissions that break it, and the workaround in practice was a sentence of prose telling readers to treat a field as required despite what the contract said.
  **Consistent with add-only, because presence is not meaning.** That rule exists to keep an inherited field's meaning stable everywhere it is used; the field still means what its declaring type said, and a subtype only states how strongly *it* expects it. Removing a field, or changing its `field_type`, `values` or meaning, is still forbidden — *Inheritance* now says so in those terms rather than by the broader word *redefine*.
  **Nothing becomes non-conformant.** Obligation describes intent and the sole hard requirement is still a non-empty `type`, so a consumer that knows only the parent type and one that knows the subtype may disagree about whether a file is complete, and each is right at its own level.
  **`deprecated` is not on the ladder** and is not reachable this way — it states something about a field's future rather than its strength, so a type inheriting a deprecated field may not mandate it.
  *Migration:* none. This permits what was previously forbidden, so every existing Type Definition and Document stays valid. Validators gain a rule; documents do not change.

## [0.0.9] — 2026-08-19

### Added
- **`workflow` and `policy` as built-in types** — both field-free, both extending `document`. A `workflow` is a procedure that gets *transformed*: consumers project it into whatever form their harness expects, selected by the type rather than by where the file sits. A `policy` is a course of action that a consumer keeps as *standing context*, where a workflow is loaded on invocation — the type is what tells otherwise-identical prose apart.
  **They complete a three-way partition rather than adding two labels.** `concept` is *retrieved when relevant*, `workflow` is *invoked*, `policy` is *standing* — three ways prose reaches a consumer, and the set is closed at three because a fourth would have to name a fourth way of engaging rather than a fourth subject.
  `policy` must be built in rather than defined per Bundle for a specific reason: tooling that makes a policy hard to ignore can only be written against a name the format guarantees. A type each Bundle defines privately is one no consumer can rely on finding. Its dispatch difference is *intended* rather than shipped, which *Resolution and namespacing* permits provided the consumer and the difference are named — and if it never acquires them it has been falsified, not merely unused.
  Corroborating count, offered as corroboration only: across the first seven Bundles written against this format, `workflow` was defined independently in 7 of 7 and `policy` in 6 of 7 — thirteen byte-identical files. **That sample is biased** and the spec now says so: all seven are governance Bundles, so finding governance vocabulary throughout them proves less than it appears to.
- **The test for declaring a type at all** — *name the consumer, and name what it does differently.* If you cannot name both, the `type` is a label, and a label costs a name every other Bundle must avoid. Three forms are given: **checked** (a validator has a contract), **transformed** (something converts it into another form), **consulted** (the format's own machinery reads it to handle other Documents). Wanting to *enumerate* Documents of a kind is explicitly not enough, since that works for any type and would grow the list without limit.
- **The further test for being built in** — earning a name is not enough; a built-in must also be **unavoidable**, either *structural* (the format's machinery depends on it) or *ubiquitous* (nearly every Bundle defines it independently, counted rather than asserted — **and then check what you counted**, since a sample drawn from one domain finds that domain's vocabulary everywhere). Records that removal is cheaper than late addition, and that the asymmetry favours admitting a balanced case now.

### Deprecated
- **`concept` is marked *under review*** and should be a deliberate choice rather than a default. It has existed since `v0.0.1` with no consumer treating it differently from `document`, which *Resolution and namespacing*'s own test calls *falsified rather than merely unused*. Its retrieval mode — pulled in when relevant — is the case for keeping it, and that case is a claim nothing yet implements. **What would settle it is a durable knowledge base**, the thing `concept` was written for and which nobody has built with this format. Held rather than removed, because a removed name re-added later collides with every Bundle that defined it privately in between.

## [0.0.8] — 2026-08-18

### Added
- **`preload` as a core field** — `optional`, an enum of `mandatory | recommended | optional`, saying how strongly a Document should be loaded before working with its Bundle. A Bundle is usually larger than any one task needs, and nothing let a Document say it was the spine rather than reference material.
  It sits on the root rather than on `bundle` because any Document may carry it. **`required` is a hard requirement**: a consumer that cannot load such a Document fails and names it, rather than starting diminished — a level that degrades quietly is a hint, and hints are ignored. The cost falls on authors, which is what keeps the level meaning anything.
  Absent means `optional`, and *the section that described it* states why that is a genuine default here while absence of `consumers` means nothing: the weakest value is also the safe one.
- **`entry_point` on the built-in `bundle` type** — `optional`, a Document ID naming where a reader should start. Without it every consumer invents its own answer — first alphabetically, longest file, name matching the directory. It is deliberately distinct from `preload: mandatory`: entry point is reading order, `preload` is context presence, and a Bundle may need several Documents loaded while still having one place to begin.
  Both are additive and non-breaking; existing Bundles remain valid unchanged.

## [0.0.7] — 2026-08-18

### Changed
- **`applies_to` on the built-in `bundle` type is renamed `consumers`**. Same obligation, same field type, same open vocabulary — only the name changes. Two reasons, both structural rather than stylistic. Every other field on `bundle` states a *property* of the Bundle (`version`, `published`, `description`) while `applies_to` stated a *relation*, and in policy languages `applies_to` conventionally means enforcement scope — "this rule applies to these targets" — whereas this field is about eligibility. `consumers` also matches how the format names every other collection: `tags` holds tags, `sources` holds sources.
  The spec now says outright that the values are **kinds, never instances** — the one ambiguity a plural noun introduces, and cheaper to close in a sentence than to name around.
  *Migration:* rename the key. Nothing published declares `applies_to`, so in practice there is nothing to migrate — the field existed for a matter of hours.
  Breaking, shipped as a patch under the pre-1.0 clause (`v0.0.z` is explicitly unstable). No deprecation cycle, because deprecating a field with no users would leave the specification carrying a dead name to protect nobody.

## [0.0.6] — 2026-08-18

### Added
- **`applies_to` on the built-in `bundle` type** — `optional`, `list of text`, naming the kinds of consumer that may adopt a Bundle. **The vocabulary is open and LKF defines no values for it.** Where a distribution model has more than one kind of consumer — a repository and an organization, a workstation and a server — a Bundle may say which of them it is for. The format does not know what those kinds are, so the values belong to whoever is distributing, exactly as `tags` is `list of text` and left loose.
  It is a list because a Bundle may legitimately apply at more than one, and that is precisely why it has to be a field. The first consumer had sorted Bundles into directories by consumer kind; a directory can express only one, so it forced the *publisher* to answer a question that often belongs to the *adopter*, permanently and with no override. Absence says nothing — not "none", not "all" — and no consumer rejects a Bundle for it.
  Additive and non-breaking: existing Bundles remain valid unchanged.

## [0.0.5] — 2026-08-17

### Changed
- **The built-in types moved to a `bundle/` directory**. They previously sat at the repository root, which made the repository itself the Bundle — and therefore made the "unit of distribution" include the changelog, the guidelines and the roadmap. Project apparatus is not knowledge, and a Bundle that drags its own governance along is not self-contained in any useful sense. `bundle/` is now exactly the unit: `BUNDLE.md` and `_types/`.
  `SPEC.md` deliberately stays at the repository root. It is the primary artifact and the first thing a reader looks for, and burying it to tidy the secondary files would be the wrong trade. This Bundle is the built-in types; a Bundle carrying the specification as loadable knowledge would be a different one.
  *Migration:* if you vendored built-in Type Definitions from this repository, they are now under `bundle/_types/` rather than `_types/`. The file contents are unchanged.

## [0.0.4] — 2026-08-17

### Added
- **The built-in types ship as real Type Definitions** in this repository's `_types/` — `document`, `concept`, `bundle` and `type_definition`. They were previously prose tables only, which meant the format claimed to be self-hosting while its own types were the one thing not expressed in it. The repository is now itself a Bundle: the Type Definitions are its Documents, and the specification, readme, licence and `docs/` are its Assets — legal only since Assets exist. A tool MAY supply the built-ins itself rather than requiring every bundle to vendor them.
- **Assets**. A Bundle's files are now partitioned: every file is either a Document (frontmatter with a `type`) or an **Asset** (no frontmatter, no type, outside Type Definition validation). An **Attachment** is an Asset a Document links to — a relationship rather than a category, so the same Asset may be an Attachment of several Documents, and one nothing links to is nobody's Attachment. Scripts, templates and binaries previously had no standing in a Bundle at all.
- **Asset links**. `[[…]]` links a Document; `[…](…)` links anything else. No new syntax, no change to ID or slug rules, and the two forms are distinguishable on sight. An Asset link MUST point inside the Bundle — reaching outside breaks self-containment — though as with Document links, an unresolved one stays legal.
- **`bundle` as a built-in type, and `BUNDLE.md`**. A Bundle describes itself in a root `BUNDLE.md` with `type: bundle`, carrying `version` (mandatory, `semver`), `published` and `description` (both recommended). `version` is mandatory because a Bundle without one cannot be pinned, compared, or reported as outdated — a consumer can say nothing honest about it.
- **`semver` field type**. A version string as [semver.org](https://semver.org) defines it, including pre-release and build metadata (`1.0.0-alpha.1+build.5`). The `v` prefix is excluded deliberately — `v1.2.3` is a tag convention rather than a version, and accepting both spellings would mean two bundles could write the same version two ways and every consumer would have to normalize.

### Changed
- **`lkf_version` moved from the root `index.md` to `BUNDLE.md`**. *Reserved files* defines `index.md` as derived navigation — "a rebuildable cache, not a source of truth" — so the format-grammar version was living on a file any tool is entitled to regenerate, and a navigation rebuild could discard it. `BUNDLE.md` is a source of truth.
  *Migration:* if a Bundle declares `lkf_version` on its root `index.md`, move it to `BUNDLE.md`. Bundles that never declared one are unaffected.

- **BREAKING: the base object is a Document, not a Concept** (*Terminology*, *Document ID*, and throughout). `Concept` named the abstraction after one of its instances: a `task` is not a concept, nor is a `lab_result`, and the definition had to stretch to "a tangible asset, an abstract idea, or anything in between" to cover them. The spec's own prose already said the right word, describing a bundle as "a collection of knowledge documents". `document` is now the root type every type implicitly extends, and `concept` becomes an ordinary built-in type for knowledge-base entries, adding no fields of its own.

  *Migration:* **No file has to change.** "Concept" was the specification's word for the object, never a value anyone wrote in frontmatter — existing files are already conformant Documents. Three things to update, all outside your content: rename `Concept ID` to `Document ID` in tooling and prose; where your documentation says "Concept" meaning *any LKF file*, it now says Document; and where it means *a knowledge-base entry specifically*, that is now the `concept` type. `extends: concept` keeps working unchanged — `concept` extends `document` and adds nothing, so anything that inherited the core fields through it still does.

- **Redefining a built-in type is now `SHOULD NOT` rather than `MUST NOT`**. The prohibition was the format's only hard "you may not", sitting inside a specification that is otherwise permissive by default and never rejects. It also over-applied: redefining `type_definition` is genuinely dangerous, while redefining the root type to *add* a field is useful and has no other mechanism, since *Inheritance* lets a type add fields but nothing lets you add to the root. One blanket rule forbade the useful case to prevent the harmful one. It is now discouraged rather than forbidden. Not a migration — this permits strictly more than before. If it later needs teeth, the shape is already in the format: *Inheritance* is add-only, so the rule would be that a redefinition may *add* to a built-in but never contradict it.
  The heading changed from "Reserved built-ins" to "Built-in names", freeing *reserved* for a possible future mechanism that reserves type names on behalf of others.

## [0.0.3] — 2026-08-15

### Added
- **`unknown` as an actor kind and as a producer**. `unknown:unknown` records that the actor was not captured; `agent:unknown` records the kind without the identity. The grammar previously had no way to say *not recorded*, so a tool that could not tell who invoked it had to guess a plausible actor or omit `by` — and omitting it discards the `at` timestamp in the same `actor_event`. **Supported but an anti-pattern:** a tool that can identify its actor should, and `unknown:unknown` is for genuine ignorance rather than a default for tools that never asked.

## [0.0.2] — 2026-08-02

### Changed
- **Field type `concept-link` → `wikilink`** *(breaking).* The internal-link field type is now type-agnostic — it links to any Concept, not only `concept`-typed ones — and named by form to pair with `uri`.
  - *Migration:* replace `field_type: concept-link` with `field_type: wikilink` in Type Definitions.
- **Composite field type `actor-event` → `actor_event`** *(breaking).* snake_case, per the identifier-casing convention.
  - *Migration:* replace `actor-event` with `actor_event` in Type Definitions.
- **Reserved type `type-definition` → `type_definition`** *(breaking).* Same casing convention; the `_types/` directory and the reserved-name rule are otherwise unchanged.
  - *Migration:* rename `type: type-definition` to `type: type_definition`.
- **Identifier casing standardized.** Field names, `type` names, and `field_type` values prefer snake_case; Concept slugs and IDs prefer kebab-case (path/URI-like). A recommendation, not a hard rule.

## [0.0.1] — 2026-08-02

Initial release.

### Added
- **Core model** — files as Concepts, path-based identity, and permissive conformance (a non-empty `type` is the only hard requirement; consumers never reject).
- **Core fields** — `type`, `title`, `description`, `tags`, `lifecycle_status`, `created`, `modified`, `verified`, `sources`, `stale_after`.
- **Provenance & trust** — `created`/`modified` (author + timestamp), `verified` with derived trust tiers, structured `sources`, and the actor convention `<kind>:<producer>/<version>`.
- **Type extensions** — Type Definitions in `_types/`, the field-type vocabulary, field `field_presence` (`required`/`recommended`/`optional`/`deprecated`), single/add-only inheritance, vendored resolution, and validation as a *suggested framework — not a contract*.

[Unreleased]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.16...HEAD
[0.0.16]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.15...v0.0.16
[0.0.15]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.14...v0.0.15
[0.0.14]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.13...v0.0.14
[0.0.13]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.12...v0.0.13
[0.0.12]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.11...v0.0.12
[0.0.11]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.10...v0.0.11
[0.0.10]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.9...v0.0.10
[0.0.9]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.8...v0.0.9
[0.0.8]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.7...v0.0.8
[0.0.7]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.6...v0.0.7
[0.0.6]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.5...v0.0.6
[0.0.5]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.4...v0.0.5
[0.0.4]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.3...v0.0.4
[0.0.3]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.2...v0.0.3
[0.0.2]: https://github.com/LumaStack/luma-knowledge-format/compare/v0.0.1...v0.0.2
[0.0.1]: https://github.com/LumaStack/luma-knowledge-format/releases/tag/v0.0.1
