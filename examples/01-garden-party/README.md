# Example 01 — Garden party, Gellért terrace

A small outdoor event, modelled from the brief quoted in the repository README:

> *"Hi — we're doing a garden party for about 300 people at the Gellért terrace on 12 September. There'll
> be a welcome speech around 7, then a live band later. Nothing fancy, but it should sound good. Can you
> send a quote?"*

Two domains — audio and power, with network underneath the audio — across all four layers. The model is
deliberately unfinished: it is what a project looks like eleven days after the first email, when three
answers are still outstanding and the quote has to go out anyway.

## Files

| File | Layer | Contains |
|---|---|---|
| `brief.yaml` | L0 | What the client and the venue said, with sources |
| `requirements.yaml` | L1 | Nine requirements, two of them unsatisfied |
| `logical.yaml` | L2 | Eight function blocks, ten connections, four flows |
| `physical.yaml` | L3 | Seventeen devices, twenty-two connections, eight flows |

## What this example demonstrates

**All five value states appear in `brief.yaml`.** The load-in time is `stated` with a source; the terrace
being uncovered is `assumed` with `ask: true`; the band's size is `unknown`; the need for a handheld
microphone is `derived` from the absence of a lectern; and the guest count is `conflicting` — 300 in the
first email, 450 in the second, both true when written, and the model refuses to silently pick one.

**Two requirements are unsatisfied, for different reasons.** `r-band-monitoring` cannot be designed because
nobody has said how many musicians there are. `r-weather-protection` could be designed today — the
information exists — but nobody has decided whether the event moves indoors if it rains. Both fire question
rule 3; only one of them is waiting on the client.

**Two requirements have no `refines` edge.** `r-rcd-protection` and `r-input-capacity` came from
professional judgement, not from anything the client said. No client asks for residual current protection.
Making that visible is the point: these are the lines on the quote that will be queried.

**Connection and Flow stay separate, and it pays off twice.** `f-stage-inputs` carries the stage inputs
from `sb1` to `fohm` over three connections; `f-clock` carries PTP the other way over the same three. Two
flows, opposite directions, one set of cables. A merged entity could express neither.

**The item changes along the path, so one path is four flows.** From the speech microphone to the left main
speaker the signal is microphone level, then a Dante flow, then line level, then loudspeaker level. Each
segment is its own `Flow`; they join through the `internal` edges of `sb1`, `fohm` and `amp-pa`. That is
what internal traversability buys — without it the walk stops at the first device it enters.

**`f-fill-chain` walks *through* a loudspeaker.** It runs from the console to `fill-right` over two
connections, passing through `fill-left`'s `in → thru` internal edge. Remove that edge and `fill-right`
appears unfed.

## Layer boundary: where the judgement was needed

`sb1`, `sw-stage` and `sw-foh` allocate to `mix-position`, which is not obvious. There is no transport block
at L2, because at L2 nothing has decided how the stage inputs reach front of house — a 16-way analogue
multicore would satisfy the same logical connection between `microphone-pool` and `mix-position` just as
well as the digital path chosen at L3 (`d-digital-transport` in `decisions.yaml` has the reasoning). What
the layer boundary has to explain is not that choice but its consequence: the stage box and the switches
exist only because that choice was made, and they exist to serve the mix position, so that is what they
are allocated to.

`power-supply` has no L3 allocation at all. Nobody knows what the terrace socket is rated at, so no supply
has been chosen. An L2 block with no L3 part underneath it is legitimate and is the visible form of an
undecided design — the inverse of boundary rule 3, which forbids the opposite case.

Four power connections run from schuko outlets into TRUE1 inlets. Both ports carry `power.item.ac_230v_1p`,
so the connections are valid; the connector difference is simply what the ends of those cables are. A model
that matched connectors rather than items could not express an adapter lead.

## The four-layer walk

Every requirement must be walkable from a brief element down to a cable. Taking `r-speech-intelligible`:

```
brief.program[1]                          L0  "Welcome speech, 19:00"
  └─refine→  r-speech-intelligible        L1  audio.req.speech_intelligibility
               └─satisfy→  main-pa        L2  audio.part.main_pa
                             └─allocate→  amp-pa    L3  audio.part.amplifier
                             └─allocate→  pa-left   L3  audio.part.loudspeaker
                             └─allocate→  pa-right  L3  audio.part.loudspeaker
                                            └─connect→  c-amp-pa-l  (amp-pa:spk_out[0] → pa-left:spk_in)
                                            └─connect→  c-amp-pa-r  (amp-pa:spk_out[1] → pa-right:spk_in)
```

And the signal path that serves it, from the microphone in the speaker's hand to the left main speaker,
crossing three devices through their internal edges:

```
vox1 ──f-vox-to-sb1──▶ sb1 ═adc═▶ sb1:net
                         └──f-stage-inputs──▶ sw-stage ──▶ sw-foh ──▶ fohm
                                                                       ═mix═▶ fohm:analog_out[0]
                                                                         └──f-pa-drive-left──▶ amp-pa
                                                                                                ═amplify═▶
                                                                                  └──f-amp-to-pa-left──▶ pa-left
```

`══▶` is an internal edge inside a part; `──▶` is a flow over connections.

