# 00 — Overview

## 1. What EventML describes

EventML describes an event's AV system **topologically, not geometrically**: what connects to what, not
where it stands. A model built in EventML says that a microphone feeds a stage box which feeds a mixer over
Dante, not that the microphone sits three metres stage-left of the drum riser. Where MVR (My Virtual Rig)
captures the 3D placement of a rig — position, rotation, room geometry — EventML captures the signal and
power graph that rig realises, independent of where any of it physically stands.

## 2. The premise

The client is not an expert. A brief like "300 people, garden party, welcome speech, band later" is normal,
not deficient — it is the starting state of every model, not an error to be corrected before work begins.
EventML treats incompleteness as data: a value can be stated, assumed, derived, unknown, or conflicting, and
the model stays valid at every one of those states. The language exists to hold a plan together while it is
still full of gaps, and to make those gaps explicit rather than silently defaulted.

## 3. What the kernel covers

v0.1 establishes base concepts across five domains — audio, lighting, video, network, power — at a shallow,
uniform depth. Every domain gets the same treatment: signal types, port kinds, core part categories, and a
handful of requirement templates. Nothing goes deep yet. The goal of this release is a kernel — the four
layers, the core entities, the traceability relations, the uncertainty model — that is stable enough to carry
depth later without being restructured. Breadth first, so that if the kernel survives five domains at base
level, adding depth to any one of them will not break it.

## 4. How to read this specification

Read the files in order:

1. `00-overview.md` — this file
2. `01-layers.md` — the four modelling layers and what belongs in each
3. `02-metamodel.md` — the core entities: definitions and usages
4. `03-relationships.md` — the traceability relations that connect layers and entities
5. `04-uncertainty.md` — value states and how the open-question list is derived
6. `05-concrete-syntax.md` — the canonical YAML syntax
7. `06-sysml-mapping.md` — the mapping from EventML back to SysML v2
8. `07-decisions.md` — the `Decision` entity and the criterion for what gets recorded

`spec/` is normative: it defines the language. `examples/` is illustrative: it shows the language in use but
does not extend it. Where an example and the spec disagree, the spec wins.

The diagrams in `spec/` are illustrative in the same sense. They are drawn in Mermaid, which renders in
place on GitHub and stays diffable in the repository, and each one restates something the surrounding prose
already says. Where a diagram and the prose beside it disagree, the prose wins.

## 5. Versioning

`eventml-core` (the metamodel defined in `spec/`) versions separately from any library, because they move at
different rates: the kernel should change rarely, a vocabulary can grow at any time. `eventml-lib` is the
version of this repository's example library, declared in its manifest — the first instance of a general
rule rather than a special case: every library carries a version of its own. Every YAML file declares the
language version it was written against in an `eventml:` header key — the lowest version whose constructs
the file actually uses, not the version of the repository as a whole. A project's four layer files can stay
at `"0.1"` while its `decisions.yaml`, the only file using the `Decision` entity, declares `"0.2"`; mixed
floors within one project are correct. At this tag `eventml-core` is `0.3.0` and `eventml-lib` is `0.2.0` —
the divergence this release makes ordinary, since the two now move on their own schedules.

## 6. Relationship to existing standards

EventML does not replace the standards below. It fills the gap between them: none describes an event from
client intent down to signal topology, and none treats missing information as a modelled state.

| Standard | What EventML takes from it |
|---|---|
| SysML v2 | The metamodel — `def`/`usage`, ports, `connect` vs `flow`, `requirement`, `satisfy`, `allocate`, `constraint`. A documented mapping back to SysML v2 is part of the spec |
| ARCADIA / Capella | The four-layer separation |
| GDTF / MVR | Signal type vocabulary and the pin-level wiring model |
| AMWA NMOS | The sender/receiver and flow abstraction for networked media |
| IFC4 | Port direction semantics and port-to-port connection |
| ISO/IEC/IEEE 29148 | The term *stakeholder need*, and its separation from a system requirement — the standard's StRS and SyRS. EventML draws the same line, and draws it at the L0/L1 boundary |
| W3C Web Annotation Data Model | Passage references: the quotation selector and the position selector, kept together and deliberately redundant |
| ISO/IEC/IEEE 42010 | Architecture Decision and Architecture Rationale; the requirement that a project state which decisions it records. EventML adopts the concepts and the names in `spec/07-decisions.md`. It is a standard for describing architectures, not a format — the vocabulary and the layers still have to be written |
