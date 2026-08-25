# Example 02 — Annual partner meeting, hotel conference room

120 people in a hotel conference room, presentations in the morning and a panel in the afternoon, streamed
for colleagues who could not travel, with two panellists joining from abroad. Four domains — audio, video,
network and lighting, on power — across all four layers.

> *"Same as last year but the streaming part is new."*

That sentence is the whole example. "Same as last year" refers to a model nobody wrote down, and the
streaming addition changes lighting, camera, network and audio routing — that is, most of it.

## What this adds over example 01

Example 01 covers audio and power, all five value states, and question rules 1 to 4. This one adds:

| | Where |
|---|---|
| **Video, network and lighting** | all three domains appear at L2 and L3 |
| **`derive`** — a requirement decomposing into children | `r-slides-visible` → `r-screen-size`, `r-sightlines` |
| **Two flows sharing one connection** | `f-programme-to-server` and `f-remote-panellists` over `c-switcher-net` |
| **`redundant_over`** — an alternative route | `f-programme-to-server`, via `c-net-server-b` |
| **Question rule 5** — a violated constraint | `video.constraint.display_legibility`, see below |

## The derive chain

`r-slides-visible` says the slides must be legible from every seat. It decomposes into two children, and
they fail differently:

```
brief.program[0]                        L0  "Presentations from the floor, slides throughout"
  └─refine→  r-slides-visible           L1  video.req.image_legibility
               ├─derive→  r-screen-size L1  is the existing screen big enough?      no part satisfies it
               └─derive→  r-sightlines  L1  can every seat see it?                  main-display satisfies it
```

`r-sightlines` is satisfied — one screen at the front, and the L2 block exists. `r-screen-size` is not, and
cannot be: the hotel's screen has never been measured and neither has the distance to the back row. The
parent requirement looks answerable until it is decomposed, which is what `derive` is for.

## Two flows over one connection, in opposite directions

`c-switcher-net` is one Cat6a run from the switcher to the production switch. Over it:

- `f-programme-to-server` — the programme out to the media server, 160 Mbit
- `f-remote-panellists` — the two remote panellists in, ~8 Mbit

Both name `c-switcher-net` in their `over` list, in opposite orders. `r-stream-bandwidth` checks
`network.constraint.bandwidth_headroom` by summing the declared bandwidth of every flow riding each
connection and comparing it against that connection's interface capacity. None of that is possible if a
cable and its payload are one entity.

`f-programme-to-server` also declares a `redundant_over` route through `c-net-server-b`, which shares
`c-switcher-net` with its primary — so it is *not* full redundancy, and
`network.req.network_redundancy` would not be satisfied by it. The model says exactly what is there, which
is what lets the gap be seen.

## The requirement that traces to the stream, not to the lights

`r-face-lighting` refines `brief.program[2]` — the live stream — not any lighting statement. Nobody asked
to be lit. The requirement exists because someone asked what the remote audience would see, and the answer
was "a dark shape at a lectern". That single edge is the argument for L0 → L1 traceability in one line: the
lighting budget on this quote is defensible only because it is traceable to something the client did ask
for.

`r-house-lights` has neither a `refines` edge nor a satisfying part. The venue controls the house lights,
nobody has agreed how they will be dimmed, and it is both a design gap and a negotiation. The model holds
it in that state rather than resolving it prematurely.

## The four-layer walk

```
brief.program[1]                            L0  "Panel discussion, 4 panellists, 2 remote"
  └─refine→  r-remote-participation         L1  video.req.remote_participation
               └─satisfy→  camera-1         L2  video.part.image_source
                             └─allocate→  cam-1     L3  video.part.camera
               └─satisfy→  video-processing L2  video.part.image_processing
                             └─allocate→  switcher  L3  video.part.video_switcher
                             └─allocate→  server    L3  video.part.media_server
                                            └─connect→  c-cam-switcher, c-switcher-net, c-net-server
```

And the signal path, from the camera at the back of the room to the screen at the front:

```
cam-1 ──f-camera-to-switcher──▶ switcher ═switch═▶ switcher:net
                                   └──f-programme-to-server──▶ sw-room ──▶ server
                                                                            ═decode+playback═▶ server:sdi_out[0]
                                                                              └──f-server-to-screen──▶ screen
```

## Which templates this brief triggers

**Hand-derived, like the question list below.** v0.3 has nothing that evaluates `applies_when` —
`spec/02-metamodel.md` says it plainly: "Nothing derives from `applies_when` in v0.3. A human or an agent
reads the sentence against a brief and decides whether the template belongs in the model." What follows is
that reading, done by hand against `brief.yaml`, against all 22 templates in `examples/lib/*/requirements.yaml`.
A later release computes it.

