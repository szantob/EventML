# Changelog

All notable changes to this project are documented in this file.
The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

`eventml-core` (the metamodel) and `eventml-lib` (the domain vocabulary) are
versioned separately. Entries state which one changed.

## [Unreleased]

Nothing yet. Work landing after v0.3.0 is recorded here until it is released.

## [0.3.0] - 2026-08-10

Makes a library a named, versioned thing that a project resolves against by `name@version` rather than a
directory a model happens to sit beside, and records in the library itself when each requirement template
applies rather than leaving that judgement to the modeller's memory.

### Added — eventml-core

- **The library manifest** (`spec/05-concrete-syntax.md`, `examples/lib/library.yaml`): `library.yaml` at a
  library's root, using the same three-key header shape as every other file — `eventml`, `kind: library`,
  and `library` naming the library itself. Its body carries `version`, `description` and `domains`. A
  library now declares its own identity instead of being identified by wherever it happens to sit.
- **The project manifest** (`spec/05-concrete-syntax.md`, `examples/*/project.yaml`): `project.yaml` beside
  the four layer files, header-shaped the same way, naming which library the project resolves against under
  `project.library`. A project directory is now `project.yaml`, the four layer files, and `decisions.yaml`
  when the project records decisions.
- **The resolution rule** (`spec/05-concrete-syntax.md`): a model resolves against exactly one library,
  named in its `project.yaml` — no search path, no fallback, no merging of two libraries. The reference
  names a library and a version, `eventml-example@0.2.0`, not a location; where that vocabulary is stored
  and how it is fetched belongs to whatever is running the model, so the reference means the same thing in a
  git repository, a folder and a web interface. The shipped example library resolves nobody's real project —
  it exists to be read and copied from as a one-time seed, and an organisation's own copy versions
  independently from that point.
- **`applies_when` on `RequirementDef`** (`spec/02-metamodel.md`): one prose sentence stating when a
  template applies — an outdoor event implies weather protection, a spoken-word programme item implies
  intelligibility. Nothing evaluates it in v0.3; a human or an agent reads it and decides, exactly as they
  already do with a `ConstraintDef.expression`.
- **Six Mermaid diagrams in `spec/`**, one per claim the prose was carrying alone. `01-layers.md` gets the
  four-layer stack with every arrow drawn the way the edge is stored, which is boundary rule 1 as a picture.
  `02-metamodel.md` gets the eleven-entity reference graph, definitions above and usages below with the four
  `def:` arrows crossing between them, and the `Connection` versus `Flow` worked case with the connections
  drawn as elements rather than as lines — one flow riding three of them, which is the whole argument for
  keeping the two entities apart. `03-relationships.md` gets the `affect` placement next to an ordinary
  relation for contrast, and its traversal chain redrawn from ASCII art. `04-uncertainty.md` gets all seven
  question rules as one tree, including the edge where a `Decision` takes a conflict off rule 4 and hands it
  to rule 7. The language is unchanged: every diagram restates something already written beside it, and
  `00-overview.md` now says so — where a diagram and the prose disagree, the prose wins.
- **`03-relationships.md`'s traversal chain uses the model's own identifiers.** The ASCII version named
  `req.monitor_coverage` and `part.monitor_world`, which are neither catalogue IDs nor instance IDs; the
  diagram that replaces it names `r-monitor-coverage` and `monitor-world`, matching the field-read walk
  printed directly below it.

> **The two entries above shipped inside the `v0.2.0` tag, not this one.** They merged to `main` after the
> `[0.2.0]` section had been written but before the tag was cut, so `v0.2.0` contains them and its own entry
> does not describe them. They are recorded here rather than added to a released section, and this note is
> here so that anybody dating the diagrams from this entry is not misled.

### Added — eventml-lib

- **`applies_when` on all 22 requirement templates**, across all five domains, prose stating when the
  template applies rather than leaving that judgement to whoever is modelling.
- **`examples/02-conference-room/README.md` derives by hand which templates the brief triggers** — the
  first worked illustration of `applies_when`, walking the brief and the venue against every template's
  condition the way a human or an agent would.

### Changed — eventml-core

- **The example library moved to `examples/lib/`**, carrying the new manifest that names it
  `eventml-example`. A top-level `lib/` looked normative however carefully the prose denied it; the
  directory now says what the words previously had to keep saying on its behalf. 22 path references updated
  across seven files.
- **The specification names the concept, not the path.** "Definitions live in `lib/`" becomes "definitions
  live in a library"; every other rule anchored to the `lib/` path is restated the same way, including
  `CLAUDE.md`'s verification rule, which no longer points at a directory that no longer exists. Three
  further statements calling the catalogue "shared across projects" — twice in `spec/02-metamodel.md`,
  including a diagram label, and once in `spec/06-sysml-mapping.md` — are sharpened to name the
  organisation whose projects, since the shipped `lib/` is exactly the reading this release retires.
