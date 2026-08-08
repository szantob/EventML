# EventML Core v0.1 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Write the EventML v0.1 language specification — `spec/`, `lib/` and `examples/` — as prose and YAML data, with no executable code in the repository.

**Architecture:** Three layers of artifact. `spec/` defines the language (four modelling layers, seven core entities, five traceability relations, five value states). `lib/` is the domain vocabulary for audio, lighting, video, network and power, written as YAML data conforming to that language. `examples/` are worked models that exercise the language end to end and serve as the only test v0.1 has.

**Tech Stack:** Markdown (CommonMark, GitHub-flavoured tables), YAML 1.2. Verification uses ripgrep and a one-off Python + PyYAML check — neither is a repository dependency.

**Source spec:** [`docs/superpowers/specs/2026-08-08-eventml-core-design.md`](../specs/2026-08-08-eventml-core-design.md)

## Global Constraints

- **No executable code ships in v0.1.** No validator, no scripts, no schema. `spec/`, `lib/`, `examples/` and `docs/` contain prose and data only.
- **Specification language is English.** Hungarian appears only in `docs/glossary-hu.md`.
- **Catalogue ID format:** `<domain>.<kind>.<name>` — lowercase, `snake_case` name, e.g. `audio.item.analog_line`, `audio.port.xlr3f`, `power.part.distro`, `audio.req.speech_intelligibility`. Domains: `audio`, `lighting`, `video`, `network`, `power`. Kinds: `item`, `port`, `part`, `interface`, `req`, `constraint`.
- **Type reference vs instance ID:** a usage names its catalogue type under key `def:`; its own identifier is `id:`. Never infer one from the other.
- **Port reference syntax:** `<part_id>:<port_id>[<index>]`, e.g. `sb1:dante[0]`. The index is omitted when the port count is 1.
- **Every YAML file starts with the same three header keys:** `eventml`, `layer`, and either `domain` (library files) or `project` (model files).
- **Version floors:** `eventml: "0.1"` in every YAML file. `eventml-core` and `eventml-lib` version separately; both are `0.1.0` at the v0.1 tag.
- **Adopt, don't invent.** Where a term exists in GDTF, NMOS, IFC or SysML v2, use that term and record the source in a `source:` key. Only coin a term when no standard has one.
- **Commit after every task.** Never push — pushing is the user's decision.

---

## File Structure

| Path | Responsibility |
|---|---|
| `.gitignore`, `.gitattributes` | OS/editor noise; line-ending normalisation (repo lives under OneDrive) |
| `CLAUDE.md` | Repo conventions for the agent — the locked decisions, so every session starts from the same place |
| `CONTRIBUTING.md` | How to propose a domain concept; the adopt-don't-invent rule |
| `CHANGELOG.md` | Keep a Changelog format |
| `spec/00-overview.md` | What EventML is, what v0.1 covers, how to read the spec, versioning |
| `spec/01-layers.md` | L0–L3, what belongs in each, the boundary rules |
| `spec/02-metamodel.md` | `def`/`usage`; the seven core entities and their attributes |
| `spec/03-relationships.md` | `refine`, `derive`, `satisfy`, `allocate`, `trace` |
| `spec/04-uncertainty.md` | Progressive wrapping, the five value states, the five question rules |
| `spec/05-concrete-syntax.md` | Canonical YAML: file layout, headers, IDs, references |
| `spec/06-sysml-mapping.md` | Every kernel entity mapped to SysML v2 |
| `lib/README.md` | How the library is organised; the four-file pattern per domain |
| `lib/<domain>/items.yaml` | `ItemDef`s — what flows |
| `lib/<domain>/ports.yaml` | `PortDef`s — connection points |
| `lib/<domain>/parts.yaml` | `PartDef`s — device and block categories |
| `lib/<domain>/requirements.yaml` | `RequirementDef` templates |
| `examples/01-garden-party/` | Small outdoor event: audio + power, all four layers, heavy uncertainty |
| `examples/02-conference-room/` | Corporate room: video + network + lighting, all four layers |
| `docs/glossary-hu.md` | Hungarian mapping of every domain term |

Each domain library is split into four files by entity kind rather than one file per domain. Items, ports, parts and requirements change at different rates and are reviewed by different eyes; keeping them apart keeps each file small enough to hold in context.

---

## Task 1: Repository scaffolding

**Files:**
- Create: `.gitignore`, `.gitattributes`, `CLAUDE.md`, `CONTRIBUTING.md`, `CHANGELOG.md`

**Interfaces:**
- Consumes: nothing
- Produces: `CLAUDE.md` carries the locked conventions — every later task reads it first. The ID format, reference syntax and header keys defined there are quoted verbatim from Global Constraints above.

- [ ] **Step 1: Install the one-off verification dependency**

```bash
pip install pyyaml
```

Expected: `Successfully installed pyyaml-...` or `Requirement already satisfied`. This is a local developer tool, not a repository dependency — it must not appear in any committed file.

- [ ] **Step 2: Write `.gitignore`**

```gitignore
# OS
.DS_Store
Thumbs.db
desktop.ini

# Editors
.vscode/
.idea/
*.swp
*~

# Python (verification tooling only — no Python ships in this repo)
__pycache__/
*.py[cod]
.venv/
venv/
```

- [ ] **Step 3: Write `.gitattributes`**

The repository lives under OneDrive on Windows; without this, every file round-trips between LF and CRLF and diffs become unreadable.

```gitattributes
* text=auto eol=lf
*.md   text eol=lf
*.yaml text eol=lf
*.yml  text eol=lf
```

- [ ] **Step 4: Write `CLAUDE.md`**

Required sections, in this order:

1. **What this repo is** — one paragraph: EventML is a specification, not software; v0.1 ships prose and data only.
2. **Locked decisions** — a table reproducing, verbatim, every bullet from the Global Constraints section of this plan. Any agent session must be able to start here without reading the design doc.
3. **Where things live** — the File Structure table above, abridged to the directory level.
4. **House rules** — English in `spec/`, `lib/`, `examples/`; Hungarian only in `docs/glossary-hu.md`. Adopt terms from GDTF/NMOS/IFC/SysML v2 with a `source:` key rather than coining synonyms. Commit after every task; never push.
5. **How v0.1 is verified** — the two example rules: every defined concept appears in at least one example, and every example is walkable across all four layers.

- [ ] **Step 5: Write `CONTRIBUTING.md`**

Required sections:

