# 04 — Uncertainty

The client is not an expert, and the brief that starts a project is always incomplete. EventML treats that
incompleteness as data rather than as an error to be cleared before modelling begins. This file defines how
a value records what is known about it, and how the model's gaps become a ranked list of questions.

## 1. Progressive wrapping

**A bare scalar means the client stated it; a wrapped object means anything else.**

That is the whole rule. The common case — a value the client gave — costs nothing to write and reads like
ordinary YAML. Everything else expands into an object that says what kind of knowledge it is.

| Attribute | Type | Required | Meaning |
|---|---|---|---|
| `value` | any | no | Absent when `state: unknown` |
| `state` | see §2 | yes | What kind of knowledge this is |
| `why` | string | when `assumed` or `derived` | The reasoning |
| `src` | source ref | when `stated` or `conflicting` | The `sources` entry it came from — see `01-layers.md` |
| `ask` | bool | no | Force this into the question list |
| `alternatives` | list | when `conflicting` | The competing values with their sources |

Both forms side by side:

```yaml
audience: 300                       # stated

venue_type:                         # everything else
  value: outdoor
  state: assumed
  why: "the brief says 'terrace'"
  ask: true
```

The design is deliberately lopsided. A brief that has been well answered is mostly bare scalars and reads
like a form; a brief that has just arrived is mostly wrappers and reads like a list of things to find out.
The shape of the file tells you how far along the project is before you have read a word of it.

Wrapping applies to any value anywhere in a model — a brief field, a `Requirement.params` entry, a
`Part.properties` entry, a `Connection` length. It is a property of values, not of any one layer.

## 2. The five states

| State | Meaning | Requires |
|---|---|---|
| `stated` | The client said it | `src` |
| `derived` | Follows from a rule or another value | `why` |
| `assumed` | We supplied it to keep moving | `why` |
| `unknown` | Missing, and we know it is missing | — |
| `conflicting` | Two sources disagree | `alternatives` |

**A bare scalar is shorthand for `state: stated`.** The two forms are identical in meaning; the wrapped form
merely adds room for a `src`. Writing `audience: 300` and writing `audience: { value: 300, state: stated }`
say exactly the same thing about the model, and either may be used wherever the other is.

The distinction that does the most work is `assumed` versus `derived`. An assumption is a decision the team
made in the absence of information — it could be wrong, and the client may need to confirm it. A derived
value follows necessarily from something already in the model, and confirming it with the client would be
pointless; what would change it is a change to its input. Conflating the two produces question lists full of
things no client can usefully answer.

```yaml
band:
  size:
    state: unknown                        # nobody has said how many musicians

  input_count:
    value: 12
    state: derived
    why: "four musicians at three inputs each, per the default in audio.req.band_inputs"

  stage_area_m2:
    value: 24
    state: assumed
    why: "typical five-piece footprint; no stage plan received"
    ask: true

  load_in_time:
    state: conflicting
    alternatives:
      - { value: "16:00", src: s-client-brief }
      - { value: "14:00", src: s-venue-call }
```

## 3. Question rules

A question is generated in exactly five situations. Each generated question carries three fields:

- `ask` — the client-facing wording, written for someone who does not know the domain
- `why` — what the answer decides, in the project manager's terms
- `blocks` — the number of requirements downstream of the open node

#### Rule 1 — an `unknown` value a requirement depends on

**Trigger.** A value with `state: unknown` that is reachable from at least one `Requirement`, whether as a
parameter of that requirement or as a brief element it refines.

An unknown nothing depends on is not a question. Half the fields in any brief are unknown and irrelevant —
nobody needs the caterer's van registration — and a list that asks about all of them is worse than no list.
The dependency is what makes it worth the client's attention.

```yaml
- ask: "How many musicians are in the band, and what does each of them play?"
  why: "decides input count, monitor mixes and stage box size"
  blocks: 3
```

#### Rule 2 — an `assumed` value marked `ask: true`

**Trigger.** A value with `state: assumed` and `ask: true`.

Not every assumption needs confirming. A default sample rate is an assumption nobody wants to be asked
about; whether the terrace is covered is an assumption that changes the plan. `ask: true` is how the person
who made the assumption says which kind it was — the judgement is recorded at the point where it is made,
by the only person in a position to make it.

