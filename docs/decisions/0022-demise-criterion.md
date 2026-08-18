# 0022. The demise criterion, and what a partly demised fragment is

## Date

2026-08-10

## Status

accepted

## Question

What says a fragment is gone, what the tool reports for the ordinary case where
it is neither gone nor untouched, and how the geometry follows the mass once
material starts leaving the body.

This is the moment the tool exists to compute, and it is one where a wrong
answer is invisible. A fragment that demises at sixty-five kilometres and one
that demises at fifty-five look the same in an output that reports only the
verdict, and a fragment reported as demised that should have reached the ground
removes its own mass from the risk number.

What is not decided here is whether melted material leaves the body. Record 0011
already fixed that, and this record cites it rather than deciding it again:

    git grep -n 'Melt layer removal' origin/main -- docs/decisions/0011-model-boundary.md
    origin/main:docs/decisions/0011-model-boundary.md:124:Melt layer removal. Left out, in the specific sense that melted material is

Read in full, that entry assumes melted material leaves the body immediately,
states the error in both directions, and marks it as an estimate with a
direction and no number. The same record sends the implementation here:

    git grep -n 'melt removal and blowing entries become code' origin/main -- docs/decisions/0011-model-boundary.md
    origin/main:docs/decisions/0011-model-boundary.md:246:body, which are where the melt removal and blowing entries become code.

## Options considered

One stage. The fragment is demised when its temperature reaches the melting
temperature of its material. One comparison, no energy accounting, and no heat
of fusion read at all. It is the shape a code arrives at by collapsing the two
stages below, and it is wrong in a direction that matters: it demises every
fragment earlier than the physics does, which removes surviving mass from the
footprint and lowers the risk number. For a massive component the heat of fusion
is far more energy than the heating up, so the error is not a correction.

Two stages, with the second accounted as an energy bucket. Taken below. Reaching
the melting temperature starts ablation and is not demise, and the mass that
actually leaves is set by the energy that arrives after that.

Two stages with the second written as a recession velocity rather than an energy
bucket. The same physics from the other end: a surface recedes at a rate set by
the net flux over the latent heat and the density, and the mass follows from the
geometry. It is the more natural form for a conduction model that resolves the
wall, and it is the same arithmetic under the default. Not taken as the primary
statement because the energy form is the one that can be checked against a hand
computation without a geometry in the loop, which is what the exact cases of
issue #64 need.

Two stages plus a retained melt layer, which insulates and keeps its mass. This
is the physically richer option and it is not available here. Record 0011 fixed
immediate removal, and record 0001 makes changing what a landed record decided a
supersession rather than an edit, so a retained layer is a new record replacing
0011 rather than a paragraph in a demise criterion.

A threshold on remaining mass, so that a fragment below some fraction of its
initial mass is called demised. Weighed and dropped in a sentence: it is a
convention with no physics behind it, and it hides the outcome this record
exists to represent, which is the fragment that lost half its mass and reached
the ground.

## Decision

Two stages, and the first is not demise.

Stage one is the surface temperature reaching the melting temperature of the
fragment's material. The surface temperature is the only temperature record 0021
exports, and the melting temperature is the material record's. Reaching it
starts ablation and changes nothing about the fragment's mass.

Stage two is the energy. From that moment the surface is held at the melting
temperature, and the net power arriving at the fragment, which is the absorbed
heat load of issue #58 less the radiative loss of issue #61 evaluated at the
melting temperature, goes into latent heat rather than into raising the
temperature. The mass removed over a step is that net power multiplied by the
step and divided by the heat of fusion from record 0016. Where the net power is
negative the fragment cools below melting and stage one has to be met again.

Demise is the remaining mass reaching zero, and nothing else. When the mass a
step would remove exceeds the mass remaining, the fragment demises inside that
step rather than at the end of it, and the instant is located by the event
mechanism issue #40 owes rather than by the step boundary.

