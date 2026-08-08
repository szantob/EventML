# EventML Core — Design

**Date:** 2026-08-08
**Status:** Approved (design), not yet implemented
**Scope:** Sub-project 1 of 3 — the language specification only

---

## 1. Context

The wider project has one goal: let a live-event technical project manager work with an AI agent to turn a
prose client brief into (a) structured project data, (b) the full list of questions the client still needs to
answer, and (c) a draft signal-and-power plan for the event's equipment. The guiding assumption is that
**the client is not an expert and does not know what they need.** Incompleteness is the normal state of the
data, not an error condition.

Three deliverables serve that goal: a modelling language, a Claude skill that speaks it, and a script
toolkit that validates and renders it. They are separate sub-projects with separate specs. **This document
covers the first one only: the language.**

### Why a new language

No existing open standard covers this ground. The survey behind this decision:

| Standard | What it gives | Why it is not enough |
|---|---|---|
| GDTF / MVR (DIN SPEC 15800/15801) | Device digital twins, DMX patch, 3D rig, `WiringObject` + `PinPatch` pin-level graph | Lighting-centric; models the physical rig, not client intent. No audio subsystem. Commercial and logistics layers absent |
| AMWA NMOS IS-04 / IS-05 / IS-08 | Node → Device → Sender/Receiver, Source → Flow; connection management | Runtime discovery protocol, not a design format. IP media only — no analogue, no power |
| IFC4 (ISO 16739) | `IfcAudioVisualAppliance`, `IfcDistributionPort`, `IfcRelConnectsPorts` — a real typed port graph | BIM-heavy, permanent-installation oriented, shallow AV semantics (no signal format, bandwidth, channel count) |
| SysML v2 / KerML | `part def`/usage, `port def`, `connect`, `flow`, `requirement`, `satisfy`, `allocate`, `constraint` | Domain-neutral by design. All AV vocabulary would have to be written anyway |
| ARCADIA / Capella | Four-layer method: Operational → System → Logical → Physical | A method, not a data format |

EventML takes SysML v2's metamodel, ARCADIA's layering, and GDTF's signal vocabulary, and adds the domain
terms and the uncertainty model that none of them provide.

---

## 2. What ships in v1

**One deliverable: the EventML specification, written as Markdown prose.** It comprises three directories:

- `spec/` — the language itself: four layers, core entities, traceability relations, uncertainty model,
  canonical YAML syntax, SysML v2 mapping table
- `lib/` — the domain library: base vocabulary for five domains, expressed as YAML data conforming to the
  language
- `examples/` — worked models that exercise the language end to end

All three are prose and data. None of them is executable.

**Explicitly not in v1:** machine-readable schema, validator, Claude skill, scripts, diagram rendering,
Rentman/Odoo integration. Each is a later sub-project that consumes this specification as input.

### The consequence of shipping prose first

Without a machine schema, nothing validates a model. The agreed rule that *the open-question list is derived,
never hand-written* cannot execute in v1 — the agent is the only checker. This is accepted deliberately: the
concepts must settle before they are frozen into a schema. **Formalising the metamodel is the first task of
v2.**

---

## 3. Decisions

| # | Decision | Rationale |
|---|---|---|
| D1 | **Four layers**, aligned to ARCADIA | The vertical span from "wedding for 150 on the terrace" to "XLR3F port 4" is too wide for a flat model. The layer boundaries are the most consequential design choice in the language |
| D2 | **Own DSL borrowing the SysML v2 metamodel**, plus a documented mapping — not a UML profile, not literal SysML | See below |
| D3 | **Progressive wrapping** for uncertainty | A bare scalar means the client stated it; a wrapped object means anything else. Cheap and readable in the common case, fully expressive when needed |
| D4 | **Kernel is schema; domain library is data** | The SysML "model library" pattern. Only one thing needs schema technology, and the vocabulary can grow without touching the language |
| D5 | **Five domains at base-concept level**: audio, lighting, video, ethernet network, electrical | Breadth over depth. If the kernel survives five domains at base level, adding depth will not restructure it |
| D6 | **Markdown prose first**, formal schema deferred | Settle concepts before freezing them |
| D7 | **Monorepo, MIT, English spec + Hungarian glossary** | Splitting repos at 0.1 is friction; MIT matches the surrounding ecosystem (pygdtf, pymvr); the audience for an open modelling language is international |

