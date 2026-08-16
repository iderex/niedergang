# 0019. Attitude: a tumbling average by default, a fixed attitude per component

## Date

2026-08-11

## Status

accepted

## Question

A fragment released in a breakup turns, and this tool does not solve for how.
Record 0011 places six degree of freedom attitude dynamics outside the model
boundary and leaves the treatment that stands in their place to this record.

Three things had to be decided. Which treatment is the default. Whether an
operator may override it, and at what granularity. And how large the error of
the default is, in a form somebody can act on.

## Options considered

One tumbling average for every fragment, with no override. The cheapest thing to
implement and to explain, and it needs no new field anywhere. It costs the cases
that genuinely do not tumble, and it costs them silently, because a run has no
way to say that its assumption did not hold for one component.

A tumbling average as the default, with a fixed attitude selectable per
component. Taken. It costs a field in the scenario and it costs the operator a
judgement, and both costs are visible.

A motion type per component chosen from a named set, which is what the reference
codes do. ORSAT predefines motion and never solves it, and offers four motion
types for a cylinder, and DRAPS extends that to 51 predefined motions over 15
shapes [1, sections 2.1 and 3.1]. This is the richest option that stays inside a
three degree of freedom model. It was dropped for now and not rejected:
nothing read for `docs/survey/existing-codes.md` gives the definition of any
individual motion type, so adopting the idea would mean inventing the set and
naming it after somebody else's.

Propagating the attitude, which is the six degree of freedom answer. Refused
here because it is already refused twice over. Record 0011 places it outside the
model boundary, and record 0005 fixes that a fragment carries no attitude
history, so taking this option would supersede both.

## Decision

The default is a random tumble, averaged over orientation, applied to every
fragment unless the scenario says otherwise. The averaged projected area is the
reference area convention of issue #47 and this record does not restate it.

A fixed attitude is selectable per component. It is never the default, and a
component that selects one is flown at that attitude for the whole trajectory
and not for a phase of it.

The attitude is never propagated. Under either treatment a fragment carries no
orientation history, so record 0005's field list stands and the body frame of
record 0004 carries the average.

The default is visible as a default and not as a value. Record 0010 already
requires that of any value with a defensible answer in the absence of an
operator's opinion, and record 0008 requires the manifest to record the input
after defaults were applied and to say which fields the tool supplied. The
attitude default is one of those fields and this record adds no rule of its own
for it.

The selection is a per component scenario field, and record 0009's component
list does not carry one. That list holds a shape, dimensions, a mass and a
material. Adding a field changes what record 0009 decided, which record 0001
makes a supersession, so the field arrives with whichever
record widens that list and not with this one. Issue #50 is where the widening is
being collected, and the attitude selection joins the wall thickness, the
quantity and the position already waiting there.

## Reasons

An override per component is the granularity every code read already uses. The
object-oriented family assigns motion per component: the
attitude equation "is not directly solved but predefined as specific motion
according to object shapes" [1, section 2.1]. A single treatment for a whole
scenario would be narrower than anything in the survey, and the field it saves is
one field.

Tumbling is the better default of the two because a fragment released in a
breakup has angular rates and nothing to remove them. Nothing read for the
surveys on this board gives a rate distribution for such a fragment, so a fixed
attitude chosen as the default would be a stronger claim resting on less. Where
neither is measured, the treatment that averages over the unknown is the one that
does not have to name it.

Keeping the average is what keeps a fragment a scalar
in the artefacts. Record 0005 argues that case and this record does not reopen
it.

## Reasons against

The default is not conservative and it is not the worst case in either direction.
The table under `What would change this` shows it sitting between the two fixed
extremes for every shape in the set, so an operator who reads the tumbling
default as a safe assumption is reading it wrongly, in both directions and by
different factors.

An average of the area is not an average of the answer. Heating and demise are
not linear in the incident flux, so a body flown at an averaged area does not
reach the temperature history of a body averaged over its outcomes. How far apart
those two are is not quantified here and no source read quantifies it.

The stability judgement is pushed onto the operator. A shape with a strong
aerodynamic restoring moment does not tumble, and under this record the tool
computes no moment and offers no warning, so the person deciding whether the
default holds is deciding it from a drawing. Record 0005 records the same
objection against representing a non-homogeneous component as several fragments,
and it is the same objection here.

The bound below is geometry and nothing else. It bounds the projected area
factor, which is one factor of the ballistic coefficient, and it says nothing
about the coefficient at either attitude.

## What would change this

The cost of the default is the spread between the tumbling average and the fixed
extremes. Record 0007 already names that spread as the attitude uncertainty and
says the shape bounds it. This is the number
behind that sentence.