Thirteen templates fire.

| Template | `applies_when` | What in the brief matched |
|---|---|---|
| `audio.req.speech_intelligibility` | the brief has any spoken-word programme item — a speech, a ceremony, a panel discussion | `brief.program[0]` (presentations) and `brief.program[1]` (panel discussion) are both spoken word |
| `audio.req.input_capacity` | anything at all is amplified — this sizes the stage box and the console before either is chosen | presenters, panellists and a lectern all need inputs the moment anything is amplified |
| `audio.req.coverage_uniformity` | the audience area is deep or oddly shaped enough that one position cannot cover it evenly | `brief.venue.room_shape` (assumed) "long and narrow", 120 people theatre-seated (assumed) |
| `video.req.image_legibility` | the audience must read something on a screen — slides, lyrics, captions, scores | `brief.program[0].slides: true`, all morning |
| `video.req.source_switching` | more than one image source reaches the same display | presenter laptops (`brief.program[0].laptop_source`), the camera for the stream, and remote participants all reach the same screen |
| `video.req.screen_coverage` | a screen is used and the room's proportions leave some seats unable to read it — too far off to the side in a wide room, or too far back in a long narrow one | `brief.venue.room_shape` (assumed) "long and narrow" — see the cross-check below, the sentence was fixed to fire on this |
| `video.req.remote_participation` | anyone takes part who is not in the room | `brief.program[1].remote_panellists: 2`, and `brief.remote_audience` for the stream |
| `network.req.bandwidth_headroom` | audio, video or control traffic shares a network link | the stream and the remote panellists' call (`brief.program[2]`) both have to travel over `brief.venue.house_network_available` |
| `network.req.segregation` | two or more domains share network infrastructure | the only network anyone has mentioned is the hotel's guest network (`s-venue-email`); nothing suggests a separate production line |
| `lighting.req.face_lighting` | faces must be seen — by the audience, by a camera, or in photographs afterwards | `brief.program[2]` streams the presenters and panel to a remote audience |
| `lighting.req.house_light_control` | the venue's own lighting affects what the audience or a camera sees | `s-client-call`: "The hotel's AV person handles the house lights" |
| `power.req.supply_capacity` | always | always |
| `power.req.residual_current_protection` | always | always |

Nine do not. A rule that only ever fires teaches nothing about when it applies, so here is why each of
these stays out:

| Template | `applies_when` | Why it does not fire |
|---|---|---|
| `audio.req.music_reproduction` | the brief has live or recorded music the audience is meant to listen to rather than talk over | nothing in `brief.yaml` mentions music of any kind |
| `audio.req.stage_monitoring` | performers play or speak together and need to hear themselves or each other | the panel (`brief.program[1]`) is people talking, seated close together — not performers who need a monitor mix to hear each other |
| `audio.req.weather_protection` | any part of the system stands outside, whether or not the brief mentions rain | the whole event is inside the Danube room; nothing in the brief is outdoors |
| `network.req.clock_stability` | networked audio or video is used — Dante, AES67 or ST 2110 | the brief never says how the stream or the remote link is carried; that is a design-layer choice made later, not something stated in `brief.yaml` |
| `network.req.network_redundancy` | the failure of one link would stop the show rather than degrade it | nothing in the brief claims the stream cannot be interrupted; a dropped stream degrades things for the colleagues who cannot travel, it does not stop the meeting itself |
| `lighting.req.stage_wash` | a stage or performance area is used after dark or in a darkened room | a daytime conference programme, no performance area, no darkened room |
| `lighting.req.control_universes` | more fixtures or parameters are used than one DMX universe carries | no lighting rig is implied beyond the existing house lights and the face lighting above; nothing suggests more than a universe's worth of fixtures |
| `power.req.phase_balance` | the site supply is three-phase | nothing in the brief states the supply is three-phase; only a patch panel is mentioned |
| `power.req.silent_supply` | the supply is a generator, or the programme has quiet passages the audience is meant to hear | the room draws on hotel mains through a patch panel, not a generator, and nothing beyond ordinary speech calls for a quiet passage |

### Cross-check against `requirements.yaml`

