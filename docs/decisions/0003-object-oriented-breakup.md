# 0003. Object-oriented breakup

## Date

2026-08-08

## Status

accepted

## Question

Does this tool represent a spacecraft as a set of simple bodies that separate at
a breakup event and are then flown independently, or does it represent the
geometry in full and let breakup emerge from the failure of that model?

The term object-oriented has a meaning in this field that is not the software
one, so the question has to be asked in the field's words.

## Options considered

The field divides the codes into two classes, and the division is not this
project's invention. The object-oriented method first reduces the shape of the
spacecraft and its components to basic geometries, then simulates the reentry by
modelling the aerodynamic, aerothermal and ablation processes. The
spacecraft-oriented method directly models the real shape of the spacecraft and
calculates the aerodynamic and aerothermal properties from it. Of the tools in
use, DAS and ORSAT are object-oriented and SCARAB is spacecraft-oriented [1].
The standard comparison of the tools in the field is [2], which is named here
for the reader and was not read for this record.

An object-oriented tool. Fast enough to run many samples on one machine.
Transparent, because each fragment's physics can be read and tested on its own.
Its inputs are a component list an operator can assemble. It rests on a breakup
assumption it cannot check: where the object comes apart is put in by hand, and
several established tools do exactly that at a single altitude near 78 km [3].

A spacecraft-oriented tool. It predicts breakup and never assumes it, which
is the one question the other class cannot ask. It costs a full geometry and a
structural and thermal model of the real vehicle, an input almost no operator
outside the manufacturer can supply, and computing that this project's kickoff
does not have. It is also harder to validate, because a model with more freedom
fits more outcomes.

A hybrid, object-oriented with a structural failure criterion bolted onto the
parent. It looks like the best of both and is the worst option to take first: it
inherits the validation burden of the second class while keeping the coarse
geometry of the first, so a disagreement cannot be attributed to either half.

## Decision

Object-oriented. The tool represents the parent object as a component list,
releases those components as fragments at a breakup event, and flies each
fragment independently through its own aerodynamics, heating and demise.

Breakup is an input assumption, not a prediction, and the tool says so in its
output and not in a footnote.

The seam. A higher fidelity mode is not promised and not designed here, but this
decision must not make one a rewrite. Three interfaces are the seam, and the
trajectory core may only reach them through their interface:

The fragment source, which answers what fragments exist and when they are
released. An implementation that predicts breakup, where this one assumes it,
replaces this one.

The aerodynamic coefficient source, which answers what the drag and lift
coefficients of a body are in a given flow regime. An implementation backed by
a table, a panel method or a solver replaces this one.

The heat load source, which answers what heat flux a body sees. The same
substitution applies.

What those three are called in the source, and what the language calls an
interface, follows from the language and toolchain decision, which is issue #12
and is not settled by this record. What is settled here is that these three
boundaries exist and that nothing else in the tool reaches across them.

The questions this tool refuses to answer, as a direct consequence, and it
refuses them, because answering them badly is the worse outcome:

Whether, when and at what altitude a given spacecraft breaks up. The tool is
told this; it does not compute it.

Whether a particular structural member fails, or at what load. There is no
structural model.

What happens when one fragment shelters another, aerodynamically or thermally.
Each fragment flies alone and sees the free stream.

How a component that stays attached changes the aerodynamics of what it is
attached to. Attached components are not represented as attached.

What the internal temperature distribution of a component is, beyond whatever
the thermal model of the day carries. This record does not fix that model.

## Reasons

The kickoff constrains this to one machine, and the risk number is a
distribution over many runs, so the tool has to run a scenario many times.
That is affordable in the first class and not in the second.

The input exists. A component list with masses, materials and rough shapes is
something an operator can assemble about their own satellite. A validated
structural and thermal model of the flight article usually is not, and a tool
whose input nobody can supply computes nothing.

Per-fragment physics is separable, which is what makes it testable at the level
a gate can refuse. A drag coefficient in a named flow regime for a named shape
is a check a machine can run. The emergent behaviour of a full-geometry model is
not.

The tools prescribed for first stage assessment are of this class [1], so a
comparison against them is like for like, and a disagreement points at physics
and not at method.

## Reasons against

The assumption this class cannot check is the one that matters most. Breakup
altitude sets how long each fragment flies in dense air, so it drives both
demise and footprint. A convention near 78 km is used by several established
tools [3], and the two reconstructed breakup altitudes in the recovered debris
survey, 77.8 km and 71.8 km, straddle it. A tool that assumes 78 km and then
agrees with a case whose breakup was reconstructed at 77.8 km has confirmed
nothing.

Fragment sheltering is not a small effect for a dense satellite, where much of
the mass is inside a structure that ablates around it. Ignoring it biases
survival in a direction this record cannot bound.

The class is old. It is the class the current agency tools are in, and part of
the argument for building this project at all is that those tools are dated. A
new implementation of the same class inherits the same blind spots, and the
answer to that is the seam above.

## What would change this

A published case where an object-oriented prediction and a spacecraft-oriented
prediction disagree materially, against ground truth that settles it. That would
move the seam from a hedge to a plan.

A cheap breakup criterion that is validated and not assumed. If one appears,
the fragment source interface is where it enters, and this record is superseded
and never stretched.

Evidence that fragment sheltering changes surviving mass by more than the spread
this tool already reports. That would make the refusals above too expensive to
keep, and the tool would have to say less.

## What depends on this

The fragment definition and its tree, which is `0005-the-fragment.md`.

Everything in the aerodynamics, heating and demise milestones, because each of
them is written against a fragment that flies alone.

The breakup event and its default altitude, which is issue #65, and which this
record makes a convention.

The validation tiers in `docs/validation/reachable-cases.md`, whose tier 1 cases
are two rocket stages, exactly the shape this class handles best. That is a
reason to distrust a good result there, not to celebrate it.

## Sources

1. Gao, H., Li, Z., Dang, D., Yang, J., Wang, N. "Reentry Risk and Safety
   Assessment of Spacecraft Debris Based on Machine Learning." arXiv:2302.10530,
   2023, section 2.1. <https://arxiv.org/abs/2302.10530>
2. Lips, T., Fritsche, B. "A comparison of commonly used re-entry analysis
   tools." Acta Astronautica 57 (2005), 312 to 323. Named as the field's
   standard comparison; not read for this record.
   <https://ui.adsabs.harvard.edu/abs/2005AcAau..57..312L/abstract>
3. Ailor, W., Hallman, W., Steckel, G., Weaver, M. "Analysis of Reentered Debris
   and Implications for Survivability Modeling." ESA SP-587, 2005, table 1 and
   section 1.
   <https://conference.sdo.esoc.esa.int/proceedings/sdc4/paper/44/SDC4-paper44.pdf>
