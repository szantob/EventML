# A project-organisation kernel — Brainstorm record

**Status: brainstorm only.** This is not a design and not a plan. No decision below has been weighed against
a release, and several of them would change constructs that are already tagged and shipped.

**Scope: a repository of its own, carried to its own 1.0 before `eventml-core` resumes.** Revised twice
during the session. The constraint stated at the outset was "after `eventml-core` 1.0"; it was withdrawn, on
the reading that EventML's own roadmap is about to spend five releases building kernel material inside
EventML, and then replaced by K14 — serialised work rather than an MVP running alongside. **What that 1.0
contains is not settled** — see OQ6. Nothing here changes `spec/` as it stands.

**Date:** 2026-08-26
**Branch at time of writing:** `eventml-core-v0.5`
**Related:** [`2026-08-26-eventml-v0.5-requirement-types-WIP.md`](2026-08-26-eventml-v0.5-requirement-types-WIP.md),
[`2026-08-24-eventml-v0.4-needs-design.md`](2026-08-24-eventml-v0.4-needs-design.md)

---

## 1. The idea

Lift L0 and L1 out of EventML as a language of their own: a kernel that builds a requirement system out of
what people actually said, and that a design language then attaches to. EventML becomes one such design
language. SysML v2 and UML become others, on the same terms.

The argument for the cut is already in the specification, written for a different purpose.
`spec/06-sysml-mapping.md` §4 lists four things in EventML that do not survive translation to SysML v2 —
the value states, the question derivation, the L2 product-name rule, and the recording criterion. **Three of
the four live at L0/L1.** The language being lifted out is therefore not a competitor to SysML: it is the
front end SysML declines to have, and it ends exactly where SysML begins.

## 2. The procedure the kernel serves

Given in the session as the reason the kernel exists. The goal is a *final* requirement system, and the
procedure is how one is reached.

1. A written source arrives — email, note, meeting transcript, a designer's own decision note — and is
   registered.
2. The source is analysed, and needs are extracted from the parts that carry information.
3. Each need is processed: its subject selects the requirement type, and the rules on that type's
   `RequirementDef` turn the stater's free words into the requirement's bound professional wording.
4. Creating requirements triggers a review of the model, which surfaces contradictions with earlier
   requirements, further questions needed for refinement, and other findings.
5. A model above the requirement model records the contradictions, gaps and pending decisions. It is
   analysed, a report is written, and somebody acts on it — a meeting is called, the client is telephoned,
   or the team decides for itself. **The outcome always enters the system as a new source**, whose needs
   produce requirements that resolve what was open.
6. The new requirements let the old contradicting ones be deprecated — never deleted, which would break
   traceability — and the imprecise ones refined.

The cycle ends when the model above empties: every question answered by somebody. The requirement model
then projects to a final, clean one by dropping the model above and the deprecated requirements.

**Three of these steps are already checkable in the language as it stands.** Step 2 by the source-coverage
report in `spec/04-uncertainty.md` §6, step 3 by question rule 8 from one side and rule 9 from the other,
step 4's questions by rules 1 and 2. Steps 3b, 4a and 5 have nothing.

## 3. Decisions