- **`eventml-lib` is the version of this repository's example library, not a special package.** Every
  library now carries a version of its own; `eventml-lib` is simply the first one to do so, generalising
  rather than staying an exception.
- **Formalisation moved from v0.5 to v0.6.** Six places named v0.5 for the machine-readable schema,
  validator and derived question list — `README.md` (three places), `spec/02-metamodel.md`,
  `spec/04-uncertainty.md`, and one outside `spec/` entirely, in
  `examples/01-garden-party/README.md`'s hand-written question list. v0.3's scope grew to cover
  applicability and the library instead, so all six are corrected to v0.6 rather than left holding a promise
  this release did not keep.

### Changed — eventml-lib

- **`video.req.screen_coverage`'s `applies_when` corrected.** It originally read "wider than its useful
  viewing angle", which excluded the long narrow room in `examples/02-conference-room` — a room that already
  had a requirement written against this exact template. Narrowed to the condition the defect actually
  required. Found by walking `applies_when` against a real example, the first time the construct was used
  for anything.

### Verification

No validator ships in this release either, so the audits below were run by hand against the repository as a
whole:

- **All 38 YAML files parse** — the 34 carried over from v0.2.0 plus the library manifest and the three
  project manifests.
- **The library-resolution audit resolves cleanly.** One library declared, `eventml-example@0.2.0`; all
  three project manifests (`01-garden-party`, `02-conference-room`, `03-festival-stage`) reference it;
  unresolved references: none.
- **The catalogue-coverage audit is empty.** 124 catalogue ids defined in `examples/lib/`, 124 distinct ids
  used across `examples/0*/`; `comm -23` between them reports nothing — no library entry sits unused by an
  example, with the library itself excluded from counting as its own consumer.
- **Version floors are correct.** `"0.3"` on the library manifest, the three project manifests and the five
  library `requirements.yaml` files (9 files in total); `"0.1"` on every `brief.yaml`.
- **The applicability walk found a real defect in the library on its first real use.** Reading
  `examples/02-conference-room`'s brief against every template's `applies_when` found that
  `video.req.screen_coverage`'s condition excluded the very room a requirement already used it for — see
  Changed — eventml-lib above. In a repository with no validator, this is the strongest evidence available
  that a new construct is pulling its weight.
- **Two templates fire with no requirement written** in `examples/02-conference-room` —
  `audio.req.coverage_uniformity` and `power.req.residual_current_protection`. Both apply on the reading
  above and neither has a requirement in the example. Recorded in the example's README as the applicability
  rule doing its job, not silently patched.

## [0.2.0] - 2026-08-09

Adds the decision record: who chose something, on what basis, and whether anyone outside the team agreed
to it.

### Added — eventml-core

- **The `Decision` entity** (`spec/07-decisions.md`): records that somebody chose among genuinely open
  alternatives and that the model now depends on the choice — an act, not a fact and not a guess. Adopts
  ISO/IEC/IEEE 42010's Architecture Decision and Architecture Rationale rather than inventing new
  vocabulary. The `by`/`agreed_by` split is what lets one entity carry an engineering choice and a client
  agreement alike without the two drifting apart into separate constructs: `by` records who made the
  decision, often the production team on its own judgement, needing no sign-off from anyone else;
  `agreed_by` records that a party outside the team was asked and assented, citing the `sources` entry
  where that assent happened rather than asserting it from memory.
- **The `affect` relation** (`spec/03-relationships.md`): the sixth traceability relation, and the only
  one stored at the upper end of its edge — on `Decision.affects`, not on the elements it determines. The
  other five sit on the lower, concrete end, so a device knows which block it realises without a search;
  `affect` breaks that pattern deliberately, because a decision's scope has to read as one reviewable
  record rather than being scattered across the files its consequences touch, and because the recording
  criterion below keeps a project's decision count low enough that scanning `decisions.yaml` backward from
  an element costs nothing. An `affects` entry also covers every value nested inside the element it names —
  naming a requirement reaches its parameters, naming a part reaches its properties — so a decision never
  has to enumerate every field it touches.
- **Question rule 7** (`spec/04-uncertainty.md`): fires on a `Decision` carrying `needs_agreement: true`
  and no `agreed_by` — something the plan depends on, possibly already built and on the truck, that nobody
  outside the team has actually signed off on. `needs_agreement` is set by the author rather than derived,
  the same pattern `ask: true` already uses on assumed values: whether a choice needs external assent is a
  judgement only the person making it can make.
