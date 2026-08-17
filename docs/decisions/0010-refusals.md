# 0010. What the tool refuses to compute, and what it never emits

## Date

2026-08-08

## Status

accepted

## Question

What does the tool do when something the calculation needs is absent, and what
does it decline to put in its output even when the calculation succeeded.

The two halves are one question because they fail in the same way. A reentry
risk tool produces a figure that somebody quotes, and the figure carries no
memory of what was missing when it was computed. A substituted default and an
over-confident presentation both produce a number that reads exactly like one
that was earned.

What this record does not answer is which values are correct for any particular
input. It answers what happens when there is no value at all.

## Options considered

On a missing input.

Substitute a default and continue. This is what a program does if nobody decides
otherwise, and it is the single failure this project has to avoid. A melting
point that was not in the library and a melting point that was are
indistinguishable downstream, so the run produces a demise altitude with the
same confidence either way. The cost is not that the answer is wrong. The cost
is that nothing in the artefact says which answer it is.

Substitute a default and print a warning. Better, and it fails for a reason that
has nothing to do with the physics. A warning is a line on a terminal that
belongs to the person who ran the tool, and the artefact travels without it. The
number reaches a reader who never saw the warning, and that is the normal case.

Substitute a default and mark the affected outputs. This is the honest form of
the option above and it is genuinely defensible. It costs a marking that has to
reach every derived quantity, which means every aggregation has to carry
provenance for the inputs beneath it, and a marking that is dropped by one
aggregation is worse than no marking at all because it looks complete.

Refuse, with no defaults anywhere. Strongest and simplest to reason about. It
costs usability in the places where a default is genuinely correct and where
refusing would make an operator invent a value, which is how a made-up number
enters the run through the front door.

Refuse a missing input, permit a named default only where a default is a real
answer, and make a default visible as a default wherever it is used. Taken
below.

On the moment a refusal happens.

Stop at the first defect found. Cheapest, and it turns fixing a scenario into as
many runs as it has defects, each one revealing the next.

Check the whole scenario before flying anything, and report every defect
together. More work to implement, because it requires the checks to be
separable from the calculation that uses the values.

On a refusal that only becomes knowable during flight.

Abort the run at the first one. This is the obvious reading of "the run stops",
and it does not survive contact with record 0008. Samples are independent and
may run in any order on any number of threads, so which in-flight refusal is
reached first is a property of the scheduler. A run that aborts at whichever
refusal arrived first reports a different fragment on a different machine from
the same seed, which is the determinism property removed by the error path.

Fly every sample to its end, mark the fragments that could not be flown, report
them in an order the run fixes, which the schedule cannot move, and withhold
every aggregate that a marked fragment would have entered. Taken below.

## Decision

A missing input is a refusal and never a default.

A refusal names four things: the input, the field, where in the scenario it came
from, and what would fix it. The last one is not decoration. An operator who has
never fetched a space weather file has no reason to know one exists, and a
refusal that says the indices are missing without naming the command that gets
them has diagnosed the problem to somebody who cannot act on it.

There are two moments at which a refusal can happen, and they behave
differently.

A pre-flight refusal is one that can be decided from the scenario and the data
sets before any integration begins. The whole scenario is checked first, every
defect is collected, and the run reports all of them together and exits non-zero
without writing artefacts. Record 0009 fixes that a run writes five artefacts and
writes all of them; a pre-flight refusal is the case where there is no run, so
there is no partial set to misread.

An in-flight refusal is one that only becomes knowable while a fragment is being
flown. The fragment ends in the Refused terminal state that record 0005 names,
and the run does not abort. Every sample is flown to its end, the refused
fragments are reported in an order derived from the sample index and the fragment
identity, which the order they were reached in cannot move, the run writes its
artefacts with the refused count in them, and it exits non-zero.

The reason the run does not abort mid-flight is record 0008. One seed reproduces
one run exactly whatever the thread count and whatever order the samples finish
in, and an abort at the first refusal reached makes the reported fragment a
function of the scheduler. An error path that is not deterministic is a hole in
the property exactly where a reader is most likely to look. Flying every sample
also gives the operator every fragment that could not be flown in one pass,
which is the same argument the pre-flight case makes.

