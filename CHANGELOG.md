# Changelog

All notable changes to this project are documented in this file.
The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

`eventml-core` (the metamodel) and `eventml-lib` (the domain vocabulary) are
versioned separately. Entries state which one changed.

## [Unreleased]

### Added — eventml-core

- **Question rule 6** (`spec/04-uncertainty.md`): an L2 `Part` with an empty `satisfies` generates a
  question. It is rule 3 seen from the other end — rule 3 reports something promised that nothing delivers,
  rule 6 reports something delivered that nothing promised. Restricted to L2, because an L3 part inherits
  its justification through `allocate`. Its `blocks` is always 0, so it ranks last, which is correct: an
  unjustified block costs money but decides nothing. Twelve L2 blocks across the three examples trigger it;
  `examples/01-garden-party` carries the worked list.

### Changed — eventml-core

- **Inline flow mappings are governed by a structural rule, not a character count**
  (`spec/05-concrete-syntax.md`). A record stays inline while it nests at most one level, carries no wrapped
  `Value`, and needs no comment of its own. The previous 100-character cap is removed: it was contradicted
  by 244 records across `lib/` and `examples/`, and the records breaking it — aligned connection tables
  running to about 170 characters — are more readable inline than split across six lines each. All 538
  inline records in the repository satisfy the structural rule.

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