- **The recording criterion** (`spec/07-decisions.md` §5): two tests a choice must pass to earn a place in
  `decisions.yaml` — a real alternative existed, and reversing the choice is expensive — drawing the line
  against the four constructs already in the language that record other kinds of thing (`assumed`,
  `derived`, `sources`, `conflicting`). Keeping the criterion narrow is also what keeps storing `affects` at
  the upper end safe: a project whose decision count approaches its requirement count is applying the
  criterion loosely.
- **The fifth project file and the `kind: decisions` header** (`spec/07-decisions.md` §2): `decisions.yaml`
  sits beside the four layer files rather than inside one of them, because a single decision routinely
  touches a brief element, a requirement and a part in the same breath. It reuses the `kind` header key
  from library files — which answers "what is in this file" — rather than `layer`, which a decisions file
  has no answer to.
- **Question rule 6** (`spec/04-uncertainty.md`): an L2 `Part` with an empty `satisfies` generates a
  question. It is rule 3 seen from the other end — rule 3 reports something promised that nothing delivers,
  rule 6 reports something delivered that nothing promised. Restricted to L2, because an L3 part inherits
  its justification through `allocate`. Its `blocks` is always 0, so it ranks last, which is correct: an
  unjustified block costs money but decides nothing. Twelve L2 blocks across the three examples trigger it;
  `examples/01-garden-party` carries the worked list.

### Added — eventml-lib

- **`decisions.yaml` in two of the three examples.** `01-garden-party` shows an agreed decision resolving a
  `conflicting` brief value (`d-audience-450`), an engineering choice needing no agreement
  (`d-digital-transport`), and a deliberately unagreed decision that rule 7 catches (`d-no-cover`).
  `03-festival-stage` shows a `supersedes` chain and the rule-4-to-rule-7 handoff: `d-foh-ground-run` is
  superseded by `d-foh-catenary`, which resolves the conflicting `brief.venue.cable_route` value but is
  itself unagreed — the council's spoken objection to a ground run was confirmed as a prohibition, not as
  assent to the flown catenary that replaced it, so the question does not vanish, it changes character
  from "which routing?" to "has the council actually signed off?". `02-conference-room` carries no
  `decisions.yaml`; the recording criterion does not require every project to have one.

### Changed — eventml-core

- **Question rule 4 gained an exception.** It no longer fires on a `conflicting` value that a `Decision`
  names in its `affects` — the value stays `conflicting` rather than being folded to `stated`, because both
  statements were genuinely made, but the question changes character instead of disappearing: if the
  deciding decision is itself unagreed, rule 7 fires on the decision in rule 4's place, tracking the
  handover from an information question to an agreement question rather than dropping it. Where one
  decision supersedes another and both name the same value, only the decision that is not itself superseded
  is the one rule 7 considers.
- **Formalisation moved from v0.2 to v0.5.** The v0.1.0 notes below said a machine-readable schema, a
  validator and a derived question list were the first task of v0.2; this release carries the decision
  record instead, so `CHANGELOG.md`, `README.md`, `spec/02-metamodel.md` and `spec/04-uncertainty.md` are
  corrected rather than left holding a promise the release did not keep.
- **The version floor is now per file, not per repository.** `eventml:` in a file's header is the lowest
  version whose constructs the file actually uses, and marks capability rather than calendar: a project's
  four layer files can stay at `"0.1"` while its `decisions.yaml` — the only file using the new `Decision`
  entity — declares `"0.2"`. Mixed floors within one project are correct.
- **Inline flow mappings are governed by a structural rule, not a character count**
  (`spec/05-concrete-syntax.md`). A record stays inline while it nests at most one level, carries no wrapped
  `Value`, and needs no comment of its own. The previous 100-character cap is removed: it was contradicted
  by 244 records across `lib/` and `examples/`, and the records breaking it — aligned connection tables
  running to about 170 characters — are more readable inline than split across six lines each. All 538
  inline records in the repository satisfy the structural rule.

### Verification

No validator ships in this release either, so the audits below were run by hand against the two new
`decisions.yaml` files and the repository as a whole:

- **Every attribute defined in `spec/07-decisions.md` is exercised.** Six decisions across the two files
  use all eleven: `id`, `date`, `by`, `decision`, `why`, `alternatives`, `affects`, `src`, `supersedes`,
  `needs_agreement`, `agreed_by`.
- **No dangling reference.** Every `affects`, `src`, `agreed_by.src` and `supersedes` in both
  `decisions.yaml` files resolves to a real id.
- **All 34 YAML files in the repository parse** — the 32 from v0.1 plus the two new `decisions.yaml`.
- **Version floors are correct**: `"0.2"` in both `decisions.yaml` files, `"0.1"` in every `brief.yaml`.
- **No catalogue concept was orphaned** by adding decisions: every `lib/` id used in `examples/` before this
  release is still used after it.

