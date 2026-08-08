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

## 3. What v0.1 covers

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

`spec/` is normative: it defines the language. `lib/` and `examples/` are illustrative: they show the
language in use but do not extend it. Where an example and the spec disagree, the spec wins.

## 5. Versioning

`eventml-core` (the metamodel defined in `spec/`) and `eventml-lib` (the domain vocabulary defined in
`lib/`) version separately, because they move at different rates: the kernel should change rarely, the
vocabulary can grow at any time. Every YAML file, in `lib/` and in `examples/` alike, declares the language
version it was written against in an `eventml:` header key. At the v0.1 tag both `eventml-core` and
`eventml-lib` are `0.1.0`, and every file declares `eventml: "0.1"`.

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
