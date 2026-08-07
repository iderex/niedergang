# 0005. The fragment

## Date

2026-08-08

## Status

accepted

## Question

What is the unit this tool flies, heats, demises and drops, what does it carry,
and how does a piece of metal on the ground get traced back to a part of the
satellite an operator described?

## Options considered

A flat list of fragments with no link to where they came from. Simplest, and it
computes the same risk number. The output is a scatter of impact points that
nobody can act on, because the question an operator asks after seeing a
footprint is which part of the satellite is causing it.

A fragment carrying its full history: every state it passed through, its
attitude over time, its neighbours, what it touched. Richest and it makes the
artefacts enormous for a sampled run, most of it never read, and it invites
physics the model does not have. A field that exists gets used.

A fragment as a small fixed record with a parent link and a named lifecycle,
taken below.

One shared body type for both the parent object and its fragments. Attractive
because a parent is also a body that flies. It collapses two things that end
differently: a parent ends by coming apart, a fragment ends on the ground, in
the air, or nowhere. Sharing the type means every consumer has to ask which kind
it has, and the tree is what tells them apart anyway.

## Decision

A fragment is a record with the following fields and no others. Each is here
because something downstream reads it.

Identity. A value that is unique within a run and stable across the run's
artefacts. Without it nothing can be cross-referenced between the trajectory
output, the impact list and the manifest.

Parent, and the event that released it. The identity of the fragment or parent
object it came from, together with the breakup event that produced the release.
The chain of parents ends at the object the operator described, and through it
at the component in the input. This is the field the whole tree rests on, and it
is the one that is expensive to add later, because every artefact and every
aggregation would have to be reopened.

Shape. One primitive from the shape set, which is fixed by issue #46, not a mesh
and not a free geometry. The aerodynamic and heat load sources answer per shape,
so a shape outside the set has no coefficients and is a refusal rather than a
nearest match.

Dimensions. The parameters that shape needs, and only those.

Mass. Current, not initial. Ablation changes it, and the demise criterion and
the impact energy both read the current value.

Material. A reference into the material library rather than a copy of its
values, so that a run reads one version of a property and the manifest can
record which. A material whose required property is missing is a refusal, and
issue #21 is where that rule is written down.

Thermal state. Whatever the thermal model of the day carries, which is a lumped
temperature and the energy absorbed at first. The interface is fixed here and
the contents are not, because conduction enters behind the same interface.

Trajectory state. Position and velocity in a stated frame at a stated epoch. The
frame and the time scale are fixed by issue #14, and this record does not
duplicate that choice.

What a fragment does not carry, which matters as much:

No attitude history. The attitude treatment is decided elsewhere, and where it
is a tumbling average, a per-fragment attitude history would be a field with
nothing behind it.

No internal structure. A fragment is homogeneous, of one material. A component
that is not is represented as more than one fragment or not at all.

No link to a sibling, no contact, no shadowing. `0003-object-oriented-breakup.md`
says each fragment flies alone and sees the free stream, and a sibling link
would be the first step to quietly contradicting it.

No knowledge of the run. A fragment does not know how many samples there are or
what the others did. Aggregation happens above it.

The lifecycle. A fragment is created, flown, and ends in exactly one of four
named states. A run reports the count in each, and every fragment is in one:

Demised. The demise criterion was met. The fragment stops being flown and its
mass leaves the surviving set.

Impacted. It reached the ground, with an impact point, a velocity and an energy.

Still flying. The simulation ended before the fragment did. This is a real
outcome and not a bug, and a run with a non-zero count here says so in its
output, because a footprint computed from a run that stopped early is missing
mass.

Refused. Something the fragment needed was absent, most often a material
property. The run stops or the fragment is reported as refused, under the rule
issue #21 holds, naming the fragment, the field and what would fix it. This
state exists so that a missing input is visible in the output rather than
silently replaced by a default.

The tree survives into the output. Every artefact defined by issue #20 that
names a fragment carries its identity and its parent link, so that an impact
point can be followed back through the breakup that released it to the component
in the input. An artefact that lists impact points without the parent link does
not satisfy this record.

## Reasons

The question that leads to a design change is which part of the satellite is
causing the risk, and only the tree can answer it. A risk number on its own tells
an operator that there is a problem; the tree tells them where.

Naming the terminal states, rather than treating an end as an absence, is what
makes a partial run readable. Three of the four states are ways a run can be
less complete than it looks, and a tool that reports only impacts hides all
three.

A fixed field list keeps the fragment cheap. A sampled run carries many of these
at once, and every optional field is a branch in the code that reads them.

## Reasons against

The parent link costs memory and artefact size in exactly the case where the
tool is most expensive, a large sample count over a large component list. A user
who wants one number pays for a tree they will not read.

A fixed field list is a promise this record cannot keep on its own. The thermal
state is already written as model-defined, which is a hole in the fixity, and
the honest reading is that the fragment is fixed except where it is not.

Making refusal a terminal state rather than an exception makes aggregation
harder: a run with refusals has no valid total, so every consumer has to decide
what to do with a partial set. That cost is deliberate, and somebody will
eventually propose a flag to ignore refused fragments. That flag is the thing
this project exists to refuse.

Representing a non-homogeneous component as several fragments pushes a modelling
judgement onto the operator, who is the person least equipped to make it and the
one whose input decides the answer.

## What would change this

An attitude decision that propagates per-fragment attitude rather than averaging
it. The no-attitude-history line would be wrong, and the field would have to
enter with the artefacts that carry it.

A thermal model with conduction that needs a spatially resolved state. The
thermal state field would stop being a scalar and the artefacts would grow.

A demise criterion that depends on a fragment's neighbours. That contradicts
`0003-object-oriented-breakup.md` before it contradicts this record, and both
would be superseded together.

## What depends on this

The breakup event and what it does to the state, in the breakup milestone, which
creates fragments and sets their parent links.

The output artefacts of issue #20, which are required above to carry the tree.

The refusals record planned by issue #21, which owns the fourth terminal state.

Fragment identity and the tree that survives into the output, issue #51, which
is where this record becomes code.

The footprint and the casualty area, which aggregate impacted fragments and must
not silently aggregate over refused ones.
