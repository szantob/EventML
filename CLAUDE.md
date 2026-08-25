# CLAUDE.md

Conventions for anyone — human or agent — working in this repository. Read this file first; it is the
starting point for every task in the EventML plan, and most tasks are executed by agents with no other
context than what is written here.

## 1. What this repo is

EventML is a specification, not software. This repository is the language specification for EventML — an
open modelling language for live-event AV systems, spanning brief, requirement, logical and physical layers.
It ships prose and data only: no validator, no scripts, no schema, nothing executable.

## 2. Locked decisions

These are locked. Do not revisit them without an explicit instruction to do so; every later task assumes
they hold.

| # | Rule |
|---|---|
| 1 | **No executable code ships.** No validator, no scripts, no schema. `spec/`, `examples/` and `docs/` contain prose and data only. |
| 2 | **Specification language is English.** Hungarian appears only in `docs/glossary-hu.md`. |
| 3 | **Catalogue ID format:** `<domain>.<kind>.<name>` — lowercase, `snake_case` name, e.g. `audio.item.analog_line`, `audio.port.xlr3f`, `power.part.distro`, `audio.req.speech_intelligibility`. Domains: `audio`, `lighting`, `video`, `network`, `power`. Kinds: `item`, `port`, `part`, `interface`, `req`, `constraint`. |
| 4 | **Type reference vs instance ID:** a usage names its catalogue type under key `def:`; its own identifier is `id:`. Never infer one from the other. |
| 5 | **Port reference syntax:** `<part_id>:<port_id>[<index>]`, e.g. `sb1:dante[0]`. The index is omitted when the port count is 1. |
| 6 | **Every YAML file opens with a three-key header document, then `---`, then the content.** Five kinds, each identifiable from those three keys. Library domain files: `eventml`, `kind` (`items`, `parts`, `ports` or `requirements`), `domain`. Layer model files: `eventml`, `layer`, `project`. The decisions file: `eventml`, `kind: decisions`, `project`. A library manifest: `eventml`, `kind: library`, `library`. A project manifest: `eventml`, `kind: project`, `project`. A library domain file never carries `layer` or `project`; a layer model file never carries `kind` or `domain`; a library manifest is the only kind whose third key is `library`. The decisions file and a project manifest carry the same three key names — both are `eventml`, `kind`, `project` — so there it is the *value* of `kind` that tells them apart. |
| 7 | **Version floors are per file and mark capability, not calendar.** `eventml` in a file's header is the lowest version whose constructs the file actually uses: a file written entirely in v0.1 constructs stays `eventml: "0.1"`; a `decisions.yaml`, which uses the v0.2 `Decision` entity, declares `eventml: "0.2"`; a file carrying `needs`, or a `refines` that names one, declares `eventml: "0.4"`; a manifest, and any library file carrying an `applies_when`, declares `eventml: "0.3"`. Mixed floors within one project are correct — a project's brief, requirement, logical and physical files can stay at `"0.1"` while its `decisions.yaml` sits at `"0.2"`. `eventml-core` and `eventml-lib` version separately, independent of any single file's floor; `eventml-lib` is now the version of this repository's example library, declared in its manifest. |
| 8 | **Adopt, don't invent.** Where a term exists in GDTF, NMOS, IFC, SysML v2 or ISO/IEC/IEEE 42010, use that term and record the source in a `source:` key. Only coin a term when no standard has one. |
| 9 | **Commit after every task.** Never push — pushing is the user's decision. |

## 3. Where things live

| Path | Responsibility |
|---|---|
| `spec/` | The language: layers, metamodel, relations, uncertainty, decisions, concrete syntax, SysML v2 mapping. Normative |
| `examples/` | Worked models exercising all four layers, and — under `examples/lib/` — the example library they resolve against. A library is not a model; it sits inside `examples/` so that fact is visible from the layout, not only from prose. These are the only test this language has |
| `decisions.yaml` | The one project file that is not a layer. Records decisions made among genuinely open alternatives; present only when a project records decisions |
| `docs/` | Design records, implementation plans, and the Hungarian glossary |

## 4. House rules

- Write English in `spec/` and `examples/`. Hungarian appears only in `docs/glossary-hu.md`.
- Adopt terms from GDTF, NMOS, IFC, SysML v2 and ISO/IEC/IEEE 42010 rather than coining synonyms. When a term
  is adopted, record where it came from in a `source:` key on the entry that defines it.
- Commit after every task. Never push — pushing is the user's decision.

## 5. How verification works

There is no validator, so `examples/` carries the verification burden. Two rules:

- Every concept defined in `spec/` or `examples/lib/` must appear in at least one example.
- Every example must be walkable end to end across all four layers, with each requirement traced back to a
  `Need` and forward to something that satisfies it.
