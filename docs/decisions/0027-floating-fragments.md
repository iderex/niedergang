# 0027. Whether each surviving fragment floats

## Date

2026-08-09

## Status

accepted

## Question

A reentry casualty risk analysis has to state, per surviving fragment, whether
it floats. `docs/survey/standards.md` reads the ESA space debris mitigation
requirements and records "Floating or non-floating fragments" as one of the
contents the normative annex lists for such an analysis, and names it as the one
required output nothing on this board carried.

What has to be decided is on what basis the answer is given, what that basis is
worth, and what the tool says about a fragment whose basis it cannot evaluate.

What is not asked here is how a floating fragment is then handled. Nothing in
this project acts on the flag, and this record does not give it a consequence.

## Options considered

Leave the field out. The risk number does not depend on it and no other output
reads it. It costs the completeness of the analysis against the one standard
this board has read on the subject, and the omission would be invisible: a
result missing a required content item looks exactly like a result that has
every one.

Answer from the material alone, by comparing the material's density against sea
water. Cheapest, it needs one property the material library already has to carry
for other reasons, and it is wrong for the class of fragment that most often
floats. A sealed aluminium tank floats and solid aluminium does not, so an
answer keyed to the material returns non-floating for the case a reader is
asking about.

Answer from the fragment as a body, by comparing its mean density, current mass
over the volume of its shape, against sea water. Every input is already in the
fragment record 0005 fixes, so it adds no field to the scenario and no property
to the library. Taken below, with its bound stated.

Model the flotation. Buoyancy against displaced volume, water ingress through a
breach, air trapped in a closed section, and the sea state that decides whether
a marginal body stays up. This is the only option that answers the question
somebody actually has. It needs inputs no scenario carries, above all whether a
component is sealed at the moment of impact, and record 0005 gives a fragment no
internal structure and no breach state to read. It would be a flotation
calculation in name with the operator's guess underneath it.

Have the operator declare it per component. Honest about where the knowledge
sits, and it puts the judgement on the person least equipped to make it. It also
answers the wrong object: what reaches the water is a fragment produced by a
breakup, not the component that was described.

## Decision

The tool emits, per surviving fragment, a floating field with three values:
floating, non-floating, unknown.

The basis is a mean density comparison and nothing more. The fragment's current
mass is divided by the volume of its shape at its dimensions, and the result is
compared against a declared sea water density. The field is derived at impact
from what record 0005 already carries; it is not a new field on the fragment and
this record does not reopen that list.

The basis is stated in the output in those words, next to the field and in the
run's own vocabulary rather than in a footnote. A reader who sees floating has
to be able to see, without leaving the artefact, that a density comparison
produced it. The field carries the reference density it was compared against.

The reference sea water density is a declared model assumption. It is recorded
in the manifest under record 0008 and printed with the field. This record does
not fix its value, because no oceanographic source was read for this record and
a number written here would be a number nobody sourced.

The field's home is the fragment outcome table of record 0009, which already
carries one row per fragment with its terminal state and, for an impacted
fragment, its impact point. It is written for fragments in the Impacted state
and is unknown for the other three of record 0005's terminal states, because a
fragment that demised, that was still flying when the run ended, or that was
refused has no body at the surface to compare.

Unknown is a real answer and it is returned in three cases.

A fragment whose shape defines no closed volume, if the shape set of issue #46
ever admits one. The comparison has no denominator and the tool does not invent
one.

A fragment whose mean density is not separated from the reference by more than
the uncertainty in either. A body within that band is one whose flotation this
basis cannot decide, and reporting it as floating or as non-floating would be
reporting the sign of a difference smaller than the error on it. Where the
uncertainty on the reference is not stated, the band is not a decision this
record can take, and the run says the band is unstated rather than treating it
as zero.

A fragment in the Refused terminal state, which is the case record 0010 already
governs.

Unknown is never written as non-floating. The two say different things: one
says the fragment sank, the other says this tool cannot tell. Collapsing them
would put the more reassuring of the two answers on every fragment the basis
could not reach, which is the substitution record 0010 exists to refuse.

An absent reference density, when a run is asked for the field at all, is a
pre-flight refusal under record 0010 rather than an unknown. The input is
knowable from the scenario and the data sets before anything is flown, and
record 0010's rule is that such an absence stops the run rather than marking a
fragment.

