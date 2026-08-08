# 02 — Metamodel

## 1. Definitions and usages

EventML separates **definitions** from **usages**, a split taken directly from SysML v2's `def`/`usage`
pattern. A definition is a reusable catalogue type: the concept of a digital stage box, of an XLR 3-pin
female socket, of balanced line-level audio. It describes what something *is*, independent of any event.
Definitions live in `lib/` and carry catalogue IDs of the form `<domain>.<kind>.<name>`.

A usage is one occurrence of that type in one project: the stage box standing at the drum riser on 12
September, with the label the crew wrote on its case. Usages live in model files under `examples/` or in a
project's own repository. A usage names its type under the key `def:` and carries its own identifier under
`id:`. These two keys are never inferred from one another — a model may hold six stage boxes, all with
`def: audio.part.stage_box` and six distinct `id:` values.

The split matters here for a practical reason. The catalogue is shared across every project and maps onto
rental inventory: `audio.part.stage_box` is the row in the hire system, and a `catalog_ref` on the
definition is what binds the model to it. Instances are per-event and disposable — they are created when a
brief arrives and discarded when the truck comes back. Keeping the two apart is what lets the vocabulary
accumulate while individual models stay small.

## 2. Definition entities

Six entities are definitions. All of them may appear in `lib/`; none of them may appear in a model file.

### ItemDef

**Purpose.** Defines what flows through the system — a signal, a power feed, a data stream.

| Attribute | Type | Required | Meaning |
|---|---|---|---|
| `id` | string | yes | Catalogue ID |
| `name` | string | yes | Human-readable name |
| `kind` | `signal` \| `power` \| `data` | yes | Broad class |
| `form` | `analog` \| `digital` \| `optical` \| `electrical` | yes | Physical realisation |
| `carries` | string | yes | What it conveys, in prose |
| `properties` | map | no | Domain-specific attributes |
| `source` | string | no | Standard the term is adopted from |
| `notes` | string | no | Free text |

**Relations.** Referenced by `PortDef.item`, `InterfaceDef.item` and `Flow.item`. An `ItemDef` references
nothing itself — it is the bottom of the type graph.

```yaml
- id: audio.item.analog_line
  name: Analogue line level
  kind: signal
  form: analog
  carries: One channel of balanced line-level audio
  properties:
    nominal_level: "+4 dBu"
    balanced: true
  source: "GDTF SignalType: AnalogAudio"
```

### PortDef

**Purpose.** Defines a connection point: what passes through it, in which direction, and through which
physical connector.

| Attribute | Type | Required | Meaning |
|---|---|---|---|
| `id` | string | yes | Catalogue ID |
| `name` | string | yes | Human-readable name |
| `item` | ItemDef ref | yes | What passes through |
| `direction` | `source` \| `sink` \| `bidir` | yes | Flow direction, IFC semantics |
| `connector` | string | yes | Physical connector |
| `pins` | int | no | Pin count where meaningful |
| `properties` | map | no | Domain-specific attributes |
| `source` | string | no | Standard the term is adopted from |

**Relations.** References one `ItemDef`. Referenced by `PartDef.ports[].def`.

The direction vocabulary is IFC4's, not an invention: `IfcDistributionPort.FlowDirection` takes `SOURCE`,
`SINK` and `SOURCEANDSINK`, which EventML writes as `source`, `sink` and `bidir`. Direction is stated from
the perspective of the part that owns the port: a `sink` port receives, a `source` port emits.

```yaml
- id: audio.port.xlr3f
  name: XLR 3-pin female
  item: audio.item.analog_line
  direction: sink
  connector: XLR3F
  pins: 3
  source: "IFC IfcDistributionPort FlowDirection=SINK"
```

### PartDef

**Purpose.** Defines a device type or, when `abstract` is true, a technology-neutral function block: its
ports, its internal traversability, and its properties.

| Attribute | Type | Required | Meaning |
|---|---|---|---|
| `id` | string | yes | Catalogue ID |
| `name` | string | yes | Human-readable name |
| `category` | string | yes | Broad class, e.g. `mixer`, `loudspeaker`, `distro` |
| `abstract` | bool | no | `true` for technology-neutral L2 blocks; default `false` |
| `ports` | list | yes | Each: `id`, `def`, `count`, optional `role` |
| `internal` | list | no | Each: `from`, `to`, optional `transform` — internal traversability |
| `properties` | map | no | Domain-specific attributes |
| `catalog_ref` | map | no | External system IDs, e.g. `{ rentman: 4417 }` |
| `source` | string | no | Standard the term is adopted from |

