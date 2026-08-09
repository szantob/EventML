# 05 — Concrete syntax

EventML's canonical concrete syntax is YAML 1.2. This file is the contract every file in a library and
`examples/` follows: where files live, how they open, how identifiers are formed, and how one element
refers to another.

YAML was chosen for one decisive reason, set out in the design record: an agent is a co-author of these
models, and agents write YAML reliably while writing strict formal grammars unreliably. A second concrete
syntax — a mapping to SysML v2 textual notation, for export and interoperability — is defined in
`06-sysml-mapping.md`. No exporter ships in v0.1.

## 1. File layout

**A project model is a directory of four files, one per layer, plus a manifest.**

```
<project>/
├── project.yaml        # names the library this project resolves against
├── brief.yaml
├── requirements.yaml
├── logical.yaml
├── physical.yaml
└── decisions.yaml      # optional
```

One file per layer rather than one file per project, because the layers are written at different times by
different people. The brief is captured during the first phone call; the physical design is finished the
week before the event and revised on site. Keeping them apart means a change to the truck pack does not
touch the file recording what the client asked for.

`decisions.yaml` is the one project file that is not a layer. It is present only when the project records
decisions, and it is absent from a model that records none — see `07-decisions.md`. `project.yaml` is not a
layer either — see §5's resolution rule for what it declares and why it stands apart.

**A library is a manifest plus a directory per domain, each a directory of four files, one per entity
kind.**

```
<library>/
├── library.yaml         # the library's name and version
├── audio/
│   ├── items.yaml
│   ├── ports.yaml
│   ├── parts.yaml
│   └── requirements.yaml
└── <other domains>/
```

Split by entity kind rather than gathered into one file per domain, because items, ports, parts and
requirements change at different rates and are reviewed by different eyes. A new connector type is a
five-line change to `ports.yaml`; it should not require opening a thousand-line domain file.

Definition entities without a file of their own — `InterfaceDef` and `ConstraintDef` — live alongside the
entity they serve: interfaces in `ports.yaml`, constraints in the file whose entities reference them.

## 2. File headers

Every YAML file is a **two-document stream**: a header document of three keys, then `---`, then the
content. The split keeps the header machine-readable without parsing the body, and makes it impossible to
write content that accidentally shadows a header key.

Library files:

```yaml
eventml: "0.1"
kind: items          # items | ports | parts | requirements
domain: audio
---
items:
  - id: audio.item.analog_line
    name: Analogue line level
    kind: signal
    form: analog
    carries: One channel of balanced line-level audio
```

Model files:

```yaml
eventml: "0.1"
layer: brief         # brief | requirement | logical | physical
project: garden-party
---
brief:
  audience: 300
  venue:
    name: Gellért terrace
```

A library manifest:

```yaml
eventml: "0.3"
kind: library
library: eventml-example
---
library:
  version: "0.2.0"
  domains: [audio, lighting, video, network, power]
```

A project manifest:

```yaml
eventml: "0.3"
kind: project
project: garden-party
---
project:
  library: eventml-example@0.2.0
```

| Key | Files | Values |
|---|---|---|
| `eventml` | all | The language version the file is written against, quoted so it stays a string. Per file, not per project: `"0.1"` for a file using only v0.1 constructs, `"0.2"` for one using the `Decision` entity. A project whose four layer files stay at `"0.1"` alongside a `decisions.yaml` at `"0.2"` is correct |
| `kind` | library domain files, decisions, library manifest, project manifest | `items` \| `ports` \| `parts` \| `requirements` \| `decisions` \| `library` \| `project` |
| `domain` | library domain files | `audio` \| `lighting` \| `video` \| `network` \| `power` |
| `layer` | model | `brief` \| `requirement` \| `logical` \| `physical`, exactly as in `01-layers.md` |
| `project` | model, decisions, project manifest | Short kebab-case project identifier, the same in all files of the project |
| `library` | library manifest | The library's own name, e.g. `eventml-example` |