```yaml
- ask: "Is the terrace covered, or fully open to the sky?"
  why: "decides PA type, weather protection and power routing"
  blocks: 4
```

#### Rule 3 — a `Requirement` no `Part` satisfies

**Trigger.** A `Requirement` whose id appears in no part's `satisfies` list.

This one is not addressed to the client. It is the model telling the project manager that something has been
promised and nothing yet delivers it — the design gap, as opposed to the information gap. It appears in the
same ranked list because it competes for the same attention, but its `ask` is phrased for internal reading.

```yaml
- ask: "Nothing yet covers the requirement that the band can hear itself on stage."
  why: "monitor world not designed; blocks the stage plan and the input list"
  blocks: 2
```

#### Rule 4 — a `conflicting` value

**Trigger.** A value with `state: conflicting`.

Two sources disagree, and the model refuses to pick a winner silently. Both alternatives stay in the file
with their sources attached, so the question can be put back to the client as a choice between two things
they themselves said, with dates.

```yaml
- ask: "Is the band's load-in at 14:00 or 16:00? The brief says 16:00, but the production manager said 14:00 on the phone on 5 August."
  why: "decides crew call time and whether soundcheck fits before doors"
  blocks: 1
```

#### Rule 5 — a violated `ConstraintDef`

**Trigger.** A `ConstraintDef` whose `expression` does not hold for the model.

It does not fit, or it will not carry the load. In v0.1 there is no validator, so this rule fires when a
human or an agent applies the constraint by hand — see the note in `02-metamodel.md` on why `expression` is
prose in this release.

```yaml
- ask: "The run from the stage box to front of house measures 120 m, but Cat6a is limited to 90 m. Can we place a switch mid-way, or should this leg go on fibre?"
  why: "network.constraint.max_copper_run violated on c-swstage-swfoh"
  blocks: 2
```

## 4. Ranking

`blocks` is computed by counting the `Requirement`s reachable downstream of the open node by following
`refine`, `derive` and `satisfy` edges. Questions sort by `blocks` descending.

This is the payoff that justifies the cost of four layers and five traceability relations. Without the
graph, a model's gaps are a flat list of forty items in the order somebody happened to write them, and the
client answers the first five and stops. With it, the list opens on the questions that decide the most:

```yaml
questions:
  - ask: "Is the terrace covered, or fully open to the sky?"
    why: "decides PA type, weather protection and power routing"
    blocks: 4
  - ask: "How many musicians are in the band, and what does each of them play?"
    why: "decides input count, monitor mixes and stage box size"
    blocks: 3
  - ask: "Where exactly will people be standing or sitting during the speeches?"
    why: "decides coverage, delay line placement and front-fill"
    blocks: 2
```

Three questions instead of forty, and the first two decide half the plan.

**In v0.1 this list is written by hand.** The examples under `examples/` contain hand-written `questions`
blocks that demonstrate the intended output of the derivation. They are illustrations, not derived
artefacts: v0.1 has no validator and no traversal engine, and the rule that the question list is derived
rather than authored cannot execute until v0.2. Where a hand-written list and the model disagree, the model
is right and the list is stale.

## 5. Client language

The `ask` string is the only part of the model a client ever reads, and it is the point of the whole
project. It is written for someone who does not know the domain — no jargon, no acronym, and no question
whose answer requires knowing what the answer is for.

| Written for us | Written for the client | What the rewrite fixes |
|---|---|---|
| "Do you need front-fill?" | "Will anyone be standing right at the front, close to the stage?" | Asks about the room, not about the equipment. The client knows where people stand; they have never heard of front-fill |
| "Is three-phase available on site?" | "Is there a large industrial power socket near the terrace — the round blue or red kind, bigger than a household plug? A photo of it would be ideal." | Replaces a term of art with something visible, and asks for evidence rather than a judgement the client is not qualified to make |
| "Confirm the SPL target for the band." | "Should the band be at background-music level, or a proper concert volume people can feel?" | Turns a number nobody outside the industry can estimate into a choice between two experiences the client can picture |

The pattern in all three: **ask about the world, not about the system.** The client is an expert on their own
event — who comes, where they stand, what it should feel like — and knows nothing about the equipment that
delivers it. Every question that crosses that line comes back either unanswered or, worse, answered wrongly
by someone being helpful.