1. **Scope** — v0.1 accepts corrections and domain-concept proposals; it does not accept code.
2. **Proposing a domain concept** — open an issue stating: the term, the domain, which entity kind it is (`item`/`port`/`part`/`interface`/`req`/`constraint`), whether an existing standard already names it (and where), and one concrete situation where a model needs it.
3. **The adopt-don't-invent rule** — stated explicitly, with GDTF `SignalType`, NMOS Sender/Receiver and IFC port direction named as the first places to look.
4. **Naming** — the catalogue ID format, quoted from `CLAUDE.md`.

- [ ] **Step 6: Write `CHANGELOG.md`**

```markdown
# Changelog

All notable changes to this project are documented in this file.
The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

`eventml-core` (the metamodel) and `eventml-lib` (the domain vocabulary) are
versioned separately. Entries state which one changed.

## [Unreleased]

### Added
- Repository scaffolding: contribution guide, agent conventions, line-ending policy.
```

- [ ] **Step 7: Verify line-ending policy took effect**

```bash
git add -A && git status --short
```

Expected: five files staged (`A` status), and **no** `LF will be replaced by CRLF` warnings — the earlier commits produced them, this one must not.

- [ ] **Step 8: Commit**

```bash
git commit -m "Add repository scaffolding

Locks the conventions in CLAUDE.md so every agent session starts from the
same decisions, and normalises line endings for a repo living under OneDrive.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Task 2: Overview and layers

**Files:**
- Create: `spec/00-overview.md`, `spec/01-layers.md`

**Interfaces:**
- Consumes: `CLAUDE.md` conventions from Task 1
- Produces: the layer names `L0 Brief`, `L1 Requirement`, `L2 Logical`, `L3 Physical` and the `layer:` header values `brief`, `requirement`, `logical`, `physical`. Tasks 3–16 use these exact strings.

- [ ] **Step 1: Write `spec/00-overview.md`**

Required sections:

1. **What EventML describes** — an event's AV system topologically, not geometrically: what connects to what, not where it stands. State the contrast with MVR explicitly in one sentence.
2. **The premise** — the client is not an expert; incompleteness is the normal state of a model, not an error.
3. **What v0.1 covers** — base concepts across five domains, kernel stable, no depth yet. Name the five domains.
4. **How to read this specification** — the file order 00→06, and the rule that `spec/` is normative while `lib/` and `examples/` are illustrative.
5. **Versioning** — `eventml-core` and `eventml-lib` version separately; every YAML file declares `eventml: "0.1"`.
6. **Relationship to existing standards** — reproduce the five-row table from `README.md` (SysML v2, ARCADIA, GDTF/MVR, NMOS, IFC4) so the spec stands alone.

- [ ] **Step 2: Write the layer table in `spec/01-layers.md`**

Open with this table, then give each layer its own subsection:

| Layer | `layer:` value | ARCADIA phase | Content |
|---|---|---|---|
| L0 Brief | `brief` | Operational Analysis | What the client and audience do, in lay language |
| L1 Requirement | `requirement` | System Analysis | What the system must achieve, technology-independent |
| L2 Logical | `logical` | Logical Architecture | Function blocks and signal paths, no technology chosen |
| L3 Physical | `physical` | Physical Architecture | Concrete devices, ports, connections |

- [ ] **Step 3: Write the per-layer subsections**

Each of the four subsections must state, in this order: **what belongs here**, **what does not**, **which entities from `spec/02-metamodel.md` may appear**, and **two worked one-line examples** — one well-formed, one that belongs in a neighbouring layer, with a sentence saying why.

Entity permissions, which later tasks depend on:

- L0 Brief: free-form structured values only. No `Part`, no `Connection`, no `Flow`.
- L1 Requirement: `Requirement` only.
- L2 Logical: `Part`, `Port`, `Connection`, `Flow` — all referencing technology-neutral `PartDef`s.
- L3 Physical: `Part`, `Port`, `Connection`, `Flow` — referencing concrete `PartDef`s, each `allocate`d from an L2 `Part`.

- [ ] **Step 4: Write the boundary rules section**

Three rules, each with a one-line rationale:

1. **A layer never references downward.** A `Requirement` does not name a device. Downward association is expressed by `satisfy` and `allocate` edges, which belong to the lower layer.
2. **L2 carries no product names.** If removing the manufacturer would change the meaning, it belongs in L3.
3. **Every L3 `Part` is allocated from exactly one L2 `Part`.** An unallocated L3 part is a modelling error, and question rule 3 will surface it.

- [ ] **Step 5: Verify layer strings are consistent**

```bash
rg -o '`(brief|requirement|logical|physical)`' spec/01-layers.md | sort -u
```

Expected: exactly four lines — `` `brief` ``, `` `requirement` ``, `` `logical` ``, `` `physical` ``. No capitals, no plurals. These strings become the `layer:` header value in every model file, so a variant here propagates into every example.

- [ ] **Step 6: Commit**

```bash
git add spec/00-overview.md spec/01-layers.md
git commit -m "Add spec overview and layer definitions

Defines the four modelling layers, which entities each admits, and the three
boundary rules that keep them separate.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Task 3: Metamodel

**Files:**
- Create: `spec/02-metamodel.md`

**Interfaces:**
- Consumes: layer names from Task 2
- Produces: the seven entity names and their complete attribute lists. Every later task — the whole of `lib/` and both examples — uses these attribute names verbatim.

- [ ] **Step 1: Write the `def`/`usage` section**

Explain the split in three paragraphs: a `def` is a reusable catalogue type that lives in `lib/`; a `usage` is a project instance that lives in a model file; a usage names its type with `def:` and carries its own `id:`. State that this is taken from SysML v2 and say why it matters here — the catalogue is shared across projects and maps onto rental inventory, while instances are per-event.

- [ ] **Step 2: Write the definition-entity sections**

One subsection per entity. Each must give: purpose in one sentence, a complete attribute table (attribute, type, required, meaning), and a full YAML example. Required attributes:

**`ItemDef`** — what flows.

| Attribute | Type | Required | Meaning |
|---|---|---|---|
| `id` | string | yes | Catalogue ID |
| `name` | string | yes | Human-readable name |
| `kind` | `signal` \| `power` \| `data` | yes | Broad class |
| `form` | `analog` \| `digital` \| `optical` \| `electrical` | yes | Physical realisation |
| `carries` | string | yes | What it conveys, in prose |
| `properties` | map | no | Domain-specific attributes |
| `source` | string | no | Standard the term is adopted from |
| `notes` | string | no | Free text |

```yaml
- id: audio.item.analog_line
  name: Analogue line level
  kind: signal
  form: analog
  carries: One channel of balanced line-level audio
  properties:
    nominal_level: "+4 dBu"
    balanced: true
  source: "GDTF SignalType: AnalogAudio"
```

**`PortDef`** — a connection point.