| # | Decision | Rationale |
|---|---|---|
| K1 | The kernel is L0/L1 — the evidence and intent chain. `Source`, `Need`, `Requirement`, `Decision`, the value-state model, the traceability relations and the checks over them. `Part`, `Port`, `Connection`, `Flow`, the definition entities below `Requirement`, `allocate`, boundary rules 2 and 3 and the AV libraries stay in EventML | `06-sysml-mapping.md` §4: three of the four things that do not map to SysML live at L0/L1. The cut follows a seam the specification already found |
| K2 | The kernel attaches symmetrically. It must sit under SysML v2, under UML, under EventML and under a design language not yet written, on the same terms in every case | Stated as the requirement the whole idea exists to meet. No design language gets a privileged path |
| K3 | The kernel sees below L1 through exactly one seam: an element outside the kernel carrying `satisfies`, naming a requirement | `06-sysml-mapping.md` §2 records `satisfy` as mapping directly to SysML v2 *including which end carries it*. The adapter the seam needs is already written. It keeps rules 3 and 6 computable over any design language |
| K4 | A design language attaches through a **binding** that declares four things: which of its elements may carry `satisfies`; its own internal refinement chain; its identifier space; and how far it takes the kernel's value model | The identifier problem is recorded in `06-sysml-mapping.md` §3 — `audio.part.stage_box` and `Audio::DigitalStageBox` are different identifier spaces, and a round trip without a map loses the stable IDs everything depends on |
| K5 | A `Requirement` carries a property marking it no longer in force. A requirement is never deleted | Traceability. Also the v0.5 WIP §5 argument for retirement: the checks are pairwise, so deleting an element silences exactly the check that fired on it |
| K6 | A `Need` carries no lifecycle state | A need belongs to its source, and a source is material of record — quoted whole, never edited. A quotation cannot cease to be true. What a need needs instead is a disposition; see OQ3 |
| K7 | Contradiction between requirements is found by human or AI review, not by rule. The kernel defines what a contradiction is, what a reviewer must cite, and how a found one is recorded — it does not detect them | Consistent with locked decision 1. A rule-based detector would find only the contradictions somebody anticipated by writing a constraint, which is the case that least needs finding |
| K8 | A requirement's type arrives with the `RequirementDef` chosen for it. Needs are not classified | D53 already has each template naming its kind. The need's subject selects the template; the kind rides along. This is why the v0.5 §5 origin clusters and a subject-driven procedure never collide — two axes, one contact point |
| K9 | Question rule 9 is a failed check, not a question | The rule's own text calls it an invariant: *"One invariant defines it: every requirement names its origin."* The rule tree already routes the parallel case — an L3 part with no `allocate` — to *"a failed check — boundary rule 3, not a question"*, and boundary rule 3's justification applies verbatim. See §6 |
| K10 | The model above the requirement model is a model, not a derived view | A contradiction links at least two requirements and must keep its identity between reviews. A recomputed report has no identity across runs; a modelled element does |
| K11 | **The requirement model changes only through sources.** Every outcome of a review — a meeting, a telephone call, the team's own decision — enters as a source | What makes traceability total rather than merely well-intentioned. It is also what makes K10 tractable: no finding can go stale unnoticed, because nothing moves beneath it without a source accounting for the move |
| K12 | The cycle ends in a **baseline**: a dated, identified cut of the requirements in force. A design language binds to a *named* baseline, not to "the requirements" | ISO/IEC/IEEE 29148's term, adopted under house rule 8 rather than coined. Without it the implementing team designs against a set that moves under them, which is the experience that makes engineers distrust requirement models. The cost is an identifier and a date |
| K13 | The baseline is a simplification for a different audience, not a model that must pass the kernel's checks. Its condition is losslessness and recoverability: everything in force is present, nothing in force was dropped, and everything dropped stays in the working model | The baseline drops the need layer, so need-coverage rules cannot apply to it — running them on it is a category error. People building the thing do not need to know what the candidate requirements were |
| K14 | **The work is serialised, not parallel.** The kernel gets its own repository, built from what exists today and carried to its own 1.0 with `eventml-core` frozen for the duration. EventML resumes afterwards, as the kernel's first consumer | The five releases on EventML's own roadmap are kernel work, so serialising does that work once instead of twice. It also dissolves the moving-target risk by decree rather than by mitigation: nothing can fork underneath a specification that is not moving. And a specification resumed against a finished kernel is a far better test of K2 than two that co-evolve and quietly accommodate each other |

## 4. Open questions

**OQ1 — One specification or two?** The kernel has two separable pieces: the evidence chain, which attaches
at exactly one seam, and the value-state model, which attaches to every value or to none. They cannot be
adopted equally — EventML's YAML lets a scalar be replaced by an object and SysML v2's textual notation does
not, so a SysML binding can take the first and not the second. Three options were put: one specification
with conformance levels (level 1 evidence chain, level 2 plus value states, level 3 plus question
derivation), two specifications, or one monolith. **Conformance levels were recommended and the question was
not answered.** The recommendation rests on the level itself becoming information: read a binding, and you
know what an exported model loses. That knowledge sits in prose today, in `06-sysml-mapping.md` §4.

**OQ2 — Does `RequirementDef` split?** It carries two kinds of attribute. `applies_when`, `params`,
`default` and `asks` are procedural, and their meaning bottoms out in the kernel's own value model.
`constraints` and `verification` bottom out in the design language's semantics — and `06-sysml-mapping.md`
§1 already maps `verification` to a SysML `verification def`, "which EventML v0.1 keeps as prose". **A split
along that test was recommended and the question was not answered.**

**OQ3 — The orphan need.** *Recorded as a problem to solve, at the user's request.*

A need that no requirement refines fires question rule 8, and the rule has two readings. The specification
already knows both. `examples/03-festival-stage/brief.yaml` says so in `n-park`'s own notes — "the park is
where the event is, not something the system must achieve — one of the two readings a person has to choose
between when the rule fires" — and `examples/01-garden-party/brief.yaml` says of `n-lunch` that it "may
constrain the load-in or mean nothing at all, and only the venue can say which."

