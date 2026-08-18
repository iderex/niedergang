# 0029. The primitive shape set, and what every shape answers

## Date

2026-08-10

## Status

accepted

## Question

An object-oriented code represents a component as a simple body, so the set of
bodies it knows is the resolution limit of every answer it gives. Which bodies
are in the set, what must each one be able to answer for the models above it,
and what happens to a component that is not one of them.

Three sub-questions come with it and are answered here rather than left to the
first implementation. The nose radius a stagnation heating correlation needs,
which two of the four shapes do not have. Whether a body is hollow, which is a
property of the shape rather than an afterthought, because a shell and a solid
of the same outline demise at different altitudes. And the tumbling-averaged
projected area, which the attitude default of issue #56 makes the area most runs
actually fly with.

What is not decided here is the characteristic length. Issue #47 owns its
definition, and the flow regime split of issue #52 reads it from there. This
record fixes that every shape in the set must answer for one, and not what the
answer is.

## Options considered

The four shapes the object-oriented family uses: a sphere, a box, a cylinder and
a flat plate. Taken below. It is what the codes this project is measured against
take, and the survey records the set and the one place they differ from each
other:

    git grep -n 'sphere, cylinder, flat plate and box' origin/main -- docs/survey/existing-codes.md
    origin/main:docs/survey/existing-codes.md:142:modeled, i.e. sphere, cylinder, flat plate and box. Only solid objects can be

Read in full, that passage says the four shapes are DAS's and ORSAT's, that only
solid objects can be modelled in DAS, and that both solid and hollow ones can be
analysed in ORSAT [2, section 2.1]. The same survey records SESAM taking a model
built from simple geometric objects of the same family:

    git grep -n 'simple geometric objects like spheres' origin/main -- docs/survey/existing-codes.md
    origin/main:docs/survey/existing-codes.md:102:built from "simple geometric objects like spheres, cylinders, boxes or plates,

The cost of the set is that it is the whole resolution: a component that is not
close to one of the four is described badly or not at all.

Fewer shapes, a sphere alone or a sphere and a cylinder. A sphere needs one
dimension, has an unambiguous nose radius, and its tumbling average is its
projected area, so every question below answers itself. It is also the shape
that survives least like a panel, and a spacecraft is mostly panels and boxes.
Dropped in a sentence: the resolution limit would sit above the components that
decide the answer.

More shapes, the direction DRAPS took:

    git grep -n 'extended to 15 types in DRAPS' origin/main -- docs/survey/existing-codes.md
    origin/main:docs/survey/existing-codes.md:292:uncertainty. The object shapes are "extended to 15 types in DRAPS as well as 51

Read in full, that sentence puts the count at fifteen shapes and fifty-one
predefined motions, including half spheres, cones, and cylinder and box
assemblies, against the four of DAS and ORSAT [2, section 3.1]. Each addition
buys fidelity for a component that would otherwise be approximated, and the
assemblies cost something the other options do not, which is set out under the
reasons below.

A mesh or a free geometry per component, which is the spacecraft-oriented route.
Not available here, and not because of cost. Record 0003 fixes the
object-oriented approach and record 0005 fixes a fragment as carrying one
primitive from this set rather than a mesh, so a mesh option would be a
supersession of two landed records under a shape set's title.

A flat plate as a degenerate box with one small edge, so that the set holds
three shapes and the fourth is a special case of one of them. Weighed and
dropped for a reason that only appears once the nose radius is written down: the
plate is the shape whose nose radius convention costs the most, and a plate
hidden inside the box case is a plate whose convention nobody states.

## Decision

The set is the sphere, the box, the cylinder and the flat plate. Four shapes,
the same four the survey records for DAS and ORSAT, and a plate is its own shape
rather than a thin box.

A component whose geometry is not one of the four is a refusal naming the
component and the shape it asked for. That is record 0005's rule rather than a
new one: a shape outside the set has no coefficients, and the refusal is what
stops a nearest match being substituted silently.

Every shape answers the same seven questions, and a shape that cannot answer one
of them is not in the set:

Total surface area. The radiating area the cooling term of issue #61 reads, and
the input to the two conventions fixed below.

Projected area normal to a given direction. What the drag and the heat load read
when the attitude is fixed rather than averaged.

Tumbling-averaged projected area. Fixed below as a theorem rather than a
per-shape convention.

Volume.

Mass for a given material and wall thickness.

Characteristic length, as issue #47 defines it. This record requires the answer
and does not fix the definition.