Record 0005 says of a missing material property that "the run stops or the
fragment is reported as refused". This record is where that choice is made, and
the deciding question is when the absence becomes knowable. A material property
named in the scenario is knowable before flight and stops the run. A regime a
trajectory turns out to enter is not, and it marks the fragment.

The inputs whose absence is a refusal. Each entry says where the obligation
comes from.

The scenario schema version. A file with no version, a version the tool does not
know, and a version older than the current one are all refused, the last one
naming the command that converts it. Record 0009 fixes this and this record does
not reopen it.

A material property that a model reads. Melting point, heat of fusion, specific
heat, thermal conductivity and emissivity are read by the thermal model and the
demise criterion, and a component whose material lacks one of them is refused by
name. Record 0005 requires a fragment to reference the library,
and record 0007 states that a missing property is a refusal and never a
sampled guess.

A material the library does not contain at all. The refusal names the component
and the material string as it was written in the scenario.

A shape outside the fixed set. Record 0005 requires one primitive from the set
that issue #46 fixes, and states that a shape outside it is a refusal, with no
nearest match offered.

The space weather indices, where the selected atmosphere model needs them.
Record 0006 fixes this, with no fallback, no substitution and no flag that turns
it into a warning, and requires the refusal to name the model, the index, the
epoch it was needed for and the fetch command.

A query outside an atmosphere model's stated validity range. Record 0006 fixes
that the answer is a refusal, checked against the range the model itself
returns. This one is in-flight: whether a trajectory
leaves the range is not knowable from the scenario.

An aerodynamic coefficient absent for a shape in a regime the trajectory enters.
In-flight, for the same reason.

A data set that a manifest identifies and the repeating machine does not have,
in the mode that reproduces a run from a manifest. Record 0008 fixes this as a
refusal.

The population data set and its reference year, when a risk number is asked for.
The number has no meaning without them and they are not defaultable, because
choosing a grid on the operator's behalf is a licensing decision as well as a
numerical one. Entry 5 of issue #1 is where the shipped grid is decided, and
whichever way it is answered, an absent grid is a refusal.

Where an input is absent, is not on the list above, and is not a default named
below, that is an oversight and a defect report against this record. The list is
the decision itself, in the same sense record 0011's list
is the model boundary.

What is a default, and how a default is visible.

The seed. A run with no seed draws one at start and records it in the manifest.
The property record 0008 promises is that a recorded seed reproduces a run, and
a seed the tool chose and wrote down satisfies it, where a refusal would make
every operator invent one. It is a default and it is marked as one.

The sample count and any other value that has a defensible answer in the absence
of an operator's opinion. Record 0007 makes the sample count an input recorded in
the manifest, and this record does not change that.

Every default is recorded as a default in the manifest, not merely as a value,
and printed in the output. The distinction a reader needs is not what the value
was but who supplied it, so a manifest that records the input after defaults were
applied, as record 0008 requires, has to record which of those fields the
operator wrote and which the tool did. A manifest in which the two are
indistinguishable satisfies record 0008 and defeats this one.

What the tool never emits, whatever the run did.

A single impact point without its distribution. Record 0007 fixes that the
footprint is sampled, and the linearised mode is labelled in its own output and
is never the default.

A risk number without the population data set and the reference year it was
computed against. Record 0009 puts the population basis in the risk summary and
this record makes its absence a refusal to emit.

An artefact without the identity of the models that produced it. Record 0006
requires every artefact produced against the static default atmosphere to carry
the model identity, and record 0007 requires every run to state which of its
distributions are assumptions.

An aggregate that a refused fragment would have entered. A footprint, a casualty
area or an expected casualty number computed over a set that is missing a
fragment nobody could fly is a number that reads as complete. The count of
refused fragments appears; the aggregate does not.

A compliance statement. Issue #21 states this and entry 9 of issue #1 is where
the project's position on the thresholds is decided. Until it is answered the
tool says nothing about them, and if the answer permits a comparison, this record
is superseded, and the superseding is written down where a reader meets it.

A distribution rounded into a reassuring sentence. The tool emits the
distribution. What a report says about it is issue #95 and entry 10 of issue #1,
and neither of them may reach back into what the tool itself prints.

## Reasons

The failure this project has to avoid is a plausible number produced from an
input that did not support it, and every option except refusal leaves that number
reachable. The marking option is the only serious competitor and it fails on
reach, and its principle is sound: a mark has to survive every aggregation between
the missing property and the risk figure, and the aggregations are exactly where
this tool does the work a reader cannot check by inspection.

