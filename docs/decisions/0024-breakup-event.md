# 0024. The breakup altitude, and its status as a convention

## Date

2026-08-08

## Status

accepted

## Question

At what altitude does the parent object come apart, where does that number come
from, and what is a reader entitled to believe about it.

Record 0003 fixes that breakup is an input assumption, so the tool is told when
the parent comes apart and does not compute it. That
leaves the number itself, and the number decides how much heating every
component sees. A component released ten kilometres higher enters the dense
atmosphere on its own for longer, and the surviving mass moves with it.

What this record does not answer is what the event does to the state of each
released fragment. That is issue #67. It also does not answer whether a
different trigger, thermal or structural, may replace the altitude. That is
issue #66, and it enters through the same interface record 0003 names.

## Options considered

Compute the altitude from a structural model. Refused by record 0003, which has
no structural model and says so, and repeating that refusal here is the only
work this option costs.

Take a single fixed altitude, adopted from the published reconstructions and
used unless an operator says otherwise. It is what the tools this project will
be compared against do, so a disagreement against them points at the physics
and leaves the assumption alone. It costs accuracy on any object whose real
breakup ran as a sequence of events.

Take a sequence of altitudes, releasing different parts of the
component list at different heights. This is closer to what was observed of the
one large vehicle anybody watched with instruments, and it costs an input an
operator cannot supply. A sequence for a satellite nobody has observed is a
guess with more parameters than the single number, which is worse because it
looks more careful.

Take a distribution and sample the altitude per run, which would put the
assumption's uncertainty into the answer where it belongs. It costs a
distribution nobody has measured. The two reconstructed values available are
from two upper stages of one rocket family, and a distribution fitted to them
would be an invented spread in the shape of evidence.

Refuse to run without an operator-supplied altitude. Honest, and it makes every
operator invent the number this record would otherwise supply once and label.
Record 0010 names that mechanism: a refusal that pushes a guess outside the tool
produces a guess nobody records.

## Decision

The breakup altitude is an input with a default of 78 km geodetic, on the
ellipsoid record 0004 names.

The default is a convention. It is not a computed result, it is not a
measurement of the object being flown, and this record states that in the words
an operator needs, so nothing has to be inferred from its absence in the
physics. Where it came from is a small number of trajectory reconstructions of
recovered upper stages, and it has been repeated since. The two reconstructions
this project could find are in `docs/survey/recovered-debris.md`: 77.8 km for a
Delta II second stage in 1997 and 71.8 km for a Delta II third stage in 2001,
both derived from impact points and tracking data.

The altitude is an operator input in every run. Where the operator supplies it,
the manifest records it as supplied. Where they do not, the manifest records 78
km as a default, per record 0010, and the output says the
run was flown on the convention. The difference a reader needs is not the number
but who chose it.

The value is never a literal in the source. Issue #35 carries the invariant that
refuses a physical constant outside the constants module, and this is one of the
values it exists for: a convention that can be changed in one place and stay
stale in another is a hidden parameter with a version number.

One altitude, not a sequence. A run releases every component in the parent's
list at the same height. Where an object is known to have come apart in stages,
that is represented by describing it as more than one scenario or not at all,
and the tool does not accept a per-component altitude, because an input nobody
can supply for a real satellite is an input that gets filled in with the default
anyway.

The sensitivity to moving the altitude is owed and is not in this record. Issue
#65 requires it to be measured, by running a representative object with the
altitude moved in each direction and recording what happens to the surviving
mass and to the footprint. No number is offered here, because nothing can run:

    git ls-tree -r --name-only origin/main | grep -c -E 'Cargo\.toml|\.rs$|rust-toolchain'
    0

run at `39ed198ce3278045bff1d9fc4a0752d7b5435af9`. Until that measurement
exists, this record states the convention and its origin and says nothing about
what depends on it, and a run may not imply otherwise.

## Reasons