| Attribute | Type | Required | Meaning |
|---|---|---|---|
| `id` | string | yes | Catalogue ID |
| `name` | string | yes | Human-readable name |
| `item` | ItemDef ref | yes | What passes through |
| `direction` | `source` \| `sink` \| `bidir` | yes | Flow direction, IFC semantics |
| `connector` | string | yes | Physical connector |
| `pins` | int | no | Pin count where meaningful |
| `properties` | map | no | |
| `source` | string | no | |

```yaml
- id: audio.port.xlr3f
  name: XLR 3-pin female
  item: audio.item.analog_line
  direction: sink
  connector: XLR3F
  pins: 3
  source: "IFC IfcDistributionPort FlowDirection=SINK"
```

**`PartDef`** — a device or block type.

| Attribute | Type | Required | Meaning |
|---|---|---|---|
| `id` | string | yes | Catalogue ID |
| `name` | string | yes | |
| `category` | string | yes | e.g. `mixer`, `loudspeaker`, `distro` |
| `abstract` | bool | no | `true` for technology-neutral L2 blocks |
| `ports` | list | yes | Each: `id`, `def`, `count`, optional `role` |
| `internal` | list | no | Each: `from`, `to`, optional `transform` — internal traversability |
| `properties` | map | no | |
| `catalog_ref` | map | no | External system IDs, e.g. `{ rentman: 4417 }` |
| `source` | string | no | |

```yaml
- id: audio.part.stage_box
  name: Digital stage box
  category: stage_box
  ports:
    - { id: analog_in,  def: audio.port.xlr3f,    count: 16 }
    - { id: analog_out, def: audio.port.xlr3m,    count: 8 }
    - { id: net,        def: network.port.ethercon_a, count: 2, role: primary_secondary }
    - { id: pwr,        def: power.port.powercon_true1, count: 1 }
  internal:
    - { from: "analog_in[*]", to: net,             transform: adc }
    - { from: net,            to: "analog_out[*]", transform: dac }
  properties:
    electrical_load_w: 100
```

**`InterfaceDef`** — a link type between two ports.

| Attribute | Type | Required | Meaning |
|---|---|---|---|
| `id` | string | yes | |
| `name` | string | yes | |
| `item` | ItemDef ref | yes | What the link carries |
| `medium` | string | yes | e.g. `cat6a`, `xlr_cable`, `h07rn-f_3g2.5` |
| `capacity` | map | no | e.g. `{ channels: 64, mbit: 1000 }` |
| `constraints` | list of ConstraintDef refs | no | |

**`RequirementDef`** — a requirement template.

| Attribute | Type | Required | Meaning |
|---|---|---|---|
| `id` | string | yes | |
| `name` | string | yes | |
| `text` | string | yes | Template with `{param}` placeholders |
| `params` | list | yes | Each: `name`, `type`, optional `default` |
| `verification` | string | yes | How you would check it was met |
| `constraints` | list of ConstraintDef refs | no | |
| `asks` | list | no | Client-facing question templates for unfilled params |

```yaml
- id: audio.req.speech_intelligibility
  name: Speech intelligibility
  text: "Speech must be intelligible across {area} for an audience of {audience}."
  params:
    - { name: area,     type: string }
    - { name: audience, type: int }
  verification: "Coverage prediction, or measured STI at the three worst listening positions."
  asks:
    - param: area
      question: "Where exactly will people be standing or sitting during the speeches?"
```

**`ConstraintDef`** — a computable rule. In v0.1 the expression is prose; v0.2 formalises it.

| Attribute | Type | Required | Meaning |
|---|---|---|---|
| `id` | string | yes | |
| `name` | string | yes | |
| `applies_to` | string | yes | Which entity kind it constrains |
| `expression` | string | yes | Prose statement of the rule |
| `params` | list | no | |

- [ ] **Step 3: Write the usage-entity sections**

Same treatment — attribute table plus YAML example — for `Part`, `Connection`, `Flow` and `Requirement`.

**`Part`**: `id`, `def` (required), `label`, `layer`, `properties` (overrides the def), `allocate` (L3 only — the L2 `Part` id it realises).

**`Connection`**: `id`, `from` (port ref), `to` (port ref), `interface` (InterfaceDef ref), `redundancy` (`primary` \| `secondary` \| `none`), `properties`.

**`Flow`**: `id`, `item` (ItemDef ref), `from` (part id), `to` (part id), `over` (ordered list of Connection ids), `redundant_over` (list), `properties` (e.g. `channels`).

**`Requirement`**: `id`, `def`, `params` (map filling the template), `refines` (L0 path), `derived_from` (Requirement id), `satisfied_by` (list of Part ids — may be empty, which is question rule 3).

- [ ] **Step 4: Write the "Connection and Flow are separate" section**

Two paragraphs and one worked example. Show a stage box, a switch and a console: three `Connection`s and one `Flow` carrying 32 channels over all three. State plainly that without the split there is no way to express that many channels share one cable, and that switches, matrices and Dante networks become unmodellable.

- [ ] **Step 5: Write the "internal traversability" section**

One paragraph plus an example. State that without `internal`, the graph is not walkable end to end, so no plan can be checked. Note that GDTF's `PinPatch` does the same job for power and DMX, and cite it as the precedent.

- [ ] **Step 6: Verify every entity is documented**

```bash
rg -n "^### (ItemDef|PortDef|PartDef|InterfaceDef|RequirementDef|ConstraintDef|Part|Connection|Flow|Requirement)" spec/02-metamodel.md
```

Expected: exactly 10 matches, one per entity.

- [ ] **Step 7: Commit**