### Not in this release

No machine-readable schema, no validator, no question-list derivation — those remain planned for v0.5.

## [0.1.0] - 2026-08-08

First release. A specification only: prose and data, nothing executable.

### Added — eventml-core 0.1.0

- **Four modelling layers** (`spec/01-layers.md`): L0 Brief, L1 Requirement, L2 Logical, L3 Physical,
  each with what belongs in it, what does not, and three boundary rules.
- **Brief sources** (`spec/01-layers.md`): the `sources` list at L0, holding the communications a brief was
  built from — each with its own `kind`, `date`, `from`, optional `sender` and `subject`, and a verbatim
  `excerpt`. This is where `trace` lands: every `src:` in a model resolves to one of these entries, so the
  chain from a cable back to the sentence that caused it ends in the client's own words. One entry per
  statement rather than per meeting, so a `conflicting` value can name who said what.
- **Ten core entities** (`spec/02-metamodel.md`): the `def`/`usage` split taken from SysML v2, with
  `ItemDef`, `PortDef`, `PartDef`, `InterfaceDef`, `RequirementDef` and `ConstraintDef` as definitions,
  and `Part`, `Connection`, `Flow` and `Requirement` as usages. Each documented with a complete attribute
  table and a worked example. Includes the two rules the model cannot work without: `Connection` and
  `Flow` are separate entities, and `PartDef` carries internal traversability.
- **Five traceability relations** (`spec/03-relationships.md`): `refine`, `derive`, `satisfy`,
  `allocate`, `trace`, with cardinalities and the meaning of an absent edge. All five are stored on the
  element at the lower end of the edge, `satisfy` included — it is carried by `Part.satisfies`, which is
  where SysML v2 puts it and what makes boundary rule 1 hold without exception. The consequence is that
  tracing back from a device to the client sentence that caused it is a chain of field reads with no
  search: `allocate`, `satisfies`, `refines`, `src`.
- **The uncertainty model** (`spec/04-uncertainty.md`): progressive wrapping, the five value states
  (`stated`, `derived`, `assumed`, `unknown`, `conflicting`), the five question rules, and the ranking
  by how many requirements each open point blocks.
- **Canonical YAML syntax** (`spec/05-concrete-syntax.md`): file layout, two-document headers, catalogue
  and instance identifier formats, and the five reference forms — catalogue type, instance, port, brief
  path and source.
- **SysML v2 mapping** (`spec/06-sysml-mapping.md`): all ten entities and five relations mapped, with a
  side-by-side translation and an account of what does not survive the round trip.

### Added — eventml-lib 0.1.0

- Base vocabulary for five domains — audio, power, network, video, lighting — as 172 catalogue entries:
  signal and power items, port and interface definitions, abstract L2 blocks, concrete L3 part
  categories, requirement templates with client-language questions, and computable constraints.
- **Domain ownership rule for cross-domain parts** (`lib/README.md`): a part belongs to the domain of the
  function it performs, not to the domain of its ports. Multi-domain is the normal case — almost every
  part declares ports from two or more domains — and the domain segment of an ID is a filing decision
  carrying no semantics a model reasons with.
- Three worked models under `examples/`, spanning 112 parts and 170 connections:
  - **01 garden party** — outdoor event, audio and power, all five value states, question rules 1–4
  - **02 conference room** — hybrid corporate event, video, network and lighting, the `derive` relation,
    two flows sharing one connection, question rule 5
  - **03 festival stage** — generator-powered outdoor stage, all five domains, a flow changing medium
    three times, partial redundancy stated as partial, and power over a data connection
- Hungarian glossary (`docs/glossary-hu.md`) covering every metamodel, layer, relation and domain term,
  recording where the Hungarian trade uses the English word unchanged.

### Verification

v0.1 has no validator, so the examples carry the verification burden. Both rules from the design record
§7 were run before tagging:

- **Every defined concept appears in at least one example: 172 of 172.** Enforcing this produced example
  03 and found four defects in the domain library — ten `PortDef`s attached to no `PartDef`, missing
  audio inputs and a missing network port on the video processing parts, and no part able to feed a
  system processor over AES3. All fixed.
- **Every example is walkable across all four layers.** All catalogue references, port references, flow
  routes and allocations resolve.

Three requirements across the three examples have no upward trace, and six are satisfied by no part. These
are deliberate and are the language working as designed: a requirement from professional judgement has no
client statement to refine, and an unsatisfied requirement is question rule 3 firing. Each is annotated in
place.

### Not in this release

No machine-readable schema, no validator, no question-list derivation, no diagram generation, no Claude
skill, no Rentman or Odoo binding. The question lists in the examples are hand-written to demonstrate the
intended output. Formalising the metamodel is the first task of v0.2.