**Relations.** References `PortDef`s through `ports[].def`. Referenced by `Part.def`. An `abstract` part
definition is what an L2 `Part` uses; a concrete one is what an L3 `Part` uses.

The `ports[].id` is local to the definition and is the name used in port references: with the definition
below, a usage `sb1` exposes `sb1:analog_in[0]` through `sb1:analog_in[15]`, and `sb1:pwr` without an index
because its count is 1.

```yaml
- id: audio.part.stage_box
  name: Digital stage box
  category: stage_box
  ports:
    - { id: analog_in,  def: audio.port.xlr3f,          count: 16 }
    - { id: analog_out, def: audio.port.xlr3m,          count: 8 }
    - { id: net,        def: network.port.ethercon_a,   count: 2, role: primary_secondary }
    - { id: pwr,        def: power.port.powercon_true1, count: 1 }
  internal:
    - { from: "analog_in[*]", to: net,             transform: adc }
    - { from: net,            to: "analog_out[*]", transform: dac }
  properties:
    electrical_load_w: 100
```

### InterfaceDef

**Purpose.** Defines a link type between two ports: what medium realises it, how much it can carry, and
what rules it must obey.

| Attribute | Type | Required | Meaning |
|---|---|---|---|
| `id` | string | yes | Catalogue ID |
| `name` | string | yes | Human-readable name |
| `item` | ItemDef ref | yes | What the link carries |
| `medium` | string | yes | e.g. `cat6a`, `xlr_cable`, `h07rn-f_3g2.5` |
| `capacity` | map | no | e.g. `{ channels: 64, mbit: 1000 }` |
| `constraints` | list of ConstraintDef refs | no | Rules the link must satisfy |

**Relations.** References one `ItemDef` and any number of `ConstraintDef`s. Referenced by
`Connection.interface`.

A `PortDef` says what a socket accepts; an `InterfaceDef` says what the cable between two sockets can do.
The distinction earns its keep at capacity: two Ethercon sockets are identical whether the link between
them is a 3-metre patch lead or a 90-metre run at the edge of the standard, and only the interface knows
which.

```yaml
- id: network.interface.cat6a_link
  name: Cat6a copper link
  item: network.item.ethernet_frame
  medium: cat6a
  capacity:
    mbit: 1000
    max_length_m: 90
  constraints:
    - network.constraint.max_copper_run
```

### RequirementDef

**Purpose.** Defines a parameterised requirement template, together with how it would be verified and what
to ask the client when a parameter is missing.

| Attribute | Type | Required | Meaning |
|---|---|---|---|
| `id` | string | yes | Catalogue ID |
| `name` | string | yes | Human-readable name |
| `text` | string | yes | Template with `{param}` placeholders |
| `params` | list | yes | Each: `name`, `type`, optional `default` |
| `verification` | string | yes | How you would check it was met |
| `constraints` | list of ConstraintDef refs | no | Rules that must hold for it to be met |
| `asks` | list | no | Client-facing question templates for unfilled params |

**Relations.** References any number of `ConstraintDef`s. Referenced by `Requirement.def`.

The `asks` list is what makes the derived question list speak the client's language rather than the
model's. Each entry binds a parameter to a question a non-expert can answer; `04-uncertainty.md` sets out
when those questions fire.

```yaml
- id: audio.req.speech_intelligibility
  name: Speech intelligibility
  text: "Speech must be intelligible across {area} for an audience of {audience}."
  params:
    - { name: area,     type: string }
    - { name: audience, type: int }
  verification: "Coverage prediction, or measured STI at the three worst listening positions."
  asks:
    - param: area
      question: "Where exactly will people be standing or sitting during the speeches?"
```

### ConstraintDef

**Purpose.** Defines a computable rule — a load limit, a channel count, a cable length — that a model can
be checked against.

