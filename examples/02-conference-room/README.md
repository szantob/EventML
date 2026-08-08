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
               ├─derive→  r-screen-size L1  is the existing screen big enough?      satisfied_by: []
               └─derive→  r-sightlines  L1  can every seat see it?                  satisfied_by: [main-display]
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