```bash
git add spec/02-metamodel.md
git commit -m "Add metamodel: def/usage split and the ten core entities

Documents each entity with a complete attribute table and a worked YAML
example, plus the two rules the model cannot work without: Connection and
Flow are separate, and PartDef carries internal traversability.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Task 4: Traceability relations

**Files:**
- Create: `spec/03-relationships.md`

**Interfaces:**
- Consumes: entity names and attributes from Task 3
- Produces: the five relation names `refine`, `derive`, `satisfy`, `allocate`, `trace` and the attribute each is carried by. Task 5's question rules and both examples depend on these.

- [ ] **Step 1: Write the relation table**

| Relation | Direction | Carried by | Meaning |
|---|---|---|---|
| `refine` | L0 → L1 | `Requirement.refines` | A brief element becomes a requirement |
| `derive` | L1 → L1 | `Requirement.derived_from` | A requirement decomposes into sub-requirements |
| `satisfy` | L2/L3 → L1 | `Requirement.satisfied_by` | This block or device meets that requirement |
| `allocate` | L2 → L3 | `Part.allocate` | A logical block is realised by a concrete device |
| `trace` | any → source | `Value.src` | Which client sentence, PM decision or default |

- [ ] **Step 2: Write one subsection per relation**

Each must give: the exact YAML form, a worked example using the garden-party scenario, cardinality (one-to-one, one-to-many, many-to-many), and what it means when the edge is absent.

Cardinalities, which the examples must respect:

- `refine`: one Requirement refines zero or one brief path. Zero means the requirement came from professional judgement, not the client — and that is worth surfacing.
- `derive`: one Requirement derives from zero or one parent.
- `satisfy`: many-to-many. Empty means unsolved — question rule 3.
- `allocate`: every L3 Part allocates from exactly one L2 Part.
- `trace`: every non-`stated` value carries one.

- [ ] **Step 3: Write the traversal section**

Show the full chain for one requirement in prose plus a fenced ASCII diagram:

```
brief.program[1].size          (L0, state: unknown)
   └─refine→ req.monitor_coverage        (L1)
              └─satisfy→ part.monitor_world   (L2)
                          └─allocate→ part.wedge_1..4  (L3)
                                       └─connect→ ...
```

State the rule that makes the whole language pay off: **any node on this chain that is missing, unknown or unsatisfied propagates upward into a question**, and the number of requirements downstream of it is its rank.

- [ ] **Step 4: Verify**

```bash
rg -n "refines|derived_from|satisfied_by|allocate|src" spec/03-relationships.md
```

Expected: each of the five attribute names appears at least twice — once in the table, once in its subsection.

- [ ] **Step 5: Commit**

```bash
git add spec/03-relationships.md
git commit -m "Add traceability relations

Defines refine, derive, satisfy, allocate and trace with cardinalities, and
the traversal rule that turns a gap anywhere on the chain into a ranked
question.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Task 5: Uncertainty and question derivation

**Files:**
- Create: `spec/04-uncertainty.md`

**Interfaces:**
- Consumes: relations from Task 4
- Produces: the `Value` wrapper attributes and the five state names. Both examples use these.

- [ ] **Step 1: Write the progressive wrapping section**

State the rule in one sentence: **a bare scalar means the client stated it; a wrapped object means anything else.** Then the wrapper's attributes:

| Attribute | Type | Required | Meaning |
|---|---|---|---|
| `value` | any | no | Absent when `state: unknown` |
| `state` | see below | yes | |
| `why` | string | when `assumed` or `derived` | The reasoning |
| `src` | string | when `stated` or `conflicting` | Where it came from |
| `ask` | bool | no | Force this into the question list |
| `alternatives` | list | when `conflicting` | The competing values with their sources |

Show both forms side by side:

```yaml
audience: 300                       # stated

venue_type:                         # everything else
  value: outdoor
  state: assumed
  why: "the brief says 'terrace'"
  ask: true
```

- [ ] **Step 2: Write the five states section**

| State | Meaning | Requires |
|---|---|---|
| `stated` | The client said it | `src` |
| `derived` | Follows from a rule or another value | `why` |
| `assumed` | We supplied it to keep moving | `why` |
| `unknown` | Missing, and we know it is missing | — |
| `conflicting` | Two sources disagree | `alternatives` |

State explicitly that a bare scalar is shorthand for `state: stated` and that the two forms are otherwise identical in meaning.

- [ ] **Step 3: Write the five question rules**

Number them 1–5, matching the design document exactly. For each: the trigger condition, what the generated question must contain, and a worked example.

1. An `unknown` value with a `Requirement` depending on it.
2. An `assumed` value marked `ask: true`.
3. A `Requirement` with an empty `satisfied_by`.
4. A `conflicting` value.
5. A violated `ConstraintDef`.

