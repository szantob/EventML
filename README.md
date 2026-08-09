# EventML

**An open modelling language for live event AV systems — from client brief to signal topology.**

> **Status: v0.3.0 — the specification is complete; nothing is executable.** The kernel is here: four
> layers, eleven core entities, six traceability relations, the uncertainty model and the canonical YAML
> syntax, in eight files under [`spec/`](spec/) — including the `Decision` entity, for recording who chose
> something and whether anyone outside the team agreed to it. Base vocabulary for five domains — 172
> concepts — is in [`examples/lib/`](examples/lib/), which now names and versions itself as `eventml-example`
> in its own manifest. Three worked models under [`examples/`](examples/) each declare, in a manifest of
> their own, which version of that library they resolve against, and exercise every one of its concepts.
> There is no schema, no validator and no tooling yet; those are planned for v0.6. The syntax may
> still change before 1.0. See [the design document](docs/superpowers/specs/2026-08-08-eventml-core-design.md).

---

## The problem

A live event technical project manager receives a brief like this:

> *"Hi — we're doing a garden party for about 300 people at the Gellért terrace on 12 September. There'll be a
> welcome speech around 7, then a live band later. Nothing fancy, but it should sound good. Can you send a
> quote?"*

Everything that matters is missing. Is the terrace covered? How big is the band? Is there three-phase power on
site? Where does the audience actually stand? The PM knows which questions to ask, but asking them well —
in an order that reflects what each answer unblocks, in language a non-expert can answer — is slow, and the
answers arrive as prose that has to be re-read every time the plan changes.

EventML makes that brief a model: incomplete on purpose, and explicit about what it does not yet know.

## The idea

Uncertainty is a first-class part of the language, not an error state.

```yaml
# A bare value means the client stated it. A wrapped value means anything else.
brief:
  audience: 300
  venue:
    name: Gellért terrace
    type:
      value: outdoor
      state: assumed
      why: "the brief says 'terrace'"
      ask: true
  program:
    - { at: "19:00", what: welcome speech }
    - { at: "21:00", what: live band, size: { state: unknown } }
```

Because every value carries its state, and every requirement is traced to the thing that satisfies it, **the
question list is derived from the model rather than written by hand** — and it can be ranked by how much each
answer unblocks:

```yaml
# derived, never authored
questions:
  - ask: "Is the terrace covered, or fully open to the sky?"
    why: "decides PA type, weather protection and power routing"
    blocks: 4 requirements
  - ask: "How many musicians, and what does each of them play?"
    why: "decides input count, monitor mixes and stage box size"
    blocks: 3 requirements
```

That ranking is the point. It turns forty scattered questions into the three that decide half the plan.

## Four layers

EventML describes an event **topologically, not geometrically** — what connects to what, not where it stands.
It spans four layers, following the [ARCADIA](https://mbse-capella.org/arcadia.html) method:

| Layer | Content | Example |
|---|---|---|
| **L0 Brief** | What the client and audience do, in lay language | "300 guests, gala dinner, band from 21:00" |
| **L1 Requirement** | What the system must achieve, technology-independent | "speech intelligible across the audience area" |
| **L2 Logical** | Function blocks and their signal paths, no technology chosen | main PA, delay line, monitor world |
| **L3 Physical** | Concrete devices, ports, connections | Rio1608 · XLR3F/4 · Cat6a link |

## Relationship to existing standards

EventML does not replace the standards below. It fills the gap between them: none describes an event from
client intent down to signal topology, and none treats missing information as a modelled state.

| Standard | What EventML takes from it |
|---|---|
| [SysML v2](https://github.com/systems-modeling/sysml-v2-release) | The metamodel — `def`/`usage`, ports, `connect` vs `flow`, `requirement`, `satisfy`, `allocate`, `constraint`. A documented mapping back to SysML v2 is part of the spec |
| [ARCADIA / Capella](https://mbse-capella.org/arcadia.html) | The four-layer separation |
| [GDTF / MVR](https://www.gdtf.eu/) | Signal type vocabulary and the pin-level wiring model |
| [AMWA NMOS](https://specs.amwa.tv/nmos/) | The sender/receiver and flow abstraction for networked media |
| [IFC4](https://ifc43-docs.standards.buildingsmart.org/) | Port direction semantics and port-to-port connection |
| [ISO/IEC/IEEE 42010](https://www.iso.org/standard/74393.html) | Architecture Decision and Architecture Rationale; the requirement that a project state which decisions it records. EventML adopts the concepts and the names in `spec/07-decisions.md`. It is a standard for describing architectures, not a format — the vocabulary and the layers still have to be written |

EventML is **not** a UML profile. SysML v2 itself moved off UML to KerML, XMI is hostile to version control,
and — decisively — an AI agent cannot write strict formal grammars reliably, while it writes YAML almost
perfectly. The reasoning is set out in full in the [design document](docs/superpowers/specs/2026-08-08-eventml-core-design.md#on-d2--why-not-a-uml-profile).

## Layout

```
spec/       the language: layers, metamodel, relations, uncertainty, decisions, syntax, SysML mapping
examples/   worked models exercising all four layers, plus the example library they resolve against
docs/       design records and the Hungarian glossary
```

The **kernel** (`spec/`) is the language. The **domain library** (`examples/lib/`) is data written in that language,
and it is versioned separately — vocabulary grows without touching the language.

## Scope at v0.3

Base concepts across five domains — audio, lighting, video, ethernet network, electrical — with the kernel
stable enough that adding depth later will not restructure it. v0.1 built that kernel; v0.2 added the
`Decision` entity and the `affect` relation, for recording who chose something and whether anyone outside
the team agreed to it.

v0.3 is about where a model's vocabulary comes from. A library declares its own name and version in a
manifest; a project declares which library it resolves against, by name and version rather than by
location, so the same reference means something in a git repository, a folder and a web interface. A model
resolves against exactly one library. Requirement templates gained `applies_when`, one sentence saying when
each applies — an outdoor event implies weather protection, a spoken-word item implies intelligibility —
so that judgement lives in the library rather than in whoever happens to be modelling.

v0.3 is still a specification: prose and data, no executable code. A machine-readable schema, a validator,
derived question lists and diagram generation follow in v0.6.

Because there is no validator, **the worked examples are the only test v0.3 has**, under two rules: every
defined concept must appear in at least one example, and every example must be walkable across all four
layers. Both are met — 172 of 172 library concepts exercised — and enforcing the first one is what produced
the third example and found four real defects in the domain library. The question lists in the examples are
hand-written to show the intended output; deriving them is a v0.6 task.

## Language

The specification is written in English. Domain terms are mapped to Hungarian in
[`docs/glossary-hu.md`](docs/glossary-hu.md). Questions generated *for clients* are localised — that is the
job of the tooling, not of the language.

## Contributing

Proposals for new domain concepts are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md). Where a term already
exists in an established standard, EventML adopts it and records the source rather than inventing a synonym.

## Licence

[MIT](LICENSE)
