# examples/lib — an example EventML domain library

**This is one library, not the library.** EventML has no central catalogue: every organisation modelling
real events owns its own, and a project resolves against exactly one of them by name and version — see
`spec/05-concrete-syntax.md`. This one exists so the worked models under `examples/` have something to
resolve against, and so that an organisation starting out has entries worth copying as a seed. Copy from it;
do not resolve a real project against it.

A library is **data, not schema.** Everything here is written in the language defined by `spec/`; nothing
here extends it. A new microphone type, a new connector, a new requirement template is a change to this
directory. A new *kind of thing* — a new entity, a new relation, a new value state — is a change to `spec/`
and needs justification.

Where the two disagree, `spec/` wins.

## The four-file pattern

Every domain is a directory of four files, one per entity kind:

```
<library>/<domain>/
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

### Which domain owns a cross-domain part

Nearly every part is cross-domain. Of the parts in this library, all but a handful declare ports from at
least two domains, and anything that plugs into the wall declares a `power` port. Multi-domain is the
normal case, not the exception, so the question is not *whether* a part crosses domains but which domain
gets its ID.

**A part belongs to the domain of the function it performs — not to the domain of its ports.** Power and
network are almost always supporting: a lighting console needs mains and an Ethernet port, but it exists to
control lights, so it is `lighting.part.lighting_console`. The rule is stated in terms of function because
a port-counting rule gives the wrong answer, and the library already contains the case that proves it.

`video.part.led_panel` has **no video port at all** — its ports are two `network.port.rj45` and two power
connectors, because the panel receives its picture as data from a processor. A rule derived from ports
would file it under `network`, next to switches and media converters, where nobody designing a screen would
look for it. It is a video part because showing a picture is what it does.

Three consequences worth stating:

- **The domain is a filing decision, and only that.** It selects a directory and the first segment of an
  ID. It carries no semantics a model reasons with, and nothing is inferred from it. What a part actually
  touches is readable from its `ports` list, which is where the cross-domain truth lives.
- **When two functions are genuinely co-equal, split the definition.** A device sold as one box but used as
  two unrelated things — a playback machine on one job, a control surface on the next — is two `PartDef`s
  with distinct IDs, not one with a vague `category`. Definitions are cheap; a model that cannot say which
  role a machine is filling is not.
- **A part with no clear owning domain is usually mis-scoped.** If naming the function does not settle the
  domain, the definition is probably describing a rack or a subsystem rather than a part, and belongs at L2
  as an abstract block instead.
