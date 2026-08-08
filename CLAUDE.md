# CLAUDE.md

Conventions for anyone — human or agent — working in this repository. Read this file first; it is the
starting point for every task in the EventML v0.1 plan, and most tasks are executed by agents with no other
context than what is written here.

## 1. What this repo is

EventML is a specification, not software. This repository is the language specification for EventML v0.1 —
an open modelling language for live-event AV systems, spanning brief, requirement, logical and physical
layers. v0.1 ships prose and data only: no validator, no scripts, no schema, nothing executable.

## 2. Locked decisions

These are locked. Do not revisit them without an explicit instruction to do so; every later task assumes
they hold.

| # | Rule |
|---|---|
| 1 | **No executable code ships in v0.1.** No validator, no scripts, no schema. `spec/`, `lib/`, `examples/` and `docs/` contain prose and data only. |
| 2 | **Specification language is English.** Hungarian appears only in `docs/glossary-hu.md`. |
| 3 | **Catalogue ID format:** `<domain>.<kind>.<name>` — lowercase, `snake_case` name, e.g. `audio.item.analog_line`, `audio.port.xlr3f`, `power.part.distro`, `audio.req.speech_intelligibility`. Domains: `audio`, `lighting`, `video`, `network`, `power`. Kinds: `item`, `port`, `part`, `interface`, `req`, `constraint`. |
| 4 | **Type reference vs instance ID:** a usage names its catalogue type under key `def:`; its own identifier is `id:`. Never infer one from the other. |
| 5 | **Port reference syntax:** `<part_id>:<port_id>[<index>]`, e.g. `sb1:dante[0]`. The index is omitted when the port count is 1. |
| 6 | **Every YAML file starts with the same three header keys:** `eventml`, `layer`, and either `domain` (library files) or `project` (model files). |
| 7 | **Version floors:** `eventml: "0.1"` in every YAML file. `eventml-core` and `eventml-lib` version separately; both are `0.1.0` at the v0.1 tag. |
| 8 | **Adopt, don't invent.** Where a term exists in GDTF, NMOS, IFC or SysML v2, use that term and record the source in a `source:` key. Only coin a term when no standard has one. |
| 9 | **Commit after every task.** Never push — pushing is the user's decision. |

## 3. Where things live

| Path | Responsibility |
|---|---|
| `spec/` | The language: layers, metamodel, relations, uncertainty, concrete syntax, SysML v2 mapping. Normative |
| `lib/` | Domain library for audio, lighting, video, network, power — YAML data written in the language. Illustrative |
| `examples/` | Worked models exercising all four layers. These are the only test v0.1 has |
| `docs/` | Design records, implementation plans, and the Hungarian glossary |

## 4. House rules

- Write English in `spec/`, `lib/` and `examples/`. Hungarian appears only in `docs/glossary-hu.md`.
- Adopt terms from GDTF, NMOS, IFC and SysML v2 rather than coining synonyms. When a term is adopted, record
  where it came from in a `source:` key on the entry that defines it.
- Commit after every task. Never push — pushing is the user's decision.

## 5. How v0.1 is verified

v0.1 has no validator, so `examples/` carries the verification burden. Two rules:

- Every concept defined in `spec/` or `lib/` must appear in at least one example.
- Every example must be walkable end to end across all four layers, with each requirement traced back to a
  brief element and forward to something that satisfies it.
