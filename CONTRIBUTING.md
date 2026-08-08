# Contributing

## 1. Scope

v0.1 accepts corrections and domain-concept proposals. It does not accept code. This repository ships prose
and data only — no validator, no scripts, no schema — and that is intentional for this release. See
[`CLAUDE.md`](CLAUDE.md) for the full set of locked decisions behind that choice.

## 2. Proposing a domain concept

Open an issue stating:

- **The term** — the name you want to add.
- **The domain** — one of `audio`, `lighting`, `video`, `network`, `power`.
- **Which entity kind it is** — `item`, `port`, `part`, `interface`, `req` or `constraint`.
- **Whether an existing standard already names it** — and where, if so.
- **One concrete situation where a model needs it** — a real case the catalogue currently cannot express.

## 3. The adopt-don't-invent rule

Where a term already exists in an established standard, EventML adopts it and records the source rather than
inventing a synonym. This is not a preference — it is the rule new proposals are checked against. The first
places to look:

- **GDTF** `SignalType` for audio, video and data signal vocabulary.
- **NMOS** Sender/Receiver for networked media flow abstractions.
- **IFC** port direction semantics for how a port's directionality is expressed.

If the term is found in one of these (or in SysML v2), use that term and record where it came from in a
`source:` key. Only coin a new term when no standard has one.

## 4. Naming

Catalogue entries use the ID format locked in [`CLAUDE.md`](CLAUDE.md):

> **Catalogue ID format:** `<domain>.<kind>.<name>` — lowercase, `snake_case` name, e.g.
> `audio.item.analog_line`, `audio.port.xlr3f`, `power.part.distro`, `audio.req.speech_intelligibility`.
> Domains: `audio`, `lighting`, `video`, `network`, `power`. Kinds: `item`, `port`, `part`, `interface`, `req`,
> `constraint`.