**What is missing is any way to record which reading won.** Seven needs across the three examples are
refined by nothing:

| Example | Needs refined by nothing |
|---|---|
| `01-garden-party` | `n-load-in`, `n-lunch` |
| `02-conference-room` | `n-same-as-last-year`, `n-room`, `n-house-lights`, `n-venue-tech` |
| `03-festival-stage` | `n-park` |

Every review re-adjudicates all seven, and the person doing it cannot see that somebody already did. The
conference room stages the sharpest case deliberately: `n-house-lights` is refined by nothing and fires rule
8, while `r-house-lights` names no origin and fires rule 9 — the same subject, both ends open, and no edge
between them.

This is not retirement, and K6 is why: the need was stated and remains stated. It is a **disposition** — a
record that the need was examined and deliberately produced nothing, with the reason. It has the same shape
as the second half of the v0.5 §3 open question: every declared kind either has a requirement in the project
or a written reason why it has none.

**The hazard to design against.** A disposition that costs nothing to write becomes a way to silence rule 8,
which is precisely the failure the v0.5 WIP identifies for deletion — *"deletion becomes a way to make a
report clean, with nothing recording that a choice was made."* Whatever form a disposition takes has to make
the silencing visible: a reason at minimum, and probably a source or a decision standing behind it.

**OQ4 — Does the kernel name the loop's steps normatively?** Asked twice and answered around both times. The
options were: entities only; entities plus the five steps as a normative process, adopting ISO/IEC/IEEE
29148's process vocabulary; or entities plus lifecycle states with no process prescribed. **The third was
recommended**, on the ground that the process is the part most likely to collide with an adopting
organisation's own way of working, and therefore the least portable thing the kernel could make normative.
K5, K6 and K10 all point that way without settling it.

**OQ5 — What is the kernel called?** **ProjectML** was put forward, with a question mark. It reads well and
it is the obvious parallel to EventML. One reservation is worth recording before a repository fixes it:
*project* claims the whole of project management — schedule, budget, resources, people — while the language
does requirements engineering and nothing else. A name that overclaims invites the wrong first question from
everybody who reads it, and a repository name is cheap to change this week and expensive in a year.

**OQ6 — What is the MVP, and where does it live?** The session opened under an "after 1.0" constraint and
withdrew it: the extraction should be an MVP project of its own, probably in a separate repository.

**The argument for not waiting.** Read the v0.5 WIP §4 release order — a library declares its requirement
kinds, then project-management logic, then what produces a decision, then how the model survives change,
then the question lifecycle. **Not one of those five is AV-specific.** `eventml-core` is about to spend five
releases building kernel material inside EventML. Waiting for 1.0 does not avoid the work; it does the work
in the wrong repository and moves it afterwards.

**The argument for caution.** The kernel's own content is the part of EventML moving fastest. `Source` and
`Need` became entities in v0.4, and the five releases above all land on the same material. An extraction now
forks a moving target, and the MVP has to be designed against that rather than around it.

**What "viable" means for a specification.** There is no software to ship, so the MVP is minimum *provable*
rather than minimum runnable, and CLAUDE.md §5 already fixes the standard: a concept that appears in no
example is unproven. The decisive test is therefore not that the kernel has the entities — moving files
proves nothing — but that **a second, non-AV domain walks end to end without the kernel changing**, and that
**two bindings can be written**, one to EventML and one to SysML v2. K2 is either true or false, and one
non-AV binding decides it.

**What failure would look like**, stated in advance so the MVP can fail honestly:

- the SysML binding cannot be written without changing the kernel's entities — K2 is false
- the non-AV example needs a concept L0/L1 does not have — the cut is in the wrong place
- the domain leaks in §5 turn out to number thirty rather than three

Three MVP shapes were put — a thin snapshot, a thick one absorbing v0.5, and one with no repository at all —
and a fourth was chosen instead: **build the repository and carry it to its own 1.0 before resuming
EventML.** That is K14, and it is not an MVP. What remains open is what its 1.0 contains.

**Three things the decision leaves unsettled.**

*What "what exists today" means.* v0.5's requirement-kinds work is brainstormed and **not implemented** —
the branch carries two records and no specification change. So the starting point is `v0.4.0` plus two
brainstorm records, which is a clean line, but only until somebody implements something. It should be named
as a tag rather than as a date.

*What happens to v0.5.* Its subject — a library declares the kinds of requirement it handles — is kernel
material entire. **The recommendation is that it moves to the kernel rather than being deferred**, and that
`eventml-core` v0.5 as currently conceived does not happen at all. Deferring it would leave a designed
release pointing at material that no longer lives in the repository.