Refusing is also the only option that is cheap to prove. A refusal is a run that
stopped and said why, which a test can assert on by message. A marking is a
property of an artefact that has to be checked at every level of derivation, and
a test suite that covers it is larger than the feature.

Collecting defects before flying is worth its implementation cost because of who
runs this tool. An operator assembling a component list of any size will have
several materials missing at once, and a tool that reveals them one run at a time
teaches them to stop reading the messages.

Withholding the aggregate is the same argument as the refusal itself, one level
up. A footprint over the fragments
that could be flown is a real number with a real meaning, and the meaning is not
the one a reader will take from it.

## Reasons against

The strongest argument against this record is that a refusal is not always the
safe answer. An operator with an incomplete material library and a deadline gets
nothing from this tool, and the alternative they take is not a better tool, it is
a spreadsheet with a guessed melting point in it and no record that it was
guessed. A tool that refuses is a tool that is not used, and a tool that is not
used protects nobody.

The counter is that the failure mode this record is against was measured on other
people's outputs, and that a guess made outside the tool is
at least a guess somebody knows they made. That counter is an argument and not a
measurement, and it is written here as an argument.

The second is the boundary between pre-flight and in-flight. It is a line the
implementation draws, and it can move: a check that is in-flight
today because nothing computes the regimes a trajectory will enter becomes
pre-flight the day something does. So an operator cannot predict from this record
alone whether a given defect stops the run or marks a fragment, which is a real
cost in a rule whose whole point is predictability.

The third is that flying every sample after a refusal spends the full cost of a
run that will not produce a risk number. On a large sampling job that is a lot of
work for an answer that is already known to be incomplete, and the argument for
it is determinism.

The fourth is that the list of refusals is a list, and lists in documents go
stale. This one names the record or the issue behind each entry so that a reader
can check it against the authority, which reduces the drift and does not remove
it.

Against the never-emit half: the compliance entry in particular denies an
operator the one comparison they most want, and the reason is a decision that has
not been taken. That is defensible while entry 9 of issue #1 is open and it is
not a permanent position, and a reader who takes it as one has read this record
wrongly.

## What would change this

An answer to entry 9 of issue #1 permitting the tool to print the comparison
against a threshold, which would supersede the never-emit half of this record.

An answer to entry 4 of issue #1 on material data, if it introduces a class of
property that is genuinely defaultable with a stated uncertainty. That would
move an entry from the refusal list to the default list, which is a change to
this record and not an implementation detail.

A measurement showing that the pre-flight check set cannot be separated from the
calculation without duplicating it. Nothing has been measured, because nothing is
implemented. The measurement is possible as soon as issue #26 exists, and if the
duplication were large the answer would be a narrower pre-flight set with the
remainder in-flight, which weakens the reporting promise and leaves the refusal
rule standing.

Evidence from use that refusing a run costs more safety than it buys, which is
the argument in the first paragraph of the reasons against. It would arrive as a
report of somebody working around the tool, which is prose, and it would
be worth a record even so.

## What depends on this

Issue #35, the greppable invariants gate, which names the enforceable half of
this record: no default substituted for a missing material property. The rule it
refuses is written here and the fixture that proves it bites is written there.

Issue #48, the material record, whose required property list is what makes the
material entries above decidable. A property that is required by the schema and
absent from a row is a pre-flight refusal by this record.

Issue #46, the shape set, which fixes what a shape outside the set means.

Issue #52 and issue #55, the regime boundaries and the bridging relation, which
decide where an absent coefficient can arise and therefore how often the
in-flight branch is taken.

Issue #71, the sampling implementation, which is where the deterministic ordering
of refused fragments is built, and issue #73, the population exposure, which
carries the grid and reference year this record refuses to emit a number without.

Issue #81, the validation standing every run prints, which is the other statement
an artefact carries without being asked and which shares whatever mechanism
writes the refusal counts.

Issue #95, the report an operator reads, which inherits the last never-emit entry
and may not undo it.

Records 0005 and 0006, which state the refusal rule from their own side and name
this record as the place it is written down. Record 0009, whose artefact set is
what a run withholds when an aggregate is refused.