Two templates fire with nothing written against them. `audio.req.coverage_uniformity` and
`power.req.residual_current_protection` both apply on the reading above, and neither has a requirement in
`requirements.yaml`. That is the applicability rule doing its job — it found something the modeller missed.
`r-room-power` (`power.req.supply_capacity`) covers the supply's capacity, but nothing in the model checks
that the circuits feeding it are RCD-protected, and nothing checks whether 120 people in a long theatre-
seated room get even coverage from wherever the PA ends up. Both are gaps a later revision of this example
should close, not gaps in the walk.

One template did not fire on its original wording, and a requirement for it already existed:
`r-sightlines` uses `video.req.screen_coverage`, and that requirement's own `notes` in `requirements.yaml`
say exactly why it belongs — "a long narrow room in theatre seating is the case where one screen at the
front stops being enough." But the template's `applies_when` read "the audience area is wider than its
useful viewing angle",
and a *narrow* room is the opposite of a *wide* one: read literally, the brief would not have triggered it.
That is a defect in the sentence, not in `r-sightlines` or in this walk, so the sentence was fixed in
`examples/lib/video/requirements.yaml` to talk about sightlines generally — wide rooms, long narrow ones,
obstructions, or seating split into blocks — rather than only rooms that are wide. With the fix, the
template fires for the reason the requirement was always written for.

The remaining ten requirements in `requirements.yaml` — `r-speech-room`, `r-panel-inputs`,
`r-slides-visible`, `r-screen-size`, `r-remote-participation`, `r-source-switching`, `r-stream-bandwidth`,
`r-stream-segregation`, `r-face-lighting` and `r-house-lights` — all use templates the walk above says fire,
on the brief evidence already cited for each template.

## The question list

**Hand-written, as in example 01.** v0.1 has no traversal engine; this is what the derivation in
`spec/04-uncertainty.md` should produce.

```yaml
questions:
  - ask: "Could someone measure the screen in the Danube room — just the height of the picture area — and pace out how far the back row is from it? A photo of the room from the back would also do."
    why: "video.constraint.display_legibility cannot be evaluated at all: both terms of the ratio are unknown. Blocks r-screen-size, and the answer may mean bringing a bigger screen"
    rule: 5
    source: video.constraint.display_legibility
    blocks: 3        # r-screen-size, r-slides-visible, r-sightlines

  - ask: "Is the Danube room set out in rows facing the front, or around tables? And is it the long narrow one? A photo or the hotel's floor plan settles it."
    why: "decides whether one screen at the front is enough, how far the PA has to throw, and where the camera goes"
    rule: 2
    source: brief.venue.room_shape
    blocks: 3        # r-sightlines, r-speech-room, r-remote-participation

  - ask: "Will the slides have small text, spreadsheets or detailed financial tables on them, or mostly headlines and images?"
    why: "sets the viewing-distance ratio the screen is sized against — 8× image height for detail, 12× for headlines"
    rule: 2
    source: r-slides-visible.params.content_detail
    blocks: 2        # r-slides-visible, r-screen-size

  - ask: "Which platform is the stream going out on — Teams, Zoom, YouTube, something of your own? And roughly how many people will be watching?"
    why: "brief.program[2].platform is unknown; it sets the outbound bitrate that r-stream-bandwidth is checked against"
    rule: 1
    source: brief.program[2].platform
    blocks: 2        # r-stream-bandwidth, r-stream-segregation

  - ask: "Is the panel four people or five? The event brief on 2 October said four, but the update on 21 October said five, one more joining — just want to confirm before we plan seats and microphones."
    why: "brief.program[1].panellists is conflicting; decides panel seating, mic count and the video switcher's input list"
    rule: 4
    source: brief.program[1].panellists
    blocks: 2        # r-panel-inputs, r-remote-participation

  - ask: "Can the room's own lights be dimmed, and can we control them ourselves — or does the hotel's technician need to do it on cue?"
    why: "r-house-lights has no satisfying part and no agreement behind it; undimmed house light washes out the screen"
    rule: 3
    source: r-house-lights
    blocks: 1

  - ask: "Will the presenters use their own laptops, and do you know what connections they have? If anyone is on a Mac we will bring adapters."
    why: "brief.program[0].laptop_source is assumed; decides the scaler's input complement and whether an adapter kit is needed"
    rule: 2
    source: brief.program[0].laptop_source
    blocks: 1        # r-source-switching
```

Note the first question. It is rule 5 — a constraint that cannot be satisfied — and it is ranked first not
because a constraint outranks an unknown, but because three requirements hang off it. The wording asks for
a measurement and a photograph, both of which a hotel event manager can produce in five minutes, rather
than for a screen specification, which they cannot produce at all.