### On D2 — why not a UML profile

1. **SysML v2 itself left UML behind.** It is built on KerML, and extension happens through model libraries and
   metadata, not stereotypes. A UML profile would target the outgoing v1.x mechanism.
2. **XMI is diff-hostile.** Every save reorders IDs; version history becomes unusable. Fatal for a format
   whose instances are versioned project documents.
3. **An agent cannot reliably write UML or SysML textual notation.** The product depends on Claude co-authoring
   models from client emails. LLMs write YAML near-perfectly and strict formal grammars unreliably, with no
   fast local validator in the loop. This decides whether the system works at all.
4. **We do not need what makes UML expensive.** No state machines, no activity semantics, no action language.
   A typed graph with traceability does not justify the metamodel cost.

**What is kept from the idea:** EventML has a real, named metamodel. It takes SysML v2 concepts one-to-one
where they fit, and introduces its own only where the domain demands it. That is precisely what SysML v2 calls
a model library extension — standards-aligned without carrying UML.

**Two concrete syntaxes:** canonical YAML (what the agent writes, what lives in git), and a documented mapping
to SysML v2 textual notation for export and interoperability. No exporter in v1.

---

## 4. Architecture

### 4.1 The four layers

| Layer | ARCADIA | Content | Example |
|---|---|---|---|
| **L0 Brief** | Operational Analysis | What the client and the audience do. Lay language, as spoken | "300 guests, gala dinner, welcome speech, band from 21:00" |
| **L1 Requirement** | System Analysis | What the system must achieve. Technology-independent, measurable | "speech intelligible across the whole audience area", "the band can hear itself" |
| **L2 Logical** | Logical Architecture | Function blocks and their signal paths. No technology choices | main PA, delay line, monitor world, microphone pool |
| **L3 Physical** | Physical Architecture | Concrete devices, ports, connections | Rio1608 · XLR3F/4 · Cat6a link |

### 4.2 Core entities

The `def`/`usage` split from SysML v2 holds throughout: the left column is a reusable catalogue type, the right
column a project instance.

| Definition (catalogue) | Usage (project) | Describes |
|---|---|---|
| `ItemDef` | — | **What flows**: AnalogAudio, DanteFlow, DMX512, SDI, AC230V |
| `PortDef` | `Port` | A connection point: which `ItemDef`, which direction, which connector |
| `PartDef` | `Part` | A device or block type: its ports, its **internal traversability**, its properties |
| `InterfaceDef` | `Connection` | A link between two ports: medium, constraints, redundancy role |
| — | `Flow` | What travels, from where to where, **over which chain of `Connection`s** |
| `RequirementDef` | `Requirement` | A requirement template with parameters, and its concrete instance |
| `ConstraintDef` | — | A computable rule: phase load, channel count, cable length |

Two points that are not obvious but without which the model is unusable:

- **`Connection` and `Flow` are separate entities.** One says *a cable exists*; the other says *what runs on it
  and by what route*. Without the split, a switch, a matrix or a Dante network cannot be modelled — there is no
  way to express that 48 channels share one Cat6.
- **`PartDef` carries internal traversability** — which input reaches which output, through what transform.
  Without it the graph is not walkable end to end and the plan cannot be checked.

---

## 5. Traceability and uncertainty

This is the intellectual core: **the question list is not a feature, it is a derivation.**

### 5.1 Traceability relations

| Relation | Direction | Meaning |
|---|---|---|
| `refine` | L0 → L1 | "band from 21:00" → "amplify live music for a five-piece band" |
| `derive` | L1 → L1 | decomposing a requirement into sub-requirements |
| `satisfy` | L2/L3 → L1 | this block or device meets that requirement |
| `allocate` | L2 → L3 | the logical block is realised by a concrete device |
| `trace` | anything → source | which client sentence, PM decision, or default |

