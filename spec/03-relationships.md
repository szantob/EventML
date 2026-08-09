# 03 — Relationships

Six relations connect the four layers and the entities within them. Between them they answer one question:
for any statement in the model, where did it come from and what depends on it.

| Relation | Direction | Carried by | Meaning |
|---|---|---|---|
| `refine` | L0 → L1 | `Requirement.refines` | A brief element becomes a requirement |
| `derive` | L1 → L1 | `Requirement.derived_from` | A requirement decomposes into sub-requirements |
| `satisfy` | L2/L3 → L1 | `Part.satisfies` | This block or device meets that requirement |
| `allocate` | L2 → L3 | `Part.allocate` | A logical block is realised by a concrete device |
| `trace` | any → source | `Value.src` | Which client sentence, PM decision or default |
| `affect` | Decision → any | `Decision.affects` | A decision determined this element |

None of the six is a standalone entity. Five are attributes on the element at the lower end of the edge — the
requirement knows which brief element it refines, the device knows which block it realises, the block knows
which requirement it meets — and that placement is what boundary rule 1 in `01-layers.md` requires: a layer
never references downward, so an L1 requirement stays valid no matter which L2 or L3 design ends up meeting
it. `affect` is the sixth, and it is a deliberate exception rather than an oversight: `affects` sits on the
`Decision`, at the upper end of its edge, not on the elements it determines. The `affect` section below gives
the reasons.

The lower-end rule has a consequence worth stating plainly, because it is what makes the model readable
backwards. Every one of the five edges points from the concrete toward the abstract, so tracing *back* from a
cable to the sentence that caused it is a chain of field reads — `allocate`, then `satisfies`, then
`refines`, then `src` — with no search at any step. Tracing *forward*, from a client sentence to the
equipment it produced, is the direction that requires a scan. That asymmetry is deliberate: the backward
question is the one asked under pressure, on site, about a device somebody is holding. `affect` is the one
edge on that walk which cannot be followed this way: reaching from an element to the decision that determined
it means scanning `decisions.yaml` for an `affects` entry that names it, not reading a field.

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

**YAML form.** `Part.satisfies` on the L2 or L3 part, holding a list of `Requirement` ids.

```yaml
# L2 — the logical blocks that meet the speech requirement
- id: main-pa
  def: audio.part.main_pa
  layer: logical
  satisfies: [r-speech-terrace]

- id: delay-line
  def: audio.part.delay_line
  layer: logical
  satisfies: [r-speech-terrace]
```

**Cardinality.** Many-to-many. One requirement may need several parts to meet it — a speech is intelligible
because of the main hang *and* the delay line — and one part may satisfy several requirements, which is what
makes a piece of equipment worth its truck space.

**Where SysML v2 puts it.** On the satisfying element, not on the requirement: `part def Drone { satisfy
requirement : CargoCapacity; }` binds the requirement's subject to the enclosing part. The standalone form,
`satisfy R1 by vehicle;`, names both ends explicitly. In neither does the requirement hold a list of the
things that meet it, and EventML follows the standard here — see `06-sysml-mapping.md`.

**When the edge is absent.** Read from the requirement's side, nothing in the design meets it yet. That is
not a modelling error; it is the normal state of a requirement captured but not yet designed for, and it is
exactly what question rule 3 in `04-uncertainty.md` fires on. A model is expected to hold requirements in
this state, sometimes for weeks.

Read from the part's side, a part with no `satisfies` is a block or device that nothing in the model asks
for. On an L3 part this is usually harmless, because the requirement is met by the L2 block it is allocated
from and the edge sits there. On an L2 part it is worth attention: a logical block no requirement calls for
is either unnecessary, or evidence of a requirement nobody wrote down.

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

## affect

**YAML form.** `Decision.affects`, holding a list of instance ids and brief paths.

```yaml
- id: d-digital-transport
  date: 2026-08-20
  by: { party: production, person: "system engineer" }
  decision: "Stage inputs reach front of house as Dante over Cat6a, not as an analogue multicore."
  why: "45 m and 32 channels; a multicore that size is heavier and slower to derig than the crew has time for"
  alternatives:
    - { option: "16-way analogue multicore", why_not: "weight and derig time" }
  affects: [sb1, sw-stage, sw-foh]
  src: s-venue-call
```

**Cardinality.** Many-to-many. One decision may determine several elements; one element may be determined by
several decisions.

**Why it sits on the decision.** Two reasons, stated plainly:

- A record must read as a record. A decision whose scope is split across five files cannot be reviewed, and
  reviewing it is the entire purpose of keeping it.
- The backward scan is bounded. Moving `satisfy` in v0.1 avoided scanning 37 requirements; the recording
  criterion in `07-decisions.md` holds decisions to roughly ten per project, and scanning ten entries costs
  nothing.

The second reason is also a check on the first: if a project accumulates as many decisions as requirements,
the fault lies in the recording rather than in the relation.

**When the edge is absent.** A decision with no `affects` determined nothing, which means it is not a
decision. Unlike the other relations, an empty `affects` is a modelling error rather than a question.

**`supersedes`.** Unlike `affects`, it obeys the ordinary rule. It sits on the newer decision, so walking a
decision's history backward stays a field read.

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

The diagram reads downward, but every edge on it is stored at its lower end, so the chain is walked upward
by reading one field at a time. Standing on site holding a wedge, the walk back to the client's own words is
four reads and no searching:

```
wedge_1        .allocate  → monitor-world        (L3 → L2)
monitor-world  .satisfies → r-monitor-coverage   (L2 → L1)
r-monitor-coverage .refines → brief.program[1].size  (L1 → L0)
brief.program[1].size .src → s-client-brief      (L0 → the email itself)
```

This is the only traversal v0.1 supports without a scan, and it is deliberately the one that matters when
somebody asks why a piece of equipment is on the truck. The opposite direction — from a brief sentence
forward to everything it caused — has no stored edge to follow and requires examining every part in the
model. `04-uncertainty.md` needs exactly that direction to compute `blocks`, which is one reason the
question list is hand-written in v0.1.

**The rule that makes the language pay off: any node on this chain that is missing, unknown or unsatisfied
propagates upward into a question, and the number of requirements downstream of it is its rank.** The band's
size is not one question among forty. It is the question that decides monitor count, monitor mixes, input
count, stage box size and stage power — and the model knows that, because those requirements are reachable
from it by following `refine`, `derive` and `satisfy` edges. Counting them is what turns a flat list into
"these three questions decide half the plan".

`04-uncertainty.md` sets out the five conditions that generate a question and how the count is computed.
