# 06 — SysML v2 mapping

EventML takes its metamodel from [SysML v2](https://github.com/systems-modeling/sysml-v2-release) and
adds domain vocabulary and an uncertainty model. This file records the mapping in both directions: what
each EventML construct corresponds to in SysML v2 textual notation, and what does not survive the
translation.

**No exporter ships in v0.1.** This is a documented correspondence, not a tool. It exists so that the
standards-alignment claim can be checked rather than asserted, and so that an exporter written later has a
specification to work from.

This file was written after the two worked examples, deliberately. Mapping a language that has been
exercised is honest; mapping one that has only been imagined is not.

## 1. Entities

| EventML | SysML v2 | Note |
|---|---|---|
| `ItemDef` | `item def` | Direct. `kind` and `form` become attributes; they have no SysML counterpart because SysML is domain-neutral |
| `PortDef` | `port def` | EventML fixes `direction` on the port definition; SysML puts direction on the feature inside the port body, via `in`/`out` on a flow feature |
| `PartDef` | `part def` | Direct for ports and attributes. `internal` has no direct equivalent — the nearest is `connect` declarations inside the part's own body, which express reachability but not `transform` |
| `InterfaceDef` | `interface def` | Direct. `capacity` becomes attributes on the interface definition |
| `RequirementDef` | `requirement def` | Direct. `params` map to the requirement's `subject` and parameter features; `verification` maps to a `verification def` reference, which EventML v0.1 keeps as prose |
| `ConstraintDef` | `constraint def` | Structurally direct, semantically not: SysML constraint bodies are evaluable expressions, EventML v0.1's `expression` is prose |
| `Part` | `part` usage | Direct |
| `Connection` | `connect` | Direct. `redundancy` has no counterpart and becomes metadata |
| `Flow` | `flow` | Direct. SysML's `flow` carries a payload between features, which is exactly `Flow.item`; `over` maps to the `flow` declaration's route through connections |
| `Requirement` | `requirement` usage | Direct |

The `def`/`usage` split is taken from SysML v2 unchanged, and it is the single most important thing EventML
borrows. Everything else follows from it: a catalogue that is shared across projects, instances that are
per-event, and a clean statement of which is which on every element.

## 2. Relations

| EventML | SysML v2 | Note |
|---|---|---|
| `refine` | `refine` | Direct. EventML's source is an L0 brief path rather than a model element, because L0 has no entities |
| `derive` | `derive` | Direct |
| `satisfy` | `satisfy` | Direct, including which end carries it. SysML v2 nests `satisfy requirement : SomeReq;` inside the satisfying part, binding the requirement's subject to the enclosing element; the standalone form `satisfy R1 by vehicle;` names both ends. EventML's `Part.satisfies` is the first form with the reference written out |
| `allocate` | `allocate` | Direct in meaning. SysML v2 reifies it as an `AllocationUsage` connecting two elements; EventML stores it as an attribute on the L3 part, which fixes the direction the SysML form leaves open |
| `trace` | metadata annotation | No first-class SysML relation. Nearest is a metadata definition carrying a source reference, applied to the annotated element |

SysML v2 has no equivalent of the value-state model. A `stated` value and an `assumed` value both become
plain attribute values on export, and the `why`, `src`, `ask` and `alternatives` that distinguish them are
lost unless carried as metadata. **This is why a round trip through SysML v2 loses information**, and why
EventML is a model library extension rather than a re-encoding.

## 3. Side by side

`audio.part.stage_box` from `lib/audio/parts.yaml`, in both notations.

**EventML:**

```yaml
- id: audio.part.stage_box
  name: Digital stage box
  category: stage_box
  ports:
    - { id: analog_in,  def: audio.port.xlr3f,          count: 16 }
    - { id: analog_out, def: audio.port.xlr3m,          count: 8 }
    - { id: net,        def: network.port.ethercon_a,   count: 2, role: primary_secondary }
    - { id: pwr,        def: power.port.powercon_true1_in, count: 1 }
  internal:
    - { from: "analog_in[*]", to: net,             transform: adc }
    - { from: net,            to: "analog_out[*]", transform: dac }
  properties:
    electrical_load_w: 100
    preamps: 16
```

**SysML v2 textual notation:**

```
package Audio {
    part def DigitalStageBox {
        attribute electricalLoadW : ScalarValues::Real = 100;
        attribute preamps         : ScalarValues::Integer = 16;

        port analogIn  : XLR3F[16];
        port analogOut : XLR3M[8];
        port net       : EtherconA[2];
        port pwr       : PowerconTrue1In[1];

        connect analogIn to net;
        connect net      to analogOut;
    }
}
```

**What survives.** The part definition, its attributes, its ports with their types and multiplicities, and
the fact that the analogue inputs reach the network port and the network port reaches the analogue outputs.
A SysML tool importing this can traverse the part end to end.

**What does not.** Three things:

- **`transform`.** SysML's `connect` says the ports are joined; it does not say the signal is converted
  from analogue to digital on the way through. Recovering it requires an `ItemFlow` with a declared payload
  change, or a metadata annotation.
- **`role: primary_secondary`.** The two network ports are a redundant pair, not two independent ports. In
  SysML this is a modelling convention rather than a construct.
- **The catalogue ID.** `audio.part.stage_box` becomes `Audio::DigitalStageBox`, which is a different
  identifier space. A round trip needs an ID map, or the stable IDs that every EventML model depends on are
  lost.

Note also what is *gained* in the SysML direction and unused in EventML: variability, state machines, action
semantics, calculation definitions. EventML declines all of it deliberately — see D2 in the design record.

## 4. What does not map

Three things in EventML have no SysML v2 counterpart. They are the reason EventML exists as its own
language rather than as a SysML profile.

**Value states.** SysML has no notion of a value being `assumed` rather than `stated`, or of a value being
`conflicting` because two sources disagree. An attribute either has a value or does not. EventML's entire
premise — that incompleteness is the normal state of the data and must be modelled rather than corrected —
has nowhere to live in SysML, and a translation flattens `{ value: outdoor, state: assumed, why: "…",
ask: true }` to `outdoor`.

**Question derivation.** The six question rules in `04-uncertainty.md` are a derivation over the value
states and the traceability graph. Half of their inputs do not survive translation, so the derived question
list cannot be recomputed from an exported model. The `blocks` ranking, which depends only on the
traceability graph, would survive; the questions themselves would not.

**The L2 product-name rule.** Boundary rule 2 in `01-layers.md` — that L2 carries no product names, and
that if removing the manufacturer changes the meaning of a statement then it belongs in L3 — is a
constraint on what may be written where. SysML has layers only by convention, through packages; there is no
construct that forbids a logical part from naming a product. The rule that keeps the logical architecture
reusable across equipment choices is not expressible.

None of this is a criticism of SysML v2, which is domain-neutral by design and would have to carry all
three as extensions anyway. It is the reason the design record chose a model library extension with a
documented mapping over a profile: EventML keeps the metamodel it borrows and adds the three things the
domain actually needs, without inheriting the machinery it does not.