For a convex body the orientation-averaged projected area is a quarter of the
total surface area, which is the identity issue #47 owns as a test. The extremes
are the projected areas of the two attitudes a body of that shape can be held in.
Every figure below is that arithmetic and nothing else:

    python -c "
    import math
    def box(L,W,T):
        S=2*(L*W+L*T+W*T); return S, S/4.0, L*W, L*T
    for name,(L,W,T) in [('panel 1.000 x 1.000 x 0.003 m',(1.0,1.0,0.003)),('panel 0.500 x 0.400 x 0.002 m',(0.5,0.4,0.002))]:
        S,avg,broad,edge=box(L,W,T)
        print('%-32s S=%.6f avg=%.6f broad=%.6f edge=%.6f  broad/avg=%.4f avg/edge=%.4f broad/edge=%.4f' % (name,S,avg,broad,edge,broad/avg,avg/edge,broad/edge))
    def cyl(d,L):
        r=d/2.0; S=2*math.pi*r*r+2*math.pi*r*L; return S,S/4.0,d*L,math.pi*r*r
    S,avg,broad,end=cyl(0.1,1.0)
    print('%-32s S=%.6f avg=%.6f side=%.6f end=%.6f  side/avg=%.4f avg/end=%.4f side/end=%.4f' % ('cylinder d 0.100 L 1.000 m',S,avg,broad,end,broad/avg,avg/end,broad/end))
    r=0.25; S=4*math.pi*r*r
    print('%-32s S=%.6f avg=%.6f any=%.6f  ratio=%.4f' % ('sphere r 0.250 m',S,S/4.0,math.pi*r*r,(S/4.0)/(math.pi*r*r)))
    "
    panel 1.000 x 1.000 x 0.003 m    S=2.012000 avg=0.503000 broad=1.000000 edge=0.003000  broad/avg=1.9881 avg/edge=167.6667 broad/edge=333.3333
    panel 0.500 x 0.400 x 0.002 m    S=0.403600 avg=0.100900 broad=0.200000 edge=0.001000  broad/avg=1.9822 avg/edge=100.9000 broad/edge=200.0000
    cylinder d 0.100 L 1.000 m       S=0.329867 avg=0.082467 side=0.100000 end=0.007854  side/avg=1.2126 avg/end=10.5000 side/end=12.7324
    sphere r 0.250 m                 S=0.785398 avg=0.196350 any=0.196350  ratio=1.0000

The sphere line is the identity checking itself. A quarter of a sphere's surface
area is its own great-circle area exactly, so the ratio is 1 and attitude cannot
matter for a sphere, which is what record 0011 says of it in words.

What the rest of the table says is that the default is wrong by about a factor of
two towards one extreme and by two orders of magnitude towards the other, for the
shape most spacecraft panels are represented as. A one metre square panel three
millimetres thick presents twice the averaged area held broadside and one
hundred and sixty-eighth of it held edge-on. The elongated cylinder is milder in
both directions and still spans an order of magnitude.

This is a bound on the projected area and it is not yet a bound on the answer.
Turning it into one needs the drag coefficient at each attitude, and this record
does not supply it. The bodies above are not equally shaped to the flow, so the
pressure coefficient at the broadside face and at the edge are different numbers,
and no source read on this board gives either for a held attitude. The low speed
coefficient that decides an impact energy is fixed twice on this board against
two different variables, in issue #54 and issue #70, and until that is one
decision there is nothing to multiply the area factor by. So the figure this
record hands to record 0007 is the area factor, marked as derived geometry rather
than as a measurement, and the coefficient factor stays owed.

Two measurements would change this record. A published
drag or heating ratio between a tumbling and a held attitude for any shape in the
set, which would replace the derived bound with a cited one. And a run of the
same object both ways once there is something to run, which is the clause of
issue #56 no document can discharge.

## What depends on this

Issue #56, which owns this decision and keeps the clauses that need code.

Record 0011, whose first boundary entry is the attitude one and whose heating
cost this record cites and never restates. A bound written into two files under
two owners moves in one of them.

Record 0007, whose attitude entry names the spread this record puts a number on.

Record 0005 and record 0004, which stay true only while the attitude is averaged
and not propagated.

Record 0009 and issue #50, which is where the per component selection becomes a
scenario field.

Issue #47, the reference area convention and the averaging identity this record
uses and does not restate, and issue #46, which fixes the shape set the table
above is written against.

Issue #59, the distribution of heat over the body, where the order of the
surface average and the tumble average is fixed, and issue #57, where a
comparison between the two treatments becomes a case.

Issues #54 and #70, which owe the low speed coefficient and the condition it
applies below, without which the area factor above cannot become an impact
energy.

## Sources

1. Wu Ziniu, Hu Ruifeng, Qu Xi, Wang Xiang, Wu Zhe. "Space Debris Reentry
   Analysis Methods and Tools." Chinese Journal of Aeronautics 24 (2011)
   387-395. doi:10.1016/S1000-9361(11)60046-0.

The full entry is in `docs/survey/existing-codes.md`, which is where it was read.