The boundary case is strict and is fixed here so that a test can pin it. A
fragment supplied exactly the energy to melt its remaining mass is not demised.
Demise requires the removed mass to exceed what is left, so equality lands on
the surviving side. That is the off-by-one the exact cases of issue #64 already
plan to pin from both directions, and it is written here rather than left for
the test to discover, because a test written against an unstated convention
tests the implementation against itself.

The geometry follows the mass by uniform recession of the heated outer surface.
The shape stays in the set record 0029 fixes, and the fragment does not become a
different primitive as it ablates. For a body with a declared wall the inner
cavity is fixed, so the wall thins from outside and the fragment is demised when
the wall reaches zero, even where its outline has barely moved. For a solid
every outer face recedes by the same distance, so a box stays a box and a plate
demises when its thickness reaches zero rather than when its face area does.

Partial demise is the normal outcome and it is a first class result rather than
a rounding of one. A fragment that lost mass and reached the ground ends in the
Impacted state of record 0005, not in Demised, and the artefacts carry its
surviving mass and its dimensions after ablation alongside its initial ones. The
impact energy issue #72 computes reads the surviving mass, and a reader who
cannot see that the mass moved cannot check the energy.

The criterion reads the melting temperature as the material record wrote it and
does not convert. Record 0016 requires the entry to say which figure it is:

    git grep -n 'liquidus or a nominal figure' origin/main -- docs/decisions/0016-material-record.md
    origin/main:docs/decisions/0016-material-record.md:114:liquidus or a nominal figure, and a source that does not say is recorded as not

A solidus is not turned into a liquidus and a nominal figure is not resolved into
either. What the run does instead is carry the kind into the artefacts beside the
demise altitude, so that two libraries disagreeing by the width of a melting
range are visible as that rather than as a physics disagreement. Two runs whose
melting temperatures are of different kinds are not comparable on demise
altitude, and the output says which kind it read rather than leaving a reader to
assume they match.

A material with no heat of fusion is a refusal and not a fragment that cannot
demise. That is record 0016's rule reaching this criterion rather than a new one,
and it is the refusal that will fire most often:

    git grep -n 'Heat of fusion. Read by the ablation rate' origin/main -- docs/survey/material-data.md
    origin/main:docs/survey/material-data.md:45:Heat of fusion. Read by the ablation rate once the melting temperature is reached.

Read in full, that entry records the heat of fusion as the property most often
absent from a general engineering data sheet, and as the most common reason a
material with a full mechanical characterisation still has no usable demise
record. A fragment whose material lacks it ends in the Refused state of record
0005 rather than being flown to a demise the tool cannot compute.

## Reasons

The two stages are separated because collapsing them is an error with a sign,
and the sign points at a lower risk number. A tool built to be independent of
the parties with an interest in the answer cannot carry a simplification whose
error runs in the reassuring direction without saying so, and the cheapest way
not to carry it is not to make it.

Demise is defined as mass reaching zero rather than as a fraction because the
fraction is the thing an operator would argue about, and because partial demise
is the outcome the footprint is actually made of. A criterion with a fraction in
it has a number nobody measured sitting between the physics and the answer.

The strict boundary is chosen so the exact cases of issue #64 have something to
pin. Either convention is defensible and neither is physical, and what is not
defensible is leaving it unstated so that the test and the code agree because
the same person wrote both.

Uniform recession is chosen because the attitude default is a tumbling average,
so no face of a fragment is preferentially heated over the flight, and a
recession that follows the heating is a uniform one under that default. It also
keeps the shape inside the set, which is what lets the aerodynamic and heating
sources keep answering for the fragment as it shrinks.

Reading the melting temperature as recorded rather than converting it keeps a
modelling choice where the reader can see it. A conversion would be a number
this project invented sitting inside a demise altitude, and the alternative
costs only a field in the output.

## Reasons against

The immediate removal assumption carries no number, and this record inherits
that. Record 0011 states the direction and marks it an estimate, so the
criterion above is exact arithmetic on top of an unquantified assumption, and
the exactness is easy to mistake for accuracy. The strongest form of the
objection is that the two-stage refinement this record is built around may be
smaller than the error in the assumption underneath it, and nothing here
measures either.