Nose radius, by the convention fixed below.

The tumbling average is the total surface area divided by four, for every shape
in the set, and this is a consequence of the set rather than a choice made for
it. All four bodies are convex, and the mean projected area of a convex body
over uniformly distributed orientations is a quarter of its surface area. The
identity is checked here against a direct quadrature over the free stream
direction, on an equal-solid-angle grid with no sampling in it, for one body of
each shape:

    awk 'BEGIN {
      pi = atan2(0,-1)
      N = 2000
      a = 0.30; b = 0.20; c = 0.10
      r = 0.15; h = 0.40
      Rs = 0.25
      Ap = 0.50 * 0.40
      sbox = 0; scyl = 0; ssph = 0; spla = 0; w = 0
      for (i = 0; i < N; i++) {
        ct = -1.0 + 2.0 * (i + 0.5) / N
        st = sqrt(1.0 - ct*ct)
        for (j = 0; j < N; j++) {
          ph = 2.0 * pi * (j + 0.5) / N
          nx = st*cos(ph); ny = st*sin(ph); nz = ct
          sbox += (b*c)*(nx<0?-nx:nx) + (a*c)*(ny<0?-ny:ny) + (a*b)*(nz<0?-nz:nz)
          scyl += (pi*r*r)*(nz<0?-nz:nz) + (2*r*h)*sqrt(nx*nx+ny*ny)
          ssph += pi*Rs*Rs
          spla += Ap*(nz<0?-nz:nz)
          w += 1
        }
      }
      printf "box      quadrature %.6f   S/4 %.6f\n", sbox/w, 2*(a*b+b*c+c*a)/4
      printf "cylinder quadrature %.6f   S/4 %.6f\n", scyl/w, (2*pi*r*r + 2*pi*r*h)/4
      printf "sphere   quadrature %.6f   S/4 %.6f\n", ssph/w, (4*pi*Rs*Rs)/4
      printf "plate    quadrature %.6f   S/4 %.6f\n", spla/w, (2*Ap)/4
    }'
    box      quadrature 0.055000   S/4 0.055000
    cylinder quadrature 0.129591   S/4 0.129591
    sphere   quadrature 0.196350   S/4 0.196350
    plate    quadrature 0.100000   S/4 0.100000

The plate in that check is a rectangle of zero thickness, so its surface area is
twice a face and its tumbling average is half a face. The quadrature is a check
on the identity for these bodies and not a proof of it, and the reason it is
worth running rather than asserting is that the plate and the cylinder are the
two where an implementer is most likely to write a different area and still see
a plausible number.

What follows from this is a rule about the set rather than about any shape in
it. Every shape admitted here must be convex, because the moment one is not, the
tumbling average stops being the surface area over four and becomes a per-shape
integral that each new shape has to supply and each reviewer has to check. That
is the specific cost of the cylinder and box assemblies in the fifteen-shape set
above, and it is the reason the set is not simply grown when a component fits
none of the four.

The nose radius is the radius of the sphere whose total surface area equals the
body's, and it is derived rather than tabulated:

    R_nose = sqrt(S / (4 pi))

For a sphere that returns the sphere's own radius, which is the property that
makes it a convention rather than a substitution. For the other three it is the
radius of the body that presents the same tumbling-averaged projected area,
since a quarter of the surface area is what both quantities are built from, so
the heating radius and the area the drag is computed against describe one
equivalent body instead of two.

What that convention costs is largest for the flat plate, and the plate is the
shape most spacecraft panels are represented as. Against the other convention
that was available, half the smallest dimension, which is the edge radius a
plate presents flying edge-on:

    awk 'BEGIN {
      pi = atan2(0,-1)
      printf "%-26s %10s %10s %10s %10s\n", "body", "S m2", "Req m", "half-min m", "ratio"
      L=0.5; W=0.4; t=0.003
      S = 2*L*W + 2*(L+W)*t; Req = sqrt(S/(4*pi)); hm = t/2
      printf "%-26s %10.6f %10.6f %10.6f %10.2f\n", "plate 0.5x0.4x0.003", S, Req, hm, Req/hm
      a=0.30; b=0.20; c=0.10
      S = 2*(a*b+b*c+c*a); Req = sqrt(S/(4*pi)); hm = c/2
      printf "%-26s %10.6f %10.6f %10.6f %10.2f\n", "box 0.30x0.20x0.10", S, Req, hm, Req/hm
      r=0.15; h=0.40
      S = 2*pi*r*r + 2*pi*r*h; Req = sqrt(S/(4*pi)); hm = r
      printf "%-26s %10.6f %10.6f %10.6f %10.2f\n", "cylinder r0.15 h0.40", S, Req, hm, Req/hm
      R=0.25
      S = 4*pi*R*R; Req = sqrt(S/(4*pi)); hm = R
      printf "%-26s %10.6f %10.6f %10.6f %10.2f\n", "sphere R0.25", S, Req, hm, Req/hm
    }'
    body                             S m2      Req m half-min m      ratio
    plate 0.5x0.4x0.003          0.405400   0.179613   0.001500     119.74
    box 0.30x0.20x0.10           0.220000   0.132314   0.050000       2.65
    cylinder r0.15 h0.40         0.518363   0.203101   0.150000       1.35
    sphere R0.25                 0.785398   0.250000   0.250000       1.00