### 5.2 Value states

Every value carries a state: `stated` (the client said it, has `src`) · `derived` (follows from a rule) ·
`assumed` (we supplied it, has `why`) · `unknown` (missing, and we know it) · `conflicting` (two sources
disagree).

### 5.3 The five question rules

A question is generated when:

1. an `unknown` value has a requirement depending on it;
2. an `assumed` value is marked `ask: true`;
3. a `Requirement` has no incoming `satisfy` edge — nothing solves it yet;
4. a value is `conflicting` — the client said two different things;
5. a `ConstraintDef` is violated — it does not fit, or will not carry the load.

Because the graph shows **how many requirements depend on each open point**, questions can be ranked. That is
what turns the output into "these three questions decide half the plan" instead of a flat list of forty. This
ranking is the payoff that justifies the cost of layering and traceability.

---

## 6. Repository

```
EventML/
├── README.md
├── LICENSE                     # MIT
├── CHANGELOG.md                # Keep a Changelog
├── CLAUDE.md                   # repo conventions for the agent
├── CONTRIBUTING.md
├── spec/
│   ├── 00-overview.md
│   ├── 01-layers.md            # L0–L3
│   ├── 02-metamodel.md         # def/usage, core entities
│   ├── 03-relationships.md     # refine/derive/satisfy/allocate/trace
│   ├── 04-uncertainty.md       # states, question derivation
│   ├── 05-concrete-syntax.md   # canonical YAML
│   └── 06-sysml-mapping.md
├── lib/                        # domain library — data, not schema
│   ├── audio/
│   ├── lighting/
│   ├── video/
│   ├── network/
│   └── power/
├── examples/
└── docs/
    └── glossary-hu.md
```

**Versioning:** `eventml-core` (metamodel) and `eventml-lib` (domain vocabulary) carry separate SemVer
versions — they move at different rates. Each file states the version it belongs to.

---

## 7. How v1 is checked

There is no validator, so **the worked examples are the test.** Two rules:

1. Every concept defined in `spec/` or `lib/` must appear in at least one example.
2. Every example must be walkable end to end across all four layers — every `Requirement` traced back to a
   `Brief` element and forward to something that satisfies it.

A review pass against both rules gates the 0.1.0 tag.

### Acceptance criteria

- [ ] All seven `spec/` files written
- [ ] Every core entity documented with purpose, attributes, relations, and at least one YAML example
- [ ] All five domains have base vocabulary: signal `ItemDef`s, `PortDef` kinds, core `PartDef` categories, and at least three `RequirementDef` templates each
- [ ] SysML v2 mapping table covers every kernel entity
- [ ] At least two worked examples exercising all four layers
- [ ] Every domain concept appears in at least one example
- [ ] Hungarian glossary covers every domain term
- [ ] `README.md`, `CONTRIBUTING.md`, `CHANGELOG.md` and `CLAUDE.md` written — the last one matters most, since the agent is the primary author across sessions

---

## 8. Risks

| Risk | Mitigation |
|---|---|
| Prose and examples drift apart with no machine check | The two example rules in §7, enforced by a review pass before tagging |
| Five shallow domains are not yet enough for a real plan | Accepted. v1 stabilises the kernel; depth follows |
| Progressive wrapping needs union types once formalised | Test the union case (`any_of` in LinkML, or the chosen alternative) on day one of v2, before committing to a schema technology |
| Vocabulary bikeshedding | Anchor to an existing standard wherever a term already exists — GDTF `SignalType`, NMOS Sender/Receiver, IFC port direction — and record the source in the spec |

---

## 9. Roadmap

1. **Sub-project 1 (this spec):** EventML language specification, prose
2. **Sub-project 2:** formalisation — machine schema, validator, derived question list, diagram generation
3. **Sub-project 3:** the Claude skill — brief extraction, question dialogue, plan drafting; plus Rentman binding