The body of issue #62 asks this record to name the code this project agrees with
if it agrees with one, and it is not named. No code read on this board is
recorded as taking immediate removal, and record 0011 names the advanced demise
and ablation model added in SCARAB 4 as the state of the art the omission is
measured against, which is a code the assumption departs from rather than one it
follows. Naming a code without having checked it would be worse than the gap, so
the gap stays.

Uniform recession is a convention with no source, and it is the second
convention in the chain resting on the tumbling default rather than on a
measurement. A fragment that flies stably ablates at the stagnation region and
keeps its back face, which is a different shape and a different ballistic
coefficient from the one this record produces. Issue #56 is where the fixed
attitude option lives, and a fixed attitude with uniform recession is an
inconsistency this record permits.

Holding the surface at the melting temperature during stage two is exact for a
pure substance and approximate for an alloy that melts over a range, which is
most of what a spacecraft is made of. The record refuses to convert between
solidus and liquidus and then assumes a single melting plateau between them, and
those two positions sit awkwardly together.

The energy bucket does not conduct. Under the lumped default there is nothing to
conduct into, so the question does not arise, but once the conduction model of
record 0021 is behind the interface the two will produce different ablation
rates for the same fragment, and how much they differ is unmeasured. The record
fixes one criterion for both models and does not say what that costs the second
one.

Nothing in this record is exercised. There is no source in the tree, no material
library, and therefore no demise altitude this criterion has ever produced:

    git ls-tree -r --name-only origin/main | grep -cE '\.rs$|Cargo\.toml|rust-toolchain'
    0

## What would change this

A demise regression case under issue #64 in which a published demise altitude
for a simple object of stated material, shape and mass disagrees with this
criterion by more than the tolerance that suite fixes. That is the measurement
this record is missing, and the search that would supply the published objects
is the one issues #64 and #80 are both waiting on.

A number attached to the immediate removal assumption, which would come from a
comparison against a retained melt layer rather than from a citation. That is a
supersession of record 0011 rather than a change here, and this record would
follow it.

A recovered debris case under issue #78 in which the tool demised a component
that was found, which is the direction the one stage error runs in and therefore
the direction a residual error in the two stage form would also run in.

A material class whose melting range is wide enough that the solidus and the
liquidus give demise altitudes further apart than the tolerance of issue #64.
That would make the refusal to convert insufficient and would require the
criterion to carry the range rather than a figure out of it.

A decision under issue #56 admitting a fixed attitude in a run, which makes
uniform recession inconsistent with the attitude in that run and forces the
recession model to follow the heating distribution of issue #59 instead.

## What depends on this

Issue #62, which planned this record and stays open on the clauses it does not
discharge: both stages implemented with the heat of fusion from the material
record, partial demise producing a surviving mass and geometry in code, and a
test of a component that reaches melting temperature without the energy to
demise.

Record 0011, whose melt layer removal entry this criterion implements and does
not decide, and whose entry is where a number for that assumption would land.

Record 0021, whose surface temperature is the temperature stage one compares and
whose two models will disagree about the ablation rate once both exist.

Record 0029, whose wall thickness is what thins under recession and whose shape
set the fragment stays inside as it ablates.

Record 0016, whose heat of fusion, melting temperature and the kind of that
temperature this criterion reads, and whose refusal on an absent required
property is the refusal that fires here most often.

Record 0005, whose four terminal states this criterion selects between, and
whose fragment carries the current mass rather than the initial one for exactly
this reason.

Issue #72, which reads the surviving mass and the ablated dimensions to get an
impact energy, and whose casualty threshold is where a partial demise becomes a
different answer rather than a smaller number.

Issue #64, whose exact cases pin the strict boundary fixed here from both sides,
and whose sensitivity table would gain the recession convention as an uncertain
input.

Issues #58, #59 and #61, which supply the absorbed heat load, its distribution
over the body and the radiative loss, and therefore supply the net power stage
two divides by the heat of fusion.

Issue #40, whose event location is what makes the demise instant exact rather
than accurate to a step.