A library domain file carries `kind` and `domain` and never `layer` or `project`. A model file carries
`layer` and `project` and never `kind` or `domain`. `decisions.yaml` carries `kind: decisions` and `project`
and never `layer` or `domain` — see `07-decisions.md`. A library manifest carries `kind: library` and
`library` naming itself; a project manifest carries `kind: project` and `project` naming itself. The
content document's single top-level key matches the header: a file with `kind: items` opens its body with
`items:`, a file with `layer: brief` opens with `brief:`, a file with `kind: decisions` opens with
`decisions:`, a library manifest opens with `library:`, and a project manifest opens with `project:`.

The third header key has always named the thing a file belongs to: `domain` for a library's domain files,
`project` for a model file. A manifest is no different — its third key names the thing it declares, a
library naming itself with `library` or a project naming itself with `project`.

## 3. Identifiers

**Catalogue IDs** are dotted: `<domain>.<kind>.<name>`, all lowercase, the name in `snake_case`.

| Position | Values |
|---|---|
| `<domain>` | `audio` \| `lighting` \| `video` \| `network` \| `power` — closed list |
| `<kind>` | `item` \| `port` \| `part` \| `interface` \| `req` \| `constraint` — closed list |
| `<name>` | `snake_case`, unique within its domain and kind |

```
audio.item.analog_line
audio.port.xlr3f
power.part.distro
audio.req.speech_intelligibility
network.constraint.max_copper_run
```

**Instance IDs** are short, lowercase, unique within a project, and **contain no dots**: `sb1`, `fohm`,
`main-pa`, `r-speech-terrace`. Kebab-case where a word break helps; a bare word where it does not.

The no-dots rule is the point. A reader — human or agent — can tell a catalogue reference from an instance
reference by sight, without knowing what either names. `def: audio.part.stage_box` is a type;
`allocate: main-pa` is an instance. No lookup is needed to distinguish them, and a mistyped reference of
one kind cannot silently resolve as the other.

## 4. References

Six reference forms, and no others.

| Form | Syntax | Example |
|---|---|---|
| Catalogue type | dotted ID under `def:` | `def: audio.part.stage_box` |
| Instance | bare short id | `sb1` |
| Port | `<part_id>:<port_id>[<index>]` | `sb1:analog_in[3]` |
| Brief path | dotted path from the `brief` root | `brief.program[1].size` |
| Source | bare id under `src:` | `src: s-client-brief` |
| Library | `<name>@<version>` under `project.library` | `eventml-example@0.2.0` |

```yaml
parts:
  - id: sb1
    def: audio.part.stage_box        # catalogue type
    layer: physical
    allocate: stage-input-block      # instance

connections:
  - id: c-mic-sb1
    from: "vox1:out"                 # port, count is 1 so no index
    to: "sb1:analog_in[3]"           # port, index 3
    interface: audio.interface.xlr_line

requirements:
  - id: r-band-monitoring
    def: audio.req.performer_monitoring
    refines: brief.program[1].size   # brief path
```

**Port indices** are zero-based, and the index is omitted when the port's `count` is 1 — `vox1:out`, not
`vox1:out[0]`. Port references are quoted, because an unquoted value containing a colon is not valid YAML.

**`[*]`** means every index of a port. It is valid only inside `PartDef.internal`, where it expresses that
all sixteen analogue inputs reach the network port; it is never valid in a `Connection`, which always joins
two specific ports.

**Brief paths** index into the `brief` document with dots and, for sequences, bracketed integers.
`brief.program[1].size` is the `size` field of the second entry in the brief's `program` list. This is the
only form that reaches inside a document rather than naming a top-level element, and it exists because L0
has no entities — see `01-layers.md`.

**Source references** name an entry in the brief's `sources` list, defined in `01-layers.md`. A source
reference is an instance reference like any other — no dots, resolved within the project — and it carries
nothing but the handle. Everything worth knowing about a source is a key on the source entry, never a
decoration on the reference to it: `src: "s-venue-call, production manager"` is not a reference but a
sentence, and nothing can resolve it.

## 5. Library resolution