A factor of 119.74 in radius for a three millimetre panel, against 2.65 for the
box and 1.35 for the cylinder, and exactly 1 for the sphere. The convention
taken is the larger radius everywhere it differs, and the larger radius is the
lower stagnation heat flux in every correlation of the usual form, so this
choice leans towards survival for the shape that dominates a spacecraft's
surface. The factor that turns 119.74 into a heat flux ratio is the exponent of
the nose radius in whichever correlation issue #58 takes, and it is not fixed
here. For an exponent of minus one half the plate ratio above is 10.94, computed
as the square root of 119.74, and that number is written as conditional on #58
rather than as a property of this record.

Hollow and solid are distinguished by a wall thickness on the shape, and the
thickness is a dimension rather than a flag. A body with no wall thickness
declared is solid. A body with one is a shell of that thickness measured inward
from the outer dimensions, its material volume is the outer volume minus the
inner one, and its mass is that volume times the density from the material
record of record 0016. The outer surface area, the projected areas and the nose
radius are all properties of the outer dimensions and do not move when the wall
does.

Two refusals come with the wall thickness, both under the rule of record 0010.
A wall thickness that does not fit, meaning twice the wall exceeds the smallest
outer dimension, is refused naming the component, the thickness and the
dimension it exceeded, rather than being clamped to a solid. And a wall
thickness of zero is refused rather than read as solid, because zero is what a
spreadsheet column produces when the value was not known.

The internal cavity is empty and radiates nothing. A shell in this tool is a
mass and a heat capacity distributed in a wall, not an enclosure with a
temperature inside it. That is record 0003's each-fragment-flies-alone rule
reaching inside a single body, and it is stated here because a reader looking
for it would otherwise look in the thermal model.

## Reasons

The four shapes are taken because they are what the codes an operator will
compare this tool against take, and comparability against a published answer is
worth more at this stage than fidelity nobody can check. The survey establishes
that the set is DAS's and ORSAT's rather than an assumption made here, which is
the difference between a decision and a habit.

The plate stays a separate shape because it is the one whose conventions are
worth stating out loud. A plate is where the nose radius has no natural answer,
where the tumbling average is furthest from the face area somebody would write
down, and where the aerodynamic model of issue #54 is recorded as weakest. All
three of those are properties of the plate and none of them is visible if the
plate is a box with a small edge.

Convexity is required because it is what makes the tumbling average a theorem.
The alternative is a table of per-shape averaged areas, and a table is a place
where a wrong entry looks exactly like a right one. Deriving the same numbers
from the surface area means one property is checked instead of four, and the
check is the quadrature above rather than a reviewer's memory.

The nose radius is derived from the surface area rather than from a smallest
dimension because it keeps one equivalent body behind both the heating and the
averaged area. A convention that gives the heating a body of one size and the
drag a body of another is the kind of inconsistency that produces a defensible
number nobody can reconstruct.

Deriving it also means the convention cannot be applied inconsistently between
shapes, which is the failure the alternative invites: half the smallest
dimension is an obvious rule for a plate and an arbitrary one for a cylinder,
where the smallest dimension is a radius that is already the physical nose
radius for one attitude and not for the other.

Making the wall thickness a dimension rather than a flag is the same argument
record 0016 makes about a constant standing in for a curve. A flag says a body
is hollow, and a dimension says how hollow, and the demise altitude depends on
the second.

## Reasons against