| Attribute | Type | Required | Meaning |
|---|---|---|---|
| `id` | string | yes | Catalogue ID |
| `name` | string | yes | Human-readable name |
| `applies_to` | string | yes | Which entity kind it constrains |
| `expression` | string | yes | Prose statement of the rule |
| `params` | list | no | Each: `name`, `type`, optional `default` |

**Relations.** Referenced by `InterfaceDef.constraints` and `RequirementDef.constraints`.

**In v0.1 the expression is prose, not a formal expression language.** There is no validator to evaluate
it, and inventing a syntax nothing executes would freeze a bad guess into the kernel. Formalising
`expression` is the first task of v0.2; until then it is written so that a human or an agent reading the
model can apply it by hand.

```yaml
- id: network.constraint.max_copper_run
  name: Maximum copper run
  applies_to: Connection
  expression: "The connection's length_m must not exceed the interface's capacity.max_length_m."
```

## 3. Usage entities

Four entities are usages. All of them appear in model files; none of them may appear in `lib/`.

### Part

**Purpose.** One occurrence of a `PartDef` in a project — a device on the truck, or a function block in the
logical architecture.

| Attribute | Type | Required | Meaning |
|---|---|---|---|
| `id` | string | yes | Instance identifier, unique within the model |
| `def` | PartDef ref | yes | The catalogue type |
| `label` | string | no | What the crew calls it |
| `layer` | `logical` \| `physical` | yes | Which layer this usage belongs to |
| `properties` | map | no | Overrides the definition's `properties` |
| `allocate` | Part id | L3 only | The L2 `Part` this one realises |

**Relations.** References one `PartDef`. An L3 `Part` references exactly one L2 `Part` through `allocate`
— boundary rule 3 in `01-layers.md`. Referenced by `Connection` port references, `Flow.from` and `Flow.to`,
and `Requirement.satisfied_by`.

```yaml
- id: sb1
  def: audio.part.stage_box
  label: "Stage box — drum riser"
  layer: physical
  allocate: stage-input-block
  properties:
    electrical_load_w: 90
```

### Connection

**Purpose.** States that a physical link exists between two ports.

| Attribute | Type | Required | Meaning |
|---|---|---|---|
| `id` | string | yes | Instance identifier |
| `from` | port ref | yes | `<part_id>:<port_id>[<index>]` |
| `to` | port ref | yes | `<part_id>:<port_id>[<index>]` |
| `interface` | InterfaceDef ref | yes | The link type |
| `redundancy` | `primary` \| `secondary` \| `none` | no | Role in a redundant pair; default `none` |
| `properties` | map | no | e.g. `length_m` |

**Relations.** References two ports, each belonging to a `Part`, and one `InterfaceDef`. Referenced by
`Flow.over` and `Flow.redundant_over`.

A `Connection` says only that the cable exists. It does not say what runs on it — that is `Flow`.

```yaml
- id: c-sb1-swstage
  from: "sb1:net[0]"
  to: "sw-stage:port[0]"
  interface: network.interface.cat6a_link
  redundancy: primary
  properties:
    length_m: 12
```

### Flow

**Purpose.** States what travels through the system, from where to where, and over which chain of
connections.

| Attribute | Type | Required | Meaning |
|---|---|---|---|
| `id` | string | yes | Instance identifier |
| `item` | ItemDef ref | yes | What travels |
| `from` | Part id | yes | Origin part |
| `to` | Part id | yes | Destination part |
| `over` | ordered list of Connection ids | yes | The route, in order from `from` to `to` |
| `redundant_over` | list of Connection ids | no | An alternative route serving the same flow |
| `properties` | map | no | e.g. `channels` |

**Relations.** References one `ItemDef`, two `Part`s, and an ordered list of `Connection`s.

```yaml
- id: f-stage-inputs
  item: audio.item.dante_flow
  from: sb1
  to: fohm
  over: [c-sb1-swstage, c-swstage-swfoh, c-swfoh-fohm]
  redundant_over: [c-sb1-swstage-b, c-swstage-swfoh-b, c-swfoh-fohm-b]
  properties:
    channels: 32
```

### Requirement

**Purpose.** One occurrence of a `RequirementDef` in a project, with its parameters filled in and its
traceability recorded.