What the tool does not do. It does not compute buoyancy, model water ingress,
model trapped air, read a sea state, or claim that a floating fragment is
recoverable. It does not use the field in the casualty area, in the footprint or
in the risk number, and no aggregate is derived from it.

## Reasons

The requirement is external and it is normative. The annex read for
`docs/survey/standards.md` lists this among the contents of a reentry casualty
risk analysis, alongside items every one of which this board already carries. An
analysis that answers all of them but one is incomplete in a way its own output
does not show.

Most of the footprint of an uncontrolled reentry is over water, which makes this
the field that separates the two questions a reader asks about such a reentry.
Whether anything reached the surface is answered by the terminal state. Whether
anything is still out there is answered by nothing else.

The mean density comparison is chosen because it can be checked. Every input to
it is in the artefacts, so a reader who disagrees with the answer can recompute
it from the same row and argue with the reference density rather than with a
model they cannot see. That property is worth more here than accuracy the tool
cannot honestly claim.

It also adds nothing. No scenario key, no material property, no fragment field,
no new physics behind an interface. A required output that costs one derived
column is a different proposition from one that costs a model.

Three values rather than two follows from the rest of this tree. Record 0005
makes refusal a terminal state so that a missing input is visible in the output,
and record 0010 makes a missing input a refusal rather than a default. A
two-valued flag would have to answer every fragment, and the answer it would
give where it could not decide is the one that sounds like an all-clear.

## Reasons against

Mean density against the volume of a primitive is a crude proxy, and it is
crudest for the fragment class most likely to float. A tank is a closed body
whose flotation depends on whether it is holed, and this basis has no way to ask
that. The flag will be confidently wrong for exactly the case a reader is most
interested in, and stating the basis next to it does not make it right, only
visible.

Emitting a field a standard requires, computed from a proxy, invites the reading
this record disclaims. Somebody will quote the floating count as a flotation
assessment, and the sentence naming the basis will not travel with the number
into the report that quotes it.

The unknown band rests on an uncertainty nobody has measured. Until the
reference density carries a stated uncertainty, the band is either unstated or
picked, and a picked band is a threshold that decides answers without anybody
having argued for its width.

A field with no consequence is a field that will acquire one. Nothing reads this
flag today. The first thing that does will be reading a density comparison as if
it were a flotation model, and this record will be the only place that says it
is not.

Declining to fix the reference density here leaves the record short of a number
it could plausibly carry, and pushes the decision to whoever writes the data set
that supplies it. That is the honest position and it is also one more open end
in a record whose job was to close one.

## What would change this

A decision that gives a fragment a sealed or breached state, or an internal void
volume. That is a change to record 0005's field list, and it would make a real
flotation criterion available. This record is superseded rather than extended if
it lands, because the basis in the output would no longer be a density
comparison.

A source that gives a defensible sea water density with its variation by
temperature, salinity and depth, and an uncertainty the unknown band can be
derived from. No such source was read here. The value is a declared assumption
until one is, and the band is unstated until one is.

The shape set of issue #46 admitting a shape with no closed volume. The unknown
case above anticipates it and the record would need to say whether such a shape
is instead refused.

A validation case with recovered debris that was observed floating or observed
sinking, which is issue #78's territory. One case that this basis gets wrong in
a direction that matters is enough to reopen the choice, and the case is the
measurement this record does not have.

A change to what the standard requires, or a reading of a second standard that
asks for the same field on a different basis. Only the ESA annex was read for
this. `docs/survey/standards.md` records that ISO 24113 and ESSB-ST-U-004 were
not read, and either could carry something this record contradicts.

## What depends on this

The fragment outcome table of record 0009, which gains a column and whose schema
under `schema/` has to carry the three values and reject a fourth.

Record 0005, which is cited above for the field list this record does not change
and for the terminal states the unknown case is keyed to. If the fragment gains
a breach or void field, both records move together.

Record 0010, which owns the refusal of an absent reference density and the
Refused terminal state that reports as unknown here.

Record 0008, which carries the reference density as a declared model assumption
in the manifest.

Issue #46, whose shape set decides whether every admitted shape has a closed
volume.

Issue #95, the report an operator reads, which is where the basis sentence has
to survive the step from artefact to document, and where it is most likely to be
dropped.

Issue #81, the standing the tool prints about itself, since a required output
computed from a proxy is part of what that standing has to say.

`docs/survey/standards.md`, which named this as the required output nothing
carried and which this record answers.
