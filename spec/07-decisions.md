# 07 — Decisions

## 1. What a decision is

A decision records that somebody chose between options that were genuinely open, and that the model now
depends on the choice. It is not a fact about the world, and it is not a guess that fills a gap — it is an
act, taken by a named party on a named date, that the rest of the model treats as settled until somebody
decides again. The concepts and the vocabulary come from ISO/IEC/IEEE 42010's Architecture Decision and its
accompanying Architecture Rationale, adopted rather than reinvented: 42010 defines a decision as the
resolution of an architecture concern among alternatives, together with the rationale that justifies it, and
`Decision` below is that definition written in EventML's terms.

This is worth separating from the two constructs already in the language that decisions resemble. An
`assumed` value, defined in `04-uncertainty.md`, fills a gap in knowledge — nobody has said what the answer
is, so the team supplies one to keep moving, and the value is provisional until the client confirms it. A
`sources` entry, defined in `01-layers.md`, records what somebody said, verbatim, as evidence. A decision is
neither: nobody was missing information, and nobody is quoting a sentence. Somebody looked at real
alternatives and picked one, and the reasoning for the pick is itself the thing being recorded. **An
assumption is resolved by learning, a decision by deciding again.**

## 2. Where decisions live

Decisions live in `decisions.yaml`, a fifth file beside the four layer files. It is not a layer file because
a decision crosses layers by nature — a single choice can touch a brief element, a requirement and a part in
the same breath, and forcing it into one of the four would misrepresent what it is. The per-file version
floor is `eventml: "0.2"`, since `Decision` is new in this release.

```yaml
eventml: "0.2"
kind: decisions
project: garden-party
---
decisions:
  - id: d-digital-transport
    # ...
```

`decisions.yaml` reuses the `kind` header key from library files rather than introducing a new one. `kind`
already answers "what is in this file", as opposed to `layer`, which answers "which layer is this" — and a
decisions file has an answer to the first question and none to the second, so `kind` is the correct key to
reuse even though the file sits among the project's model files rather than among the library's.

## 3. Attributes

| Attribute | Type | Required | Meaning |
|---|---|---|---|
| `id` | string | yes | Instance identifier, `d-` prefixed, no dots |
| `date` | date | yes | When the decision was made |
| `by` | map | yes | `{ party, person }`. `party` is `client` \| `venue` \| `production` \| `authority`; `person` is optional |
| `decision` | string | yes | The choice itself, in one sentence |
| `why` | string | yes | The reasoning |
| `alternatives` | list | no | Each: `option`, `why_not` |
| `affects` | list | yes | Instance ids — `Need` ids included — or brief paths |
| `src` | source ref | no | The `sources` entry the decision rests on |
| `supersedes` | Decision id | no | The earlier decision this one replaces |
| `needs_agreement` | bool | no | `true` when somebody outside the team must assent |
| `agreed_by` | map | no | `{ party, person, src }` |

## 4. `by` and `agreed_by`

`by` and `agreed_by` are what let a single entity carry both an engineering choice and a client agreement,
rather than needing two entities that would drift apart. `by` records who made the decision — often the
production team, exercising professional judgement, with nobody outside the team involved at all. An
engineering choice of this kind has no `agreed_by` and needs none: nothing outside the team has to sign off
on which switch model carries the Dante trunk.

`agreed_by` records something different: that a party outside the team was asked and assented. A decision
carrying `needs_agreement: true` and no `agreed_by` is one nobody has assented to yet, and question rule 7
in `04-uncertainty.md` fires on exactly that gap — a decision the plan depends on that the client (or venue,
or authority) has not actually signed off on. `agreed_by` cites a source rather than asserting agreement:
it carries `src`, pointing at the `sources` entry — an email, a call, a confirmation — where the assent
actually happened, so consent is evidence that can be checked rather than a memory that somebody agreed.

## 5. The recording criterion

ISO/IEC/IEEE 42010 requires a project to define which decisions are worth recording — the standard says a
decision exists whenever a choice is made, but leaves it to the project to decide which choices earn a
place in the record rather than passing unremarked. This section is that definition for EventML. Two tests,
both of which must hold:

1. **A real alternative existed.** Where nothing else could have been chosen, the fact is a derivation and
   `state: derived` already holds it.
2. **Reversal is expensive.** Where undoing it later is free, recording it is noise.

Applying both tests draws a boundary against the constructs already in the language that record other kinds
of thing:

| When | Use |
|---|---|
| Information was missing and we filled the gap | `assumed` value with `why` |
| It follows from another value | `derived` value |
| Somebody said something | `sources` entry |
| Two sources disagree and nobody has resolved it | `conflicting` value |

A decision is the residue left once those four have taken everything they cover: a genuine choice, among
real alternatives, that costs something to reverse. If a project's decision count approaches its requirement
count, the criterion is being applied loosely — most projects have far fewer choices that are both genuinely
open and expensive to undo than they have things the system must achieve. That review signal is what
justifies the `affects` exception in `03-relationships.md`: `affects` may sit at the upper end of its edge,
against the lower-end rule the other five relations keep, precisely because a well-applied criterion keeps
the list short enough to scan.

## 6. Worked example

The garden-party audience decision, showing a `conflicting` value being resolved:

```yaml
- id: d-audience-450
  date: 2026-08-12
  by: { party: production, person: "system engineer" }
  decision: "The system is sized for 450 guests."
  why: "the guest list grew between the two client emails; sizing for the larger number now avoids rebuilding the PA plan later if it holds"
  alternatives:
    - { option: "300 guests", why_not: "superseded by the update of 5 August" }
  affects: [n-audience, main-pa, r-speech-intelligible]
  src: s-client-update
  needs_agreement: true
  agreed_by: { party: client, person: "event manager", src: s-client-confirm }
```

The brief originally said 300 guests, stated and settled. The guest list then grew, the two figures
disagreed for a moment, and `d-audience-450` is the record of how that was resolved: not by treating 450 as
a new stated fact that quietly overwrote the old one, but by recording that somebody decided, on 12 August,
to size the system for the larger number — with the smaller figure kept as the alternative it superseded,
the reasoning attached, and the client's confirmation cited by source rather than assumed.

Because `d-audience-450` names `n-audience` in `affects`, question rule 4 in `04-uncertainty.md` no
longer fires on the value — and since this decision is both `needs_agreement: true` and has an `agreed_by`,
rule 7 does not fire either, so the conflict generates no question at all.