| Attribute | Type | Required | Meaning |
|---|---|---|---|
| `id` | string | yes | Instance identifier |
| `def` | RequirementDef ref | yes | The template |
| `params` | map | yes | Fills the template's placeholders |
| `refines` | L0 path | no | The brief element this requirement refines |
| `derived_from` | Requirement id | no | The parent requirement, when decomposed |
| `satisfied_by` | list of Part ids | no | What meets it — **may be empty** |

**Relations.** References one `RequirementDef`, optionally one L0 brief element, optionally one parent
`Requirement`, and any number of `Part`s.

`satisfied_by` may legitimately be empty, and an empty list is not a modelling error — it is the normal
state of a requirement that has been captured but not yet designed for. It is question rule 3 in
`04-uncertainty.md`: a requirement nothing satisfies is a question waiting to be asked, and the model is
expected to hold it in that state.

```yaml
- id: r-speech-terrace
  def: audio.req.speech_intelligibility
  params:
    area: "the terrace seating area"
    audience: 300
  refines: brief.program.welcome_speech
  satisfied_by: [main-pa, delay-line]
```

## 4. Connection and Flow are separate

`Connection` and `Flow` describe the same cable run from two different angles, and both angles are needed.
A `Connection` says *a link exists between these two ports*. A `Flow` says *this content travels from here
to there, over this chain of links*. Collapsing them into one entity — the obvious simplification, and the
one most cable-oriented tools make — costs the ability to model anything that shares a link.

Everything modern shares links. Thirty-two channels of audio ride one Cat6a run; a video matrix takes eight
inputs and reaches sixteen outputs over a backplane no cable represents; a Dante network carries flows whose
route is decided by a switch rather than by the person who patched it. With a single merged entity there is
no way to say that one cable carries thirty-two things, or that two different flows share a trunk and
compete for its capacity. Switches, matrices and networked audio become unmodellable, and capacity can never
be checked, because the model has nowhere to record what is actually loaded onto a link.

The worked case: a stage box at the drum riser, a switch on stage, a switch at front of house, and the
console. Three connections form the path; one flow of 32 channels rides all three.

```yaml
connections:
  - { id: c-sb1-swstage,   from: "sb1:net[0]",        to: "sw-stage:port[0]", interface: network.interface.cat6a_link, redundancy: primary, properties: { length_m: 12 } }
  - { id: c-swstage-swfoh, from: "sw-stage:port[8]",  to: "sw-foh:port[0]",   interface: network.interface.cat6a_link, redundancy: primary, properties: { length_m: 60 } }
  - { id: c-swfoh-fohm,    from: "sw-foh:port[4]",    to: "fohm:net[0]",      interface: network.interface.cat6a_link, redundancy: primary, properties: { length_m: 5 } }

flows:
  - id: f-stage-inputs
    item: audio.item.dante_flow
    from: sb1
    to: fohm
    over: [c-sb1-swstage, c-swstage-swfoh, c-swfoh-fohm]
    properties:
      channels: 32
```

Three `Connection`s, one `Flow`, and the 32 channels recorded once — on the flow, where they belong, not
smeared across three cables. The middle link now has something to be checked against: its interface allows
1000 Mbit, and the flow riding it declares what it costs. Add a second flow over the same trunk and the
question of whether the trunk carries both becomes answerable, which is exactly what a merged entity cannot
express.

## 5. Internal traversability

`PartDef.internal` records which of a part's ports reach which others, and through what transformation. A
stage box's `analog_in` ports reach its `net` ports through an analogue-to-digital conversion; its `net`
port reaches its `analog_out` ports through the reverse. Without those entries the model is a set of
devices with cables between them and no statement about what happens inside any of them — so a walk from a
microphone to a loudspeaker stops at the first device it enters. The graph is not walkable end to end, and
a plan that cannot be walked cannot be checked: nothing can confirm that the vocal microphone actually
reaches the main PA, or that the requirement claiming to be satisfied by that path is in fact satisfied.

The precedent is GDTF, which solves the same problem for power and DMX with `WiringObject` and `PinPatch`:
a fixture declares not only its sockets but which internal pin connects to which, so that a rig's power and
data topology can be traversed through the fixtures rather than only between them. EventML's `internal` is
that idea at port granularity rather than pin granularity, applied to every domain rather than to lighting
alone. `from` and `to` name port IDs local to the definition, `[*]` meaning all indices of that port, and
`transform` names what the part does to the item on the way through.
