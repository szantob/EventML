# lib — the EventML domain library

The library is **data, not schema.** Everything here is written in the language defined by `spec/`; nothing
here extends it. A new microphone type, a new connector, a new requirement template is a change to this
directory. A new *kind of thing* — a new entity, a new relation, a new value state — is a change to `spec/`
and needs justification.

Where the two disagree, `spec/` wins.

## The four-file pattern

Every domain is a directory of four files, one per entity kind:

```
lib/<domain>/
├── items.yaml          # ItemDef      — what flows
├── ports.yaml          # PortDef      — connection points (and InterfaceDefs)
├── parts.yaml          # PartDef      — device types and abstract L2 blocks
└── requirements.yaml   # RequirementDef — parameterised requirement templates
```

The five domains are `audio`, `lighting`, `video`, `network` and `power`. The list is closed in v0.1: a
sixth domain is a `spec/` change, because the domain name is the first segment of every catalogue ID.

`InterfaceDef`s live in `ports.yaml` alongside the ports they join. `ConstraintDef`s live in the file whose
entities reference them — a cable-length rule sits with the interface it constrains.

Splitting by entity kind rather than gathering each domain into one file keeps every file small enough to
read in one sitting, and matches how the content changes: connectors are added often, requirement templates
rarely, and the two are reviewed by different people.

## Catalogue IDs

`<domain>.<kind>.<name>` — lowercase, `snake_case` name. Kinds are `item`, `port`, `part`, `interface`,
`req`, `constraint`.

```
audio.item.analog_line
network.port.ethercon_a
power.part.distro
audio.req.speech_intelligibility
```

IDs are stable. Renaming one breaks every model that references it, so a concept that turns out to be
misnamed keeps its ID and gains a better `name`.

## Versioning

`eventml-lib` versions separately from `eventml-core`. The kernel should change rarely; the vocabulary can
grow at any time, and a new microphone must never require a new language version. Every file declares the
*language* version it is written against in its `eventml:` header key — at the v0.1 tag both are `0.1.0`
and every file declares `eventml: "0.1"`.

## Adopt, don't invent

Where a term already exists in GDTF, AES, AMWA NMOS, IFC4 or SysML v2, EventML uses that term and records
where it came from in a `source:` key on the entry that defines it:

```yaml
- id: audio.item.analog_line
  name: Analogue line level
  kind: signal
  form: analog
  carries: One channel of balanced line-level audio
  source: "GDTF SignalType: AnalogAudio"
```

A term is coined only when no standard has one, and then it is coined once and reused. The `source:` key is
what stops the vocabulary drifting into a private dialect nobody else can read — and what makes an eventual
export to SysML v2 or GDTF a mapping rather than a translation.

## Cross-domain references

Domains reference each other freely, and must: an audio stage box has a power inlet and a network port.
References cross by catalogue ID with no import declaration — `audio.part.stage_box` naming
`power.port.powercon_true1` is ordinary, not exceptional. The domain segment of an ID says which directory
defines it.