Each generated question carries: `ask` (the client-facing wording), `why` (what it decides, in the PM's terms), and `blocks` (the count of requirements downstream).

- [ ] **Step 4: Write the ranking section**

State that `blocks` is computed by counting requirements reachable downstream of the open node through `refine`, `derive` and `satisfy` edges, and that questions sort by `blocks` descending. Give the garden-party example with three questions and their counts. Add the note that in v0.1 this list is written by hand in examples to demonstrate the intended output; v0.2 derives it.

- [ ] **Step 5: Write the client-language section**

One short section, because it is the point of the whole project: the `ask` string is written for someone who does not know the domain. Three before/after pairs — "Do you need front-fill?" → "Will anyone be standing right at the front, close to the stage?" — with one sentence on what makes the rewrite better.

- [ ] **Step 6: Verify**

```bash
rg -n "stated|derived|assumed|unknown|conflicting" spec/04-uncertainty.md | head -20
rg -c "^#### Rule [1-5]" spec/04-uncertainty.md
```

Expected: all five states present; exactly 5 rule headings.

- [ ] **Step 7: Commit**

```bash
git add spec/04-uncertainty.md
git commit -m "Add uncertainty model and question derivation

Progressive wrapping, the five value states, and the five rules that turn a
model's gaps into a ranked, client-readable question list.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Task 6: Concrete syntax

**Files:**
- Create: `spec/05-concrete-syntax.md`

**Interfaces:**
- Consumes: everything from Tasks 2–5
- Produces: the file headers, directory layout and reference syntax that every file in `lib/` and `examples/` must follow. This task gates all of them.

- [ ] **Step 1: Write the file layout section**

A project model is a directory of four files, one per layer:

```
<project>/
├── brief.yaml
├── requirements.yaml
├── logical.yaml
└── physical.yaml
```

A library domain is a directory of four files, one per entity kind:

```
lib/<domain>/
├── items.yaml
├── ports.yaml
├── parts.yaml
└── requirements.yaml
```

- [ ] **Step 2: Write the header section**

Every YAML file opens with these keys. Library files:

```yaml
eventml: "0.1"
kind: items          # items | ports | parts | requirements
domain: audio
---
items:
  - id: audio.item.analog_line
    ...
```

Model files:

```yaml
eventml: "0.1"
layer: brief         # brief | requirement | logical | physical
project: garden-party
---
brief:
  ...
```

- [ ] **Step 3: Write the identifier section**

State the catalogue ID format `<domain>.<kind>.<name>` with the closed domain and kind lists from Global Constraints. State that instance IDs are short, lowercase, unique within a project, and carry no dots — so a reader can tell a catalogue reference from an instance reference by sight.

- [ ] **Step 4: Write the reference syntax section**

Four reference forms, each with an example:

| Form | Syntax | Example |
|---|---|---|
| Catalogue type | dotted ID under `def:` | `def: audio.part.stage_box` |
| Instance | bare short id | `sb1` |
| Port | `<part_id>:<port_id>[<index>]` | `sb1:analog_in[3]` |
| Brief path | dotted path from the `brief` root | `brief.program[1].size` |

State that the index is omitted when `count: 1`, and that `[*]` means all instances of a port, valid only inside `PartDef.internal`.

- [ ] **Step 5: Write the conventions section**

Two-space indent, no tabs. Inline flow mappings permitted for short single-line records (ports, internal edges) and required to stay under 100 characters. Keys ordered `id`, `name`/`label`, `def`, then the rest. UTF-8, LF endings.

- [ ] **Step 6: Verify the examples in this file parse**

```bash
python -c "import yaml,re,sys,pathlib; t=pathlib.Path('spec/05-concrete-syntax.md').read_text(encoding='utf-8'); blocks=re.findall(r'\`\`\`yaml\n(.*?)\`\`\`', t, re.S); [list(yaml.safe_load_all(b)) for b in blocks]; print(f'{len(blocks)} yaml blocks parsed OK')"
```

Expected: `N yaml blocks parsed OK` with no traceback.

- [ ] **Step 7: Commit**

```bash
git add spec/05-concrete-syntax.md
git commit -m "Add concrete syntax

File layout, headers, identifier format and the four reference forms. This
is the contract every library and example file follows.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Task 7: Audio domain library

**Files:**
- Create: `lib/README.md`, `lib/audio/items.yaml`, `lib/audio/ports.yaml`, `lib/audio/parts.yaml`, `lib/audio/requirements.yaml`

**Interfaces:**
- Consumes: syntax from Task 6, entity attributes from Task 3
- Produces: the four-file-per-domain pattern that Tasks 8, 10, 11 and 12 replicate. Audio goes first because its topology is the richest — analogue and networked, sources and sinks, matrixing — so it stresses the metamodel hardest.

- [ ] **Step 1: Write `lib/README.md`**

Cover: the four-file pattern; that library content is data, not schema; that `eventml-lib` versions separately from `eventml-core`; and the adopt-don't-invent rule with the `source:` key.

- [ ] **Step 2: Write `lib/audio/items.yaml`**

At minimum: `analog_mic`, `analog_line`, `aes3`, `dante_flow`, `madi`, `speaker_level`. Each with `kind`, `form`, `carries`, and a `source:` where GDTF or AES names it.

- [ ] **Step 3: Write `lib/audio/ports.yaml`**

At minimum: `xlr3f`, `xlr3m`, `trs_jack`, `nl4`, `speakon_nl8`, `rj45_dante`, `bnc_madi`. Each names its `item` and `direction`.

- [ ] **Step 4: Write `lib/audio/parts.yaml`**

Two groups, clearly separated by a comment.

**Abstract (L2, `abstract: true`)**: `main_pa`, `delay_line`, `front_fill`, `monitor_world`, `microphone_pool`, `playback_source`, `mix_position`.

**Concrete categories (L3)**: `microphone`, `di_box`, `stage_box`, `mixing_console`, `dsp_processor`, `amplifier`, `loudspeaker`, `wireless_receiver`.

Every concrete part must declare `ports` and, where it has both inputs and outputs, `internal`.

- [ ] **Step 5: Write `lib/audio/requirements.yaml`**

At least five templates: `speech_intelligibility`, `music_reproduction`, `stage_monitoring`, `input_capacity`, `coverage_uniformity`. Each with `text`, `params`, `verification`, and at least one `asks` entry written in client language.

- [ ] **Step 6: Verify the files parse and IDs are well-formed**

```bash
python -c "import yaml,glob; [list(yaml.safe_load_all(open(f,encoding='utf-8'))) for f in glob.glob('lib/audio/*.yaml')]; print('audio parses OK')"
rg -o "id: [a-z_]+\.[a-z]+\.[a-z0-9_]+" lib/audio/ | rg -v "^lib/audio/[a-z]+\.yaml:id: audio\." || echo "all IDs correctly namespaced"
```

Expected: `audio parses OK`, then `all IDs correctly namespaced`.

- [ ] **Step 7: Commit**

```bash
git add lib/README.md lib/audio/
git commit -m "Add audio domain library

Items, ports, abstract L2 blocks and concrete L3 categories, plus five
requirement templates with client-language questions. Audio goes first
because its topology stresses the metamodel hardest.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Task 8: Power domain library

**Files:**
- Create: `lib/power/items.yaml`, `lib/power/ports.yaml`, `lib/power/parts.yaml`, `lib/power/requirements.yaml`

**Interfaces:**
- Consumes: the four-file pattern from Task 7
- Produces: `power.port.powercon_true1` and the load properties that `audio.part.stage_box` already references. Task 10's example needs audio, power and network all resolvable to be walkable.

- [ ] **Step 1: Write `lib/power/items.yaml`**

At minimum: `ac_230v_1p`, `ac_400v_3p`, `dc_48v`. Each with `kind: power`, `form: electrical`, and voltage/frequency in `properties`.

- [ ] **Step 2: Write `lib/power/ports.yaml`**

At minimum: `schuko`, `powercon_true1_in`, `powercon_true1_out`, `cee_16a_3p`, `cee_32a_3p`, `harting_6p`. Note that inlet and outlet are separate `PortDef`s because `direction` differs — call this out in a comment, since it is the first place a reader meets the pattern.

- [ ] **Step 3: Write `lib/power/parts.yaml`**

**Abstract (L2)**: `power_source`, `power_distribution`, `load`.
**Concrete (L3)**: `distro`, `cable_drum`, `ups`, `generator`, `rcd_unit`.

Each concrete part declares `ports` and `internal` where current passes through.

- [ ] **Step 4: Write `lib/power/requirements.yaml`**

At least three: `supply_capacity`, `phase_balance`, `residual_current_protection`. `supply_capacity` must have an `asks` entry that a venue manager could answer — not "what is the service rating" but "can you send a photo of the distribution board, or tell us what the biggest socket on site looks like?"

- [ ] **Step 5: Verify**

```bash
python -c "import yaml,glob; [list(yaml.safe_load_all(open(f,encoding='utf-8'))) for f in glob.glob('lib/power/*.yaml')]; print('power parses OK')"
rg -n "power.port.powercon_true1" lib/ | head
```

Expected: `power parses OK`, and the port referenced by `audio.part.stage_box` in Task 7 now resolves — if the ID differs, fix the library, not the reference.

- [ ] **Step 6: Commit**

```bash
git add lib/power/
git commit -m "Add power domain library

Items, ports, distribution parts and three requirement templates. Resolves
the power port that the audio stage box already referenced.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Task 9: Network domain library

**Files:**
- Create: `lib/network/items.yaml`, `lib/network/ports.yaml`, `lib/network/parts.yaml`, `lib/network/requirements.yaml`

**Interfaces:**
- Consumes: the pattern from Task 7; `network.port.ethercon_a` is already referenced by `audio.part.stage_box` and is dangling until this task lands
- Produces: transport types the video and lighting libraries build on, and the switching parts Task 10's example needs for its stage-box → switch → console chain

- [ ] **Step 1: Write `items.yaml`**

At minimum: `ethernet_1g`, `ethernet_10g`, `fibre_om3`, `dante_transport`, `sacn`, `artnet`, `ptp_clock`. Adopt NMOS and AES67 terminology where it exists and record it in `source:`.

- [ ] **Step 2: Write `ports.yaml`**

At minimum: `rj45`, `ethercon_a`, `ethercon_b`, `sfp_cage`, `lc_duplex`.

- [ ] **Step 3: Write `parts.yaml`**

**Abstract (L2)**: `network_backbone`, `network_edge`, `clock_master`.
**Concrete (L3)**: `switch_managed`, `switch_unmanaged`, `media_converter`, `wireless_ap`.

`switch_managed` must declare `internal` connecting every port to every other — this is the case that proves `Flow` cannot be inferred from `Connection` alone.

- [ ] **Step 4: Write `requirements.yaml`**

At least three: `bandwidth_headroom`, `clock_stability`, `network_redundancy`.

- [ ] **Step 5: Verify, and confirm the audio library's forward reference now resolves**

```bash
python -c "import yaml,glob; [list(yaml.safe_load_all(open(f,encoding='utf-8'))) for f in glob.glob('lib/network/*.yaml')]; print('network parses OK')"
rg -n "network.port.ethercon_a" lib/
```

Expected: `network parses OK`, and the ID appears both in `lib/audio/parts.yaml` (the reference) and `lib/network/ports.yaml` (the definition). If they differ, fix the definition to match the reference.

- [ ] **Step 6: Commit**

```bash
git add lib/network/
git commit -m "Add network domain library

Transport items, port types, switching parts with full internal
traversability, and three requirement templates. Resolves the ethercon port
the audio stage box already referenced.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Task 10: Garden party example

**Files:**
- Create: `examples/01-garden-party/README.md`, `brief.yaml`, `requirements.yaml`, `logical.yaml`, `physical.yaml`

**Interfaces:**
- Consumes: everything from Tasks 2–9
- Produces: the first end-to-end proof that the language works. If a concept cannot be expressed here, the spec is wrong and must be fixed in this task, not worked around.

This is the plan's first real test. Treat a failure to express something as a spec defect.

- [ ] **Step 1: Write `brief.yaml`**

Model the brief quoted in `README.md` — 300 guests, Gellért terrace, 12 September, welcome speech at 19:00, band later, "should sound good". It must contain at least one value of each state: `stated` (audience), `assumed` (venue type), `unknown` (band size), and one `conflicting` value (invent a plausible second client message giving a different guest count).

- [ ] **Step 2: Write `requirements.yaml`**

At least five `Requirement`s drawn from `lib/audio` and `lib/power` templates. Each carries `refines` pointing at a brief path. **At least one must have an empty `satisfied_by`** so question rule 3 fires and the example demonstrates it.

- [ ] **Step 3: Write `logical.yaml`**

L2 `Part`s from the abstract audio and power blocks: `main_pa`, `front_fill`, `monitor_world`, `microphone_pool`, `power_distribution`. Connect them with `Connection`s and route at least two `Flow`s.

- [ ] **Step 4: Write `physical.yaml`**

L3 `Part`s, each with `allocate` naming its L2 parent. Include the stage-box → switch → console chain so one `Flow` runs over three `Connection`s and demonstrates the Connection/Flow split concretely.

- [ ] **Step 5: Write `README.md` with the hand-derived question list**

Narrate what the example demonstrates, then give the ranked question list — hand-written in v0.1 — showing `ask`, `why` and `blocks` for at least four questions, with at least one from each of question rules 1, 2, 3 and 4. State clearly at the top that this list is hand-derived to show intended output, and that v0.2 computes it.

- [ ] **Step 6: Verify the example parses and every reference resolves**

```bash
python -c "import yaml,glob; [list(yaml.safe_load_all(open(f,encoding='utf-8'))) for f in glob.glob('examples/01-garden-party/*.yaml')]; print('example parses OK')"
```

Then check every `def:` reference resolves to a library ID:

```bash
rg -oh "def: ([a-z_]+\.[a-z]+\.[a-z0-9_]+)" -r '$1' examples/01-garden-party/ | sort -u > /tmp/used.txt
rg -oh "id: ([a-z_]+\.[a-z]+\.[a-z0-9_]+)" -r '$1' lib/ | sort -u > /tmp/defined.txt
comm -23 /tmp/used.txt /tmp/defined.txt
```

Expected: **empty output.** Any line printed is a reference to a type that does not exist — add it to the library or fix the reference.

- [ ] **Step 7: Verify the four-layer walk**

By hand, pick one requirement and trace it: brief path → `refines` → requirement → `satisfied_by` → L2 part → `allocate` → L3 part → `Connection`. Write the chain into the example README as a fenced block. If any link is missing, the example is incomplete.

- [ ] **Step 8: Commit**

```bash
git add examples/01-garden-party/
git commit -m "Add garden party example

First end-to-end model: all four layers, all five value states, and a
hand-derived ranked question list demonstrating four of the five question
rules.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Task 11: Video domain library

**Files:**
- Create: `lib/video/items.yaml`, `lib/video/ports.yaml`, `lib/video/parts.yaml`, `lib/video/requirements.yaml`

**Interfaces:**
- Consumes: the pattern from Task 7; network transports from Task 9
- Produces: parts used by Task 13's example

- [ ] **Step 1: Write `items.yaml`**

At minimum: `sdi_3g`, `sdi_12g`, `hdmi_2_0`, `dp_1_4`, `ndi_stream`, `st2110_video`. Record the SMPTE and NMOS sources.

- [ ] **Step 2: Write `ports.yaml`**

At minimum: `bnc_sdi`, `hdmi_a`, `displayport`, `rj45_ndi`, `lc_duplex_video`.

- [ ] **Step 3: Write `parts.yaml`**

**Abstract (L2)**: `image_source`, `image_processing`, `image_display`.
**Concrete (L3)**: `camera`, `media_server`, `video_switcher`, `scaler`, `led_processor`, `led_panel`, `projector`, `display_screen`.

`video_switcher` and `scaler` must declare `internal`.

- [ ] **Step 4: Write `requirements.yaml`**

At least three: `image_legibility`, `source_switching`, `screen_coverage`. `image_legibility` needs an `asks` entry a client can answer — about the furthest seat and whether slides will carry small text, not about pixel pitch.

- [ ] **Step 5: Verify**

```bash
python -c "import yaml,glob; [list(yaml.safe_load_all(open(f,encoding='utf-8'))) for f in glob.glob('lib/video/*.yaml')]; print('video parses OK')"
```

- [ ] **Step 6: Commit**

```bash
git add lib/video/
git commit -m "Add video domain library

Signal items, connector ports, source/processing/display parts, and three
requirement templates.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Task 12: Lighting domain library

**Files:**
- Create: `lib/lighting/items.yaml`, `lib/lighting/ports.yaml`, `lib/lighting/parts.yaml`, `lib/lighting/requirements.yaml`

**Interfaces:**
- Consumes: the pattern from Task 7; `sacn` and `artnet` from Task 9
- Produces: the domain where GDTF alignment is tightest — cite GDTF in `source:` throughout

- [ ] **Step 1: Write `items.yaml`**

At minimum: `dmx512`, `rdm`, `dmx_over_sacn`, `dmx_over_artnet`. Cite GDTF `SignalType: DMX512` as the source.

- [ ] **Step 2: Write `ports.yaml`**

At minimum: `xlr5f_dmx`, `xlr5m_dmx`, `xlr3f_dmx`, `rj45_sacn`.

- [ ] **Step 3: Write `parts.yaml`**

**Abstract (L2)**: `key_light`, `wash_light`, `effect_light`, `control_surface`, `dmx_distribution`.
**Concrete (L3)**: `moving_head`, `par_fixture`, `profile_fixture`, `lighting_console`, `dmx_node`, `dimmer`.

State in a comment that concrete fixture *models* belong in GDTF, not here — EventML models the topology and defers the device digital twin to GDTF. Add `catalog_ref: { gdtf: ... }` to at least one part to show the intended binding.

- [ ] **Step 4: Write `requirements.yaml`**

At least three: `face_lighting`, `stage_wash`, `control_universes`.

- [ ] **Step 5: Verify**

```bash
python -c "import yaml,glob; [list(yaml.safe_load_all(open(f,encoding='utf-8'))) for f in glob.glob('lib/lighting/*.yaml')]; print('lighting parses OK')"
rg -c "gdtf" lib/lighting/
```

Expected: parses OK, and at least three GDTF citations.

- [ ] **Step 6: Commit**

```bash
git add lib/lighting/
git commit -m "Add lighting domain library

DMX transport items, connector ports, fixture and control parts, and three
requirement templates. Defers device digital twins to GDTF and shows the
catalog_ref binding.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Task 13: Conference room example

**Files:**
- Create: `examples/02-conference-room/README.md`, `brief.yaml`, `requirements.yaml`, `logical.yaml`, `physical.yaml`

**Interfaces:**
- Consumes: all five domain libraries
- Produces: coverage of video, network and lighting, which example 01 does not exercise

- [ ] **Step 1: Write `brief.yaml`**

A corporate event: 120 people, hotel conference room, panel discussion plus slide presentation, hybrid — remote speakers dialling in. Include at least two `unknown` values and one `assumed` value with `ask: true`.

- [ ] **Step 2: Write `requirements.yaml`**

At least six `Requirement`s spanning audio, video, network and lighting. Include one `derived_from` chain — a parent requirement decomposing into two children — since example 01 does not demonstrate `derive`.

- [ ] **Step 3: Write `logical.yaml`**

L2 parts across four domains, connected, with at least three `Flow`s. Include the case a switch makes necessary: two `Flow`s sharing one `Connection`.

- [ ] **Step 4: Write `physical.yaml`**

L3 parts, each `allocate`d. Include the camera → switcher → media server → display chain.

- [ ] **Step 5: Write `README.md`**

Narrate what this example adds over example 01 — video, network, lighting, `derive`, and shared-link flows — and give a hand-derived ranked question list including one instance of question rule 5 (a violated constraint), which example 01 does not cover.

- [ ] **Step 6: Verify parse and reference resolution**

```bash
python -c "import yaml,glob; [list(yaml.safe_load_all(open(f,encoding='utf-8'))) for f in glob.glob('examples/02-conference-room/*.yaml')]; print('example 02 parses OK')"
rg -oh "def: ([a-z_]+\.[a-z]+\.[a-z0-9_]+)" -r '$1' examples/02-conference-room/ | sort -u > /tmp/used2.txt
rg -oh "id: ([a-z_]+\.[a-z]+\.[a-z0-9_]+)" -r '$1' lib/ | sort -u > /tmp/defined.txt
comm -23 /tmp/used2.txt /tmp/defined.txt
```

Expected: parses OK, and empty output from `comm`.

- [ ] **Step 7: Commit**

```bash
git add examples/02-conference-room/
git commit -m "Add conference room example

Covers video, network and lighting, plus the derive relation and two flows
sharing one connection. Demonstrates question rule 5.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Task 14: SysML v2 mapping

**Files:**
- Create: `spec/06-sysml-mapping.md`

**Interfaces:**
- Consumes: the ten entities from Task 3 and five relations from Task 4
- Produces: the standards-alignment claim the README makes

Written after the examples deliberately — mapping a language you have exercised is honest; mapping one you have only imagined is not.

- [ ] **Step 1: Write the entity mapping table**

Every one of the ten entities in one row: EventML name, SysML v2 construct, and a note where the fit is imperfect.

| EventML | SysML v2 | Note |
|---|---|---|
| `ItemDef` | `item def` | Direct |
| `PortDef` | `port def` | EventML fixes direction on the port; SysML puts it on the feature |
| `PartDef` | `part def` | `internal` has no direct SysML equivalent; nearest is internal `connect` in the part's body |
| `InterfaceDef` | `interface def` | Direct |
| `RequirementDef` | `requirement def` | Direct |
| `ConstraintDef` | `constraint def` | EventML v0.1 keeps the expression as prose |
| `Part` | `part` usage | Direct |
| `Connection` | `connect` | Direct |
| `Flow` | `flow` | Direct |
| `Requirement` | `requirement` usage | Direct |

- [ ] **Step 2: Write the relation mapping table**

`refine` → `refine`; `derive` → `derive`; `satisfy` → `satisfy`; `allocate` → `allocate`; `trace` → metadata annotation. Note that SysML v2 has no equivalent of the value-state model — that is EventML's addition, and the reason a straight translation loses information.

- [ ] **Step 3: Write a side-by-side translation**

Take one `PartDef` from `lib/audio/parts.yaml` and show it in EventML YAML and in SysML v2 textual notation, side by side. State which information survives the round trip and which does not.

- [ ] **Step 4: Write the "what does not map" section**

Three items: value states, question derivation, and the layer constraint that L2 carries no product names. Say plainly that these are why EventML exists rather than a SysML profile.

- [ ] **Step 5: Verify every entity and relation appears**

```bash
rg -c "ItemDef|PortDef|PartDef|InterfaceDef|RequirementDef|ConstraintDef" spec/06-sysml-mapping.md
rg -c "refine|derive|satisfy|allocate|trace" spec/06-sysml-mapping.md
```

Expected: both non-zero, and manual read confirms all ten entities and five relations have rows.

- [ ] **Step 6: Commit**

```bash
git add spec/06-sysml-mapping.md
git commit -m "Add SysML v2 mapping

Maps all ten entities and five relations, with a side-by-side translation and
an honest account of what does not survive the round trip.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Task 15: Hungarian glossary

**Files:**
- Create: `docs/glossary-hu.md`

**Interfaces:**
- Consumes: every term defined in `spec/` and `lib/`
- Produces: the file `README.md` already links to

- [ ] **Step 1: Extract every term needing translation**

```bash
rg -oh "^\s*name: (.+)$" -r '$1' lib/ | sort -u
```

This is the working list. Every line must appear in the glossary.

- [ ] **Step 2: Write the glossary**

Four tables — metamodel terms, layer names, domain terms grouped by domain, and relation names. Each row: English term, Hungarian term, one-line note where the Hungarian trade term differs from a literal translation. Where the Hungarian industry uses the English word unchanged, say so rather than inventing a Hungarian one.

- [ ] **Step 3: Verify coverage**

```bash
rg -oh "^\s*name: (.+)$" -r '$1' lib/ | sort -u > /tmp/terms.txt
wc -l < /tmp/terms.txt
rg -c "\|" docs/glossary-hu.md
```

Expected: the glossary's row count is at least the term count. Spot-check five terms by hand.

- [ ] **Step 4: Commit**

```bash
git add docs/glossary-hu.md
git commit -m "Add Hungarian glossary

Maps every metamodel, layer, relation and domain term, noting where the
Hungarian trade uses the English word unchanged.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Task 16: Release gate

**Files:**
- Modify: `README.md`, `CHANGELOG.md`

**Interfaces:**
- Consumes: everything
- Produces: the v0.1.0 tag

- [ ] **Step 1: Run the concept-coverage audit**

Rule 1 from the design document: every concept defined in `spec/` or `lib/` appears in at least one example.

```bash
rg -oh "id: ([a-z_]+\.[a-z]+\.[a-z0-9_]+)" -r '$1' lib/ | sort -u > /tmp/defined.txt
rg -oh "([a-z_]+\.[a-z]+\.[a-z0-9_]+)" -r '$1' examples/ | sort -u > /tmp/used.txt
comm -23 /tmp/defined.txt /tmp/used.txt
```

Expected: empty. Every line printed is a defined concept no example uses. For each, either extend an example to use it or delete the concept — an unexercised concept is unvalidated, and v0.1 has no other validation.

- [ ] **Step 2: Run the reference-integrity audit**

```bash
comm -13 /tmp/defined.txt /tmp/used.txt
```

Expected: only instance IDs and brief paths, no catalogue-shaped IDs. Any dotted `<domain>.<kind>.<name>` here is a dangling reference.

- [ ] **Step 3: Verify every YAML file in the repo parses**

```bash
python -c "import yaml,glob; fs=glob.glob('**/*.yaml',recursive=True); [list(yaml.safe_load_all(open(f,encoding='utf-8'))) for f in fs]; print(f'{len(fs)} files parsed OK')"
```

Expected: a count and no traceback.

- [ ] **Step 4: Fix the README's dangling links**

`README.md` links `CONTRIBUTING.md` (created in Task 1) and `docs/glossary-hu.md` (created in Task 15). Confirm both resolve:

```bash
ls CONTRIBUTING.md docs/glossary-hu.md
```

Then add a short **Status** update to `README.md` replacing "Nothing here is implemented yet" with an accurate statement of what v0.1 contains.

- [ ] **Step 5: Update `CHANGELOG.md`**

Move everything from `[Unreleased]` into a `## [0.1.0] - <today>` section, listing: `eventml-core 0.1.0` (four layers, ten entities, five relations, five value states, canonical YAML syntax, SysML v2 mapping) and `eventml-lib 0.1.0` (base vocabulary for five domains, two worked examples).

- [ ] **Step 6: Commit and tag**

```bash
git add README.md CHANGELOG.md
git commit -m "Release v0.1.0

Completes the EventML core specification: four layers, ten entities, five
traceability relations, the uncertainty model, base vocabulary across five
domains, and two worked examples that exercise all of it.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
git tag -a v0.1.0 -m "EventML core 0.1.0 — language specification"
```

- [ ] **Step 7: Report, do not push**

Report the tag and the audit results to the user. Pushing the branch and the tag is the user's decision.

---

## Self-Review

**Spec coverage.** Every section of the design document maps to a task: §2 deliverable → Tasks 2–15; §4.1 layers → Task 2; §4.2 entities → Task 3; §5.1 relations → Task 4; §5.2–5.3 uncertainty and question rules → Task 5; §6 repository → Task 1; §7 verification rules and acceptance criteria → Task 16, with the per-domain criteria enforced in Tasks 7–12 and the example criteria in Tasks 9 and 13; §3 D7 glossary → Task 15. The SysML mapping required by the acceptance criteria is Task 14. No design-document requirement lacks a task.

**Deliberate ordering choices.** The audio, power and network libraries precede example 01 because that example's stage-box → switch → console chain references all three, and its reference-integrity check would fail without them. Example 01 then precedes the video and lighting libraries so the metamodel meets a real model early, while changing it is still cheap. The SysML mapping comes after both examples so it maps a language that has been exercised.

**Type consistency.** Attribute names are used identically throughout: `def`, `id`, `label`, `layer`, `allocate`, `refines`, `derived_from`, `satisfied_by`, `over`, `redundant_over`, `state`, `why`, `src`, `ask`, `alternatives`, `internal`, `catalog_ref`, `source`. Port reference syntax is `<part_id>:<port_id>[<index>]` in every task that uses it. The five state names and five relation names are spelled the same in Tasks 4, 5, 9, 13 and 14.

**Known gap, accepted.** Question rule 5 requires a violated `ConstraintDef`, but v0.1 constraint expressions are prose, so the violation is asserted by the example author rather than detected. Task 13 Step 5 covers it as a demonstration. Detection arrives with the validator in v0.2.
