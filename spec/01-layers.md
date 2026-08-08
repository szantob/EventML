# 01 — Layers

EventML spans four layers, following the [ARCADIA](https://mbse-capella.org/arcadia.html) method. Each layer
answers a different question about the same event, at a different distance from the client's own language.

| Layer | `layer:` value | ARCADIA phase | Content |
|---|---|---|---|
| L0 Brief | `brief` | Operational Analysis | What the client and audience do, in lay language |
| L1 Requirement | `requirement` | System Analysis | What the system must achieve, technology-independent |
| L2 Logical | `logical` | Logical Architecture | Function blocks and signal paths, no technology chosen |
| L3 Physical | `physical` | Physical Architecture | Concrete devices, ports, connections |

The `layer:` value is the literal string written in the header of every YAML file that models something at
that layer. It is written exactly as it appears in the table above: lowercase, singular.

## L0 Brief

**What belongs here.** Free-form structured values describing what the client and the audience do, in the
client's own terms: attendee counts, schedule items, venue description, stated preferences. This is the
layer closest to the source conversation — a note, an email, a phone call.

**What does not belong here.** Anything phrased as a system property or measured against a technical
standard. L0 records what people do, not what the system must achieve.

**Which entities may appear.** Free-form structured values only. No `Part`, no `Connection`, no `Flow` — L0
has no notion of equipment or signal path at all.

**Worked examples.**

- Well-formed: "300 guests at a garden party on the Gellért terrace, welcome speech at 19:00, live band from
  21:00."
- Belongs in L1 instead: "the room must support simultaneous interpretation into three languages." This is a
  measurable statement of what the system must achieve, not a description of what the client or audience do,
  so it belongs in L1 Requirement.

### Sources

L0 carries one construct no other layer has: `sources`, a reserved key in the `brief` document holding the
communications the brief was built from. Each entry records one statement — an email, a phone call, a
document, a site visit — with its date, the party that made it, and the text itself, quoted rather than
paraphrased.

```yaml
brief:
  sources:
    - id: s-client-brief
      kind: email
      date: 2026-08-01
      from: client
      subject: "Garden party, 12 September — quote request"
      excerpt: >-
        Hi — we're doing a garden party for about 300 people at the Gellért terrace on 12 September.
        There'll be a welcome speech around 7, then a live band later.
```

| Key | Required | Meaning |
|---|---|---|
| `id` | yes | The handle `src:` references. An instance ID: short, lowercase, no dots |
| `kind` | yes | `email` \| `call` \| `document` \| `site_visit` |
| `date` | yes | When the statement was made, ISO 8601 |
| `from` | yes | The party that made it: `client` \| `venue` \| `production` \| `authority` |
| `sender` | no | The individual, where which person said it matters |
| `subject` | for `email` and `document` | The subject line or title |
| `excerpt` | yes | The text, verbatim |

Sources exist because `trace` needs somewhere to land. Every `src:` anywhere in a model names one of these
entries, so the chain from a cable back to the sentence that caused it ends in words the client actually
wrote — `03-relationships.md` walks that chain. A paraphrase would break it: it puts the modeller's reading
of the brief where the brief itself should be, and the disagreement that surfaces three weeks later has
nothing to be adjudicated against.

**Each fact gets its own key.** The date is not encoded in the `id`, and the person is not appended to the
`src` string that points at the entry. An identifier is a handle, not a record. Two places holding the same
fact eventually disagree, and nothing in v0.1 would catch it; worse, the `conflicting` state resolves by
asking which statement came later, so it needs a date it can compare rather than one it has to parse out of
a name.

**One entry per statement, not per meeting.** A site visit at which three people say three things is three
sources sharing a date, not one source with three sentences in its `excerpt`. The test is whether any value
would ever need to cite one of them alone: a `conflicting` value whose alternatives both cite the same
entry cannot say who said what, and question rule 4 — which puts the disagreement back as a choice between
two attributed statements — has nothing to work with. `examples/03-festival-stage/brief.yaml` carries the
worked case, where a site manager and a council representative contradict each other on the same
afternoon.

## L1 Requirement

**What belongs here.** Statements of what the system must achieve: measurable, technology-independent, each
one traceable back to an L0 element it refines.

**What does not belong here.** Lay narrative of what the client or audience do belongs in L0, not here.
Naming a function block, a signal path, or a device belongs in L2 or L3 — a requirement says what must be
true, never how it is achieved.

**Which entities may appear.** `Requirement` only.

**Worked examples.**

- Well-formed: "in the 200-seat conference room, every delegate must be able to hear the chair without
  leaning forward."
- Belongs in L2 instead: "the PA needs a main hang plus a delay line for the far tables." This names function
  blocks — main PA, delay line — which is a logical architecture decision, so it belongs in L2 Logical.

## L2 Logical

**What belongs here.** Function blocks and their signal paths, with no technology chosen yet: what talks to
what, and over what kind of link, described in terms generic enough to survive a change of manufacturer.

**What does not belong here.** No product names or model numbers — see boundary rule 2 below. No bare
requirement statements without an associated block, and no lay narrative.

**Which entities may appear.** `Part`, `Port`, `Connection`, `Flow` — all referencing technology-neutral
`PartDef`s.

**Worked examples.**

- Well-formed: "the interpretation booth's audio block connects to the delegate receiver block over an RF
  link."
- Belongs in L3 instead: "the main PA is a pair of d&b Y-series tops flown left and right." Naming a
  manufacturer and model is a concrete device choice, so it belongs in L3 Physical.

## L3 Physical

**What belongs here.** Concrete devices, their ports, and the connections between them — the layer that
answers "what do we actually rent and cable."

**What does not belong here.** No function block that lacks an L2 counterpart: every L3 `Part` must trace
back via `allocate` to an L2 `Part` — see boundary rule 3 below. No lay narrative or bare requirements.

**Which entities may appear.** `Part`, `Port`, `Connection`, `Flow` — referencing concrete `PartDef`s, each
`allocate`d from an L2 `Part`.

**Worked examples.**

- Well-formed: "stage box `sb1` connects to the mixer console over Dante on Cat6a, `sb1:dante[0]` to
  `console:dante[0]`."
- Belongs in L2 instead: "an interpretation booth function block sits between the chair mic and the delegate
  receivers." This names a function block with no product attached, so it belongs in L2 Logical.

## Boundary rules

1. **A layer never references downward.** A `Requirement` does not name a device. Downward association is
   expressed by `satisfy` and `allocate` edges, which belong to the lower layer — this keeps an L1
   requirement valid regardless of which L2 or L3 design ends up meeting it.
2. **L2 carries no product names.** If removing the manufacturer would change the meaning of a statement, it
   belongs in L3, not L2 — this is what keeps the logical architecture reusable across different equipment
   choices.
3. **Every L3 `Part` is allocated from exactly one L2 `Part`.** An unallocated L3 part is a modelling error,
   and question rule 3 will surface it — this is what keeps the physical design traceable back to a logical
   decision instead of appearing out of nowhere.