## The decisions

`decisions.yaml` holds three decisions, each demonstrating a different part of `spec/07-decisions.md`.

`d-audience-450` is a client agreement resolving a `conflicting` value. `brief.audience` stays
`conflicting` — the model does not turn 450 into a new `stated` fact, because both 300 and 450 were true
when written. What changes is that the plan now depends on 450, and this decision is the record of who
chose that number, when, and on what evidence: the client's update email as `src`, and the client's
confirmation cited by `agreed_by.src` rather than assumed.

`d-digital-transport` is an engineering choice with no `agreed_by` — nothing outside the team needed to
sign off on how the stage inputs reach front of house. It is also the sentence this README used to carry
in prose, in the "Layer boundary" section above: the stage box and both switches exist only because a
digital transport was chosen at L3, and nothing at L2 records that a digital transport exists at all. That
sentence is now a `Decision`, not a paragraph — the model holds it, the README just points at it.

`d-no-cover` is a decision nobody has agreed to. It carries `needs_agreement: true` and no `agreed_by`,
so question rule 7 in `spec/04-uncertainty.md` fires on it: the client has not been told that a stage cover
is out of scope, which means an outdoor event's weather risk is being carried by the client without their
knowledge. See the question list below.

## The question list

**This list is hand-written.** v0.1 has no validator and no traversal engine, so nothing here was computed
— it is what the derivation defined in `spec/04-uncertainty.md` should produce from this model, written out
by hand to show the intended output. v0.2 derives it. Where this list and the model disagree, the model is
right and this list is stale.

Ranked by `blocks` — the number of requirements reachable downstream of the open node.

```yaml
questions:
  - ask: "Is the terrace covered, or fully open to the sky? If it rains that evening, does the party carry on outside, move indoors, or get called off?"
    why: "decides PA type, weather protection, power routing and whether we carry a cover budget"
    rule: 2
    source: brief.venue.covered
    blocks: 4        # r-weather-protection, r-music-reproduction, r-supply-capacity, r-rcd-protection

  - ask: "How many musicians are in the band, and what does each of them play? If they have a rider or a stage plan, that answers it completely."
    why: "decides input count, monitor mixes, stage box size and stage power"
    rule: 1
    source: brief.program[3].size
    blocks: 3        # r-band-monitoring, r-input-capacity, r-supply-capacity

  - ask: "Can you send a photo of the outdoor socket by the service door — and of the fuse box it comes from, if you can find it? Also roughly how many paces it is from there to where the band will play."
    why: "brief.power.rating is unknown; everything downstream of the supply is provisional until it is answered"
    rule: 1
    source: brief.power.rating
    blocks: 2        # r-supply-capacity, r-silent-supply

  - ask: "Nothing in the plan yet lets the band hear themselves on stage."
    why: "r-band-monitoring has no satisfying part; blocked on the band's size, so it cannot be designed yet"
    rule: 3
    source: r-band-monitoring
    blocks: 1
    internal: true   # for the project manager, not for the client

  - ask: "How late can the music run? We have assumed 23:00, and there is usually a municipal limit on outdoor amplified sound from 22:00."
    why: "decides whether the band's second set needs a level cap, and whether to warn the client now"
    rule: 2
    source: brief.program[3].until
    blocks: 1        # r-music-reproduction

  - ask: "The plan includes a mix position, a playback source and a monitor world, and nothing in the brief or the requirements asks for any of them."
    why: "three L2 blocks satisfy no requirement; playback and the mix position are billable and unjustified, and monitor-world is the same gap as r-band-monitoring seen from the other end"
    rule: 6
    source: [mix-position, playback, monitor-world]
    blocks: 0
    internal: true   # for the project manager, not for the client

  - ask: "Can you confirm we are not carrying a cover? If it rains on the 12th the terrace has nowhere to go, and we would rather agree now what happens than decide it on the night."
    why: "d-no-cover is unagreed; it moves a weather risk onto the client without their knowledge"
    rule: 7
    source: d-no-cover
    blocks: 0
    internal: true   # for the project manager, not for the client
```

Seven questions, and the first two decide most of the plan. Forty things in this model are unspecified;
these are the ones with requirements hanging off them.

Note what rule 2 does in the first question. Nobody stated that the terrace is uncovered — it was assumed
from the word "terrace" — and the person who made that assumption marked it `ask: true` because they knew
it mattered. The ranking then puts it first. That judgement is recorded at the point it was made, by the
only person in a position to make it, and the model carries it forward.

Note also the wording. Not "what is the service rating of the supply", which no client can answer, but
"send a photo of the socket and the fuse box" — which settles the question completely and which anyone can
do while standing in the venue.

The last question is rule 6, and `monitor-world` is why the rule earns its place. That block appears twice
in this list: once under rule 3, because `r-band-monitoring` is satisfied by nothing, and once under rule 6,
because the block satisfies nothing. It is one gap seen from both ends — a requirement and a block that
ought to be joined and are not, because nobody knows how big the band is. Rule 3 alone would report the
requirement and leave the block looking deliberate. `mix-position` and `playback` are the other kind: no
requirement anywhere refers to them, and they are on the quote regardless.