The number has to be somewhere, and the two places it could be are a document
and every operator's head. A default that is written down, labelled as a
convention and recorded in the manifest is auditable; the same number arrived at
independently by each operator is not, and it is what happens if the tool
refuses instead.

78 km, and neither reconstructed value, because the reconstructions are
the evidence the convention rests on, and
because the tools this project will be compared against use the convention.
Picking 77.8 km would look more precise, would be a value from one stage of one
rocket, and would break the like-for-like comparison record 0003 argues for
without buying any accuracy.

Making it an input is what turns the assumption into
something an operator can test. The sensitivity study issue #65 asks for is only
possible if the value can be moved from outside the source, and an operator who
disbelieves the convention can run their own number and see what it costs.

Recording it as a default is the same argument record
0010 makes for every other default, and this is the one where it matters most,
because the number looks like a measurement.

## Reasons against

The strongest argument against this record is that the convention is not
independent of the cases it would be validated against.
`docs/survey/recovered-debris.md` states it plainly: a model that assumes 78 km
and is checked against a case whose breakup altitude was reconstructed at 77.8
km has not tested its assumption, it has met it. So agreement on those two cases
is worth less than it reads, and this record does not fix that. Nothing can fix
it except a case whose breakup altitude was measured directly.

The second is that a single altitude is a simplification of the one event
anybody actually watched. The ATV-1 observation campaign in the same document
measured a sequence: solar arrays separating near 86 km,
first breakup near 75 km, and the main cargo cabin holding together to about 68
km. That spread is wider than the range issue #65 proposes to test the
sensitivity over, so a sensitivity study bounded at plus or minus ten kilometres
would not reach the behaviour of a large pressurised vehicle. The counter is
that a sequence is an input an operator cannot supply, and it is a counter about
what is possible.

The third is that the two reconstructions behind the convention are both upper
stages of one rocket family, which is a narrow base for a number applied to
every satellite. The convention is repeated widely, and repetition is not
independent evidence.

The fourth is that this record fixes a default whose consequence is unmeasured.
The sensitivity is exactly the thing that would say whether the choice matters,
and it is owed, so the argument for 78 km over any other
number in the high seventies is currently an argument from convention alone.

## What would change this

The sensitivity measurement issue #65 requires. If moving the altitude by a few
kilometres moves the surviving mass by little, this record is a small decision
recorded carefully. If it moves it by a lot, the default stops being adequate on
its own and the answer is a sampled altitude under record 0007. The measurement
is possible as soon as issue #26, issue #40 and issue #62 exist, and no figure
is guessed in advance of it.

A breakup altitude measured directly for a spacecraft, which is neither a
reconstruction from impact points nor an observation of a cargo vehicle.
`docs/survey/recovered-debris.md` records that no data had been collected during
the breakup of an unprotected spacecraft, which is the gap the instrumented
recorders were built for, and nothing found in that survey shows it closed. A
measurement of that kind would replace the convention with evidence and would
supersede this record.

A thermally or structurally triggered breakup arriving under issue #66. That
enters through the fragment source interface record 0003 names, so it would make
the altitude one trigger among several, and this record
would state which applies when.

An operator population whose objects are systematically unlike the two upper
stages behind the convention, large pressurised vehicles being the obvious case,
which would argue for a per-class default.

## What depends on this

Issue #65, which owns the sensitivity measurement this record leaves owed and
the operator documentation that has to carry it.

Issue #67, the mechanics of the release, which takes the altitude fixed here as
the moment it acts on, and issue #66, which may supply a different trigger
behind the same interface.

Issue #69, the conservation checks, which run across the event this record
places.

Issue #35, which carries the invariant refusing this value as a literal outside
the constants module.

Record 0003, which fixes breakup as an assumption the tool is told, and which
this record supplies the number for. Record 0010, which
fixes how a default is made visible as a default in the manifest and the output.
Record 0004, which fixes the altitude convention the 78 km is measured in.