The strongest argument against this record is that the nose radius convention is
not sourced. Nothing on the default branch carries one, and the scan is run
against that branch rather than against a tree holding this record, since a scan
including this file can no longer show what the convention was checked against:

    git grep -n -i 'nose radius' origin/main -- docs/ ; echo "exit=$?"
    exit=1

The surveys read the reference codes for their shape sets and their motion
types, and none of them recorded what any of those codes uses for the nose
radius of a box or a plate. So the convention above is derived and internally
consistent, and it is not the convention any comparable code is known to use.
The Done-when of issue #46 asks for the convention recorded with its source, and
this record supplies the first half. A disagreement with another code on the
demise of a panel could be this convention, and today nothing in the tree would
let a reader tell.

The convention is also the one that leans towards survival on the shape that
dominates a spacecraft's surface, which is the direction that produces a higher
risk number rather than a lower one. That is the safer direction for a safety
tool and it is still a lean, and a tool that leans is a tool whose comparisons
against others are biased in a knowable direction. Saying which way it leans is
worth more than the lean being small, and issue #90 is where that statement
belongs.

Four shapes is the resolution limit and it is a coarse one. A tank is a
cylinder with domed ends, a reaction wheel is a cylinder with most of its mass
in a rim, and a harness is nothing in this set at all. Each of those is
represented by a body with the same mass and the wrong area, and the error goes
into the ballistic coefficient, which is the thing the along-track spread of the
footprint is made of. The fifteen-shape set exists because somebody found this
insufficient.

Requiring convexity forecloses the cheapest way to grow the set. Assemblies are
how DRAPS reached fifteen shapes, and this record makes an assembly ineligible
rather than expensive. A future shape that is genuinely needed and not convex
would have to supply its own tumbling average and would break the one-property
check above, so the rule that keeps the set honest is also the rule that will
have to be argued with first.

The empty cavity is an assumption with no number attached. A shell that encloses
a cavity does radiate into it and does re-absorb some of what it radiates, and
this record states the simplification without bounding it. Record 0011 is where
an unquantified exclusion belongs, and this one is not in it yet.

## What would change this

A source establishing what the comparable codes use for the nose radius of a box
or a plate. That is the missing half of this record and it would either confirm
the convention or replace it, and either outcome is worth more than the argument
above. The search that would find it is the one issue #64 and issue #80 are
already waiting on, over the reentry conference proceedings.

A demise comparison under issue #64 in which a panel demises at an altitude that
moves by more than the tolerance when the nose radius convention is changed from
the equivalent sphere to half the thickness. That is the measurement that says
whether the factor of 119.74 above matters, and until it exists the size of this
decision is unmeasured.

A component in a real validation scenario under issue #78 that none of the four
shapes represents without an error a reader can see. That is the evidence that
would move the set rather than the conventions inside it, and it is the form the
argument for a fifth shape has to take.

A decision under issue #47 whose characteristic length cannot be answered by one
of these four shapes, which would mean the set and the length definition were
chosen against each other.

## What depends on this

Issue #46, which planned this record and stays open on the clauses it does not
discharge: each shape answering every question in code, hollow and solid forms
distinguishable in an implementation, and the test comparing each shape's areas
and volume against hand-computed values.

Record 0005, which fixes a fragment as carrying one primitive from this set and
names issue #46 rather than a file. This record is the file, and the refusal
0005 describes for a shape outside the set is refused against the four here.

Issue #47, which defines the characteristic length and the reference area
convention that every shape here has to answer for, and which is where a
disagreement between the tumbling average fixed here and a reference area would
surface.

Issue #52, whose Knudsen number reads the characteristic length per fragment,
and issue #54, whose pressure model is evaluated over these shapes and which
records the plate as the shape it is weakest on.

Issue #58, which owns the stagnation heating correlation and therefore owns the
exponent that turns the nose radius ratios above into heat flux ratios. The
number 10.94 in this record is conditional on that exponent and is not a result
of this one.

Issue #56, whose tumbling default is what makes the averaged area the area most
runs fly with, and whose fixed-attitude option is what makes the per-orientation
projected area a question every shape still has to answer.

Issue #60 and issue #62, which read the wall thickness fixed here: the thermal
model through the conduction path across it, and the demise criterion through
the mass that has to be melted.

Issue #50, whose component list carries the shape, the dimensions and the wall
thickness an operator writes, and whose mass consistency check compares a
declared mass against the mass this record's shapes compute.

Record 0016, whose density and required properties are what a wall thickness
turns into a mass, and record 0010, under which the two wall thickness refusals
above are placed.