**A model resolves against exactly one library**, named in its `project.yaml`. There is no search path, no
fallback, and no merging of two libraries. A `def:` the named library does not define is a dangling
reference, not an invitation to look elsewhere. A model whose vocabulary could come from two places has no
answer to "what does this id mean", and the answer has to survive the person who wrote it — a reviewer
reading the model a year later, without whatever second library happened to be on the original author's
machine, has to reach the same resolution they did.

**The reference names a library, not a place.** `eventml-example@0.2.0` says which vocabulary at which
version; where it is stored and how it is fetched belongs to whatever runs the model — a git repository, a
folder on a machine, a web interface. This is the same separation `src:` already makes: a `src` names a
`sources` entry, not a file offset, because a reference that means something on one machine and nothing on
another is not a reference at all.

**The shipped example library resolves no real project.** It exists to be read and to be copied from as a
one-time seed — the manifests in `examples/` point at `eventml-example@0.2.0` only because the worked
examples must resolve against something. An organisation that copies entries from it owns the copies, and
its library versions independently from that point: nothing ties its version numbers to `eventml-example`'s
after the copy is made.

## 6. Conventions

- **Indent two spaces. Never tabs.**
- **Inline flow mappings** carry one record on one line: a port declaration, an `internal` edge, a `params`
  entry, a `Connection`, a `Flow`, or a `Part` whose values are all plain scalars. The test is structural,
  not a character count. A record stays inline while all three hold: it nests at most one level deep
  (`properties: { length_m: 12 }` is inline, a map inside that map is not); none of its values is a wrapped
  `Value`, so anything carrying `state:` expands; and it needs no comment of its own. The moment one fails,
  the record expands to block form.
- **Align the columns within a run of inline records.** The alignment is what makes a sixty-entry
  connection list scannable, and it is the whole reason the inline form is worth having. A run whose
  columns do not line up should be block form instead.
- **Key order:** `id` first, then `name` (definitions) or `label` (usages), then `def`, then everything
  else. A reader scanning a file's left edge should see what each element is before reading what it does.
- **Encoding is UTF-8, line endings are LF.** Venue and client names are Hungarian; `Gellért` is written as
  written, not transliterated.
- **Comments** explain why, not what. `# 32 A three-phase, confirmed on the site visit` earns its line;
  `# the stage box` does not.
- **`notes` is permitted on any entry**, in library files and in model files alike, and holds free text about the
  entry rather than part of it. It is where a modeller says what a reader would otherwise have to
  reconstruct: why a value was left unknown, what a decision would cost to reverse, which of two readings
  of a brief sentence was taken. Nothing in the language derives anything from it. The entity tables in
  `02-metamodel.md` and `07-decisions.md` do not repeat it — it is a property of entries, not of any one
  entity, and a comment is the wrong tool because comments do not survive a round trip through a parser.

```yaml
connections:
  - { id: c-sb1-swstage,   from: "sb1:net[0]",      to: "sw-stage:port[0]", interface: network.interface.cat6a_link, redundancy: primary, properties: { length_m: 12 } }
  - { id: c-swstage-swfoh, from: "sw-stage:port[8]", to: "sw-foh:port[0]",   interface: network.interface.cat6a_link, redundancy: primary, properties: { length_m: 60 } }

parts:
  - id: sb1
    label: "Stage box — drum riser"
    def: audio.part.stage_box
    layer: physical
    allocate: stage-input-block
    satisfies: [r-input-capacity]
    properties:
      electrical_load_w: 90        # measured, not the datasheet figure
```

The connections stay inline at some length because they are a table and read as one; `sb1` expands because
its `properties` carries a comment and would nest a second level. **An earlier draft of this file capped
inline records at 100 characters. The cap was removed because it was wrong in practice:** the records that
matter most — a connection naming two ports, an interface and a length — run to about 170 characters once
aligned, and splitting each into six lines turns a readable sixty-line table into four hundred lines nobody
scans. Length is not what makes an inline record hard to read. Nesting and hidden state are, and those are
what the rule now tests.