*What 1.0 means.* Undefined, and the plan rests on it. A 1.0 defined as a feature list is a horizon, not a
release. **The recommendation is to define it by the procedure in §2: 1.0 is reached when the loop runs end
to end and terminates in a baseline** — which makes it walkable, in the sense CLAUDE.md §5 already requires,
rather than countable.

**One constraint to fix now, because schedule pressure will take it first.** A 1.0 whose only worked
examples are events has not proved neutrality; it is EventML's L0/L1 under a new name. **A non-AV worked
example, and a SysML binding, are release criteria and not extras.** The gate recommended earlier survives
K14 in a weaker but still useful form: write the SysML binding *early* in the kernel's life rather than as a
precondition for the repository, so that a false K2 is discovered in week two instead of at 1.0.

The question that follows 1.0, and is not settled: how EventML then relates to the kernel — a citation with
the kernel authoritative, or a copy. K14 makes the first much more likely, since a frozen EventML resuming
against a finished kernel has no reason to keep a copy.

## 5. Findings worth keeping

**The value-state model is not confined to L0/L1.** `spec/04-uncertainty.md` §1 says wrapping applies to
"any value anywhere in a model — a brief field, a `Requirement.params` entry, a `Part.properties` entry, a
`Connection` length". This is what forces the two-piece structure behind OQ1: the evidence chain attaches at
one seam, and the value model attaches everywhere or nowhere.

**Three domain leaks sit in an otherwise neutral L0/L1**, and would have to become library-declared
vocabularies rather than fixed enumerations:

- `Source.kind`: `email | call | document | site_visit | rider | regulation` — **rider** is event-specific
- `Source.from` and `Decision.by.party`: `client | venue | production | authority | performer` — **venue**
  and **performer** are event-specific
- the catalogue identifier's domain enumeration, fixed by locked decision 3

**D55 has a stronger reason than the one recorded.** The v0.5 WIP justifies leaving the taxonomy to the
library by ownership (the v0.3 rule) and by house rule 8 (coin nothing). Both hold. The stronger reason is
structural: **a taxonomy fixed by the language would fix a single classification axis**, and the WIP's own
§5 finding is that SysML classifies by the nature of a requirement while this domain's logic splits by
origin. A fixed taxonomy would exclude every design language classifying on another axis — which is to say,
it would make the kernel unattachable. D55 is not a courtesy to library owners; it is what makes K2
possible.

**Findings come in three kinds, and they behave differently.** A failed check and a question are both
decidable by script and are recomputed on every run, so storing them only risks staleness. A review finding
needs judgement to produce and is lost unless it is stored. K10 settles the storage question; the
distinction still matters, because only the third kind can carry a state.

**`Source.answers` is the natural closing edge for a finding.** It already runs from a source to the earlier
sources it responds to. Extending it to close a finding makes closure evidence rather than a tick somebody
applied — the same move `agreed_by` already makes for consent.

**`Source.from` correlates with the v0.5 origin clusters but is not a function.** Client to Programme, venue
and site visit to Site, authority and regulation to Regulatory, our own judgement to Consequential — four
origins, four clusters, and the correspondence is not a coincidence, because "who could answer" is what the
clusters split on. It is not a dispatch, though: a client can state a site fact. Under K8 nothing turns on
it, and it survives at most as a smell test — a kind that disagrees with its source's origin is worth a
second look. Whether that is signal or noise could be measured against the 22 templates.

## 6. What this touches in released work

**Question rule 9 shipped in `v0.4.0`.** K9 is therefore a change to released specification, and it is
available independently of everything else here — it needs no kernel and no 1.0. Two further pieces of
evidence beyond the rule's own wording: its worked example carries `blocks: 0`, so it never competes for
attention the way a question does; and `spec/04-uncertainty.md` §4 already calls it "the exception among the
rules added in v0.4", which is a specification being uncomfortable with its own categorisation.

**One consequence has to be accepted with it.** `spec/04-uncertainty.md` says "Rule 9 is rule 6 one layer
up, and the symmetry is exact." If rule 9 moves and rule 6 does not, the symmetry stops being exact and that
sentence needs correcting. Rule 6 should not move with it: no invariant says every L2 part satisfies
something, and an infrastructure block that satisfies nothing is a fair question rather than an error.

## 7. Resume here

**OQ6 first**, because it decides whether the others are answered here or in another repository. Then OQ1
and OQ2, the two that were put and left unanswered; OQ3, the problem this record was asked to capture; OQ4,
recommended but never confirmed; OQ5 last, waiting for the other five on purpose.

Nothing in §3 has been checked against a release plan. §6 bears on `eventml-core` whatever OQ6 decides,
since rule 9 is already released and its recategorisation needs no kernel.
