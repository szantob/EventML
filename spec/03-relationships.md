# 03 — Relationships

Five relations connect the four layers and the entities within them. Between them they answer one question:
for any statement in the model, where did it come from and what depends on it.

| Relation | Direction | Carried by | Meaning |
|---|---|---|---|
| `refine` | L0 → L1 | `Requirement.refines` | A brief element becomes a requirement |
| `derive` | L1 → L1 | `Requirement.derived_from` | A requirement decomposes into sub-requirements |
| `satisfy` | L2/L3 → L1 | `Requirement.satisfied_by` | This block or device meets that requirement |
| `allocate` | L2 → L3 | `Part.allocate` | A logical block is realised by a concrete device |
| `trace` | any → source | `Value.src` | Which client sentence, PM decision or default |

None of the five is a standalone entity. Each is an attribute on the element at the lower end of the edge —
the requirement knows which brief element it refines, the device knows which block it realises. This is what
boundary rule 1 in `01-layers.md` requires: a layer never references downward, so an L1 requirement stays
valid no matter which L2 or L3 design ends up meeting it.

The worked examples throughout this file use one scenario: a garden party for 300 on a terrace, with a
welcome speech at 19:00 and a live band from 21:00, where nobody has yet said how many musicians there are.

## refine

**YAML form.** `Requirement.refines`, holding a path into the L0 brief.

```yaml
- id: r-speech-terrace
  def: audio.req.speech_intelligibility
  params:
    area: "the terrace seating area"
    audience: 300
  refines: brief.program.welcome_speech
```

**Cardinality.** One requirement refines zero or one brief element. A brief element may be refined by many
requirements — "live band from 21:00" produces a requirement for amplification, one for monitoring, and one
for stage power.

**When the edge is absent.** The requirement did not come from the client; it came from professional
judgement. That is legitimate and common — nobody asks for a residual current device — but it is worth
surfacing, because a requirement with no client origin is one the client has never agreed to and may not
expect to pay for. An absent `refines` is not an error, but it should be visible in review.

## derive

**YAML form.** `Requirement.derived_from`, holding the id of the parent requirement.

```yaml
- id: r-band-monitoring
  def: audio.req.performer_monitoring
  params:
    performers: { state: unknown }
  derived_from: r-band-amplification
```

**Cardinality.** One requirement derives from zero or one parent. A parent may have many children. The
result is a forest, not a graph: decomposition is a tree, and a requirement with two parents is a modelling
error that means one of the two edges is really a `satisfy` in disguise.

**When the edge is absent.** The requirement is a root — it came straight from a brief element via `refine`,
or from judgement. Root requirements are the ones worth reading to a client; derived ones are usually
technical consequences the client does not need to see.

## satisfy

**YAML form.** `Requirement.satisfied_by`, holding a list of `Part` ids at L2, L3, or both.

```yaml
- id: r-speech-terrace
  def: audio.req.speech_intelligibility
  params: { area: "the terrace seating area", audience: 300 }
  refines: brief.program.welcome_speech
  satisfied_by: [main-pa, delay-line, pa-left, pa-right]
```

**Cardinality.** Many-to-many. One requirement may need several parts to meet it — a speech is intelligible
because of the main hang *and* the delay line — and one part may satisfy several requirements, which is what
makes a piece of equipment worth its truck space.

**When the edge is absent.** Nothing in the design meets this requirement yet. An empty `satisfied_by` is
not a modelling error; it is the normal state of a requirement that has been captured but not yet designed
for, and it is exactly what question rule 3 in `04-uncertainty.md` fires on. A model is expected to hold
requirements in this state, sometimes for weeks.

## allocate

**YAML form.** `Part.allocate` on the L3 part, holding the id of the L2 part it realises.

```yaml
# L2 — the logical block
- id: main-pa
  def: audio.part.pa_system
  layer: logical

# L3 — the devices realising it
- id: pa-left
  def: audio.part.line_array_top
  layer: physical
  allocate: main-pa

- id: pa-right
  def: audio.part.line_array_top
  layer: physical
  allocate: main-pa
```

**Cardinality.** Every L3 part allocates from exactly one L2 part; one L2 part may be realised by many L3
parts. This is boundary rule 3 in `01-layers.md`.

**When the edge is absent.** An L3 part with no `allocate` is a modelling error, not an open question: a
device has appeared in the plan that no logical decision called for. In practice it means either that the
device is genuinely unnecessary, or — more often — that a logical block was skipped and the reasoning behind
the device was never written down. Either way it needs fixing rather than asking.

## trace

**YAML form.** `src` inside a value wrapper, naming an entry in the brief's `sources` list. The wrapper is
defined in `04-uncertainty.md` and the source entry in `01-layers.md`; here only the `src` attribute matters.

```yaml
brief:
  sources:
    - id: s-site-visit
      kind: site_visit
      date: 2026-08-04
      from: production
      sender: "technical contact, venue"
      excerpt: >-
        Confirmed the CEE outlet by the service door — 32 A three-phase, on its own RCD.

  audience: 300                       # bare scalar: stated, source is the brief itself
  venue:
    covered:
      value: false
      state: assumed
      why: "the brief says 'terrace'; no marquee mentioned"
      ask: true
    power_available:
      value: "32 A three-phase"
      state: stated
      src: s-site-visit
```

**Cardinality.** Every non-`stated` value carries one `src` or one `why`, depending on its state. A single
source may be traced to by any number of values — one site visit answers a dozen questions.

**When the edge is absent.** For a `stated` value written as a bare scalar, absence is correct: the source is
the brief document itself, and repeating that on every line would be noise. For any other state, a missing
`src` or `why` means the model has lost the reasoning behind a value, and the next person to read it cannot
tell an assumption from a fact. That is the failure mode `trace` exists to prevent.

## Traversal

The five relations chain into one path from a sentence in a client's email down to a cable. Following the
band from the garden-party brief:

```
brief.program[1].size          (L0, state: unknown)
   └─refine→ req.monitor_coverage        (L1)
              └─satisfy→ part.monitor_world   (L2)
                          └─allocate→ part.wedge_1..4  (L3)
                                       └─connect→ c-amp-wedge-1..4
```

Nobody has said how many musicians are in the band. That single unknown at L0 refines into a requirement
for monitor coverage, which is satisfied by a logical monitor world, which is allocated to four wedges,
which are connected to an amplifier. Every element below the unknown is provisional: four wedges is a guess
that follows from a guess about the band's size.

**The rule that makes the language pay off: any node on this chain that is missing, unknown or unsatisfied
propagates upward into a question, and the number of requirements downstream of it is its rank.** The band's
size is not one question among forty. It is the question that decides monitor count, monitor mixes, input
count, stage box size and stage power — and the model knows that, because those requirements are reachable
from it by following `refine`, `derive` and `satisfy` edges. Counting them is what turns a flat list into
"these three questions decide half the plan".

`04-uncertainty.md` sets out the five conditions that generate a question and how the count is computed.
