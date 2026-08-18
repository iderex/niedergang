# 0021. The thermal model, and the criterion that says which one applies

## Date

2026-08-10

## Status

accepted

## Question

Whether a fragment is treated as one temperature or as a temperature field
through its wall, which of the two the tool does by default, and what the
trajectory core asks for so that it does not know which one answered.

The choice is not a refinement. For a thick or poorly conducting component the
surface runs hotter than the mean, and the surface is what the radiative cooling
term and the demise criterion both read, so a lumped body reaches melting later
than a real one and demises at a lower altitude or not at all. Most of what
survives to the ground is exactly the thick, poorly conducting part of the set.

What is also decided here is the shape of the criterion that says the default is
being used outside its range. Issue #60 asks for that criterion as a
dimensionless number rather than as a rule of thumb, and asks the tool to report
it per fragment.

What is not decided here is what happens once the surface reaches melting. That
is the demise criterion of issue #62, and record 0011 already fixes that melted
material is assumed to leave the body immediately:

    git grep -n 'Melt layer removal' origin/main -- docs/decisions/0011-model-boundary.md
    origin/main:docs/decisions/0011-model-boundary.md:124:Melt layer removal. Left out, in the specific sense that melted material is

## Options considered

The lumped mass model alone. One temperature per fragment, one ordinary
differential equation, and no length scale in the thermal state at all. It is
the baseline every comparable code offers and it is a limit any better model has
to reproduce. Its cost is the whole of the question above, and taking it alone
means the departure is never measured because there is nothing to measure it
against.

A one-dimensional through-thickness model alone. Strictly the larger model, and
it contains the lumped one as a limit. Taken alone it costs the comparison in
the other direction: the limit is never checked, and a conduction solver whose
thin-wall answer is wrong is a solver nobody catches, because the wrong answer
still looks like a temperature.

Both, as two model types the caller selects between, side by side rather than
behind one interface. Rejected. The trajectory core would then hold a branch on
which thermal model it has, and every consumer of a temperature would grow the
same branch. Record 0005 already fixes the other arrangement.

Both behind one interface, with the lumped model as the default. Taken below.
The survey records the same pair inside the codes this project is measured
against:

    git grep -n 'zero-dimensional or' origin/main -- docs/survey/existing-codes.md
    origin/main:docs/survey/existing-codes.md:289:ballistic model for trajectory prediction and zero-dimensional or

Read in full, that passage says DRAPS adopts a zero-dimensional or
one-dimensional heat conduction approach for ablation analysis, which is the
same as ORSAT [2, section 3].

Three-dimensional conduction over a meshed body. This is the spacecraft-oriented
route and it is not available here for the same reason a mesh is not available
to record 0029: record 0003 fixes the object-oriented approach and record 0005
fixes a fragment as homogeneous and of one material. A three-dimensional field
would be a supersession of two landed records under a thermal model's title.

## Decision

Two implementations, one interface, and the lumped mass model is the default.

The interface is the one record 0005 already fixed, and this record names its
two directions rather than restating the field:

    git grep -n 'conduction enters behind the same interface' origin/main -- docs/decisions/0005-the-fragment.md
    origin/main:docs/decisions/0005-the-fragment.md:71:the contents are not, because conduction enters behind the same interface.

Per step, the model is given the absorbed heat flux over the body and the step
length, and it returns the surface temperature. The core keeps no temperature of
its own. Everything else the model needs is the model's own state, and the
lumped model's state is one temperature while the conduction model's is a
profile through the wall. A consumer of the surface temperature cannot tell
which one produced it, and that is the property the interface exists for.

The surface temperature is the only temperature that leaves the model. The
radiative cooling of issue #61 radiates at it, the demise criterion of issue #62
compares against it, and a mean temperature is not exported at all. Exporting
both is how a consumer ends up reading whichever one it happened to be handed.

The conduction length is the wall thickness where record 0029 declares one, and
the volume over the total surface area for a solid body. Two definitions rather
than one, and the reason is measured rather than asserted. The two agree in the
thin-wall limit and part company as the wall thickens, and because the length
enters the Fourier number below as a square, the departure is larger there than
it looks in the length:

    awk 'BEGIN {
      pi = atan2(0,-1)
      printf "%-26s %10s %10s %8s %10s\n", "shell", "V/S m", "wall m", "V/S / t", "Fo factor"
      R=0.25
      split("0.003 0.010 0.050", ts, " ")
      for (k=1; k<=3; k++) {
        t = ts[k] + 0
        V = 4.0/3.0*pi*(R*R*R - (R-t)^3); S = 4*pi*R*R; q = (V/S)/t
        printf "%-26s %10.6f %10.3f %8.4f %10.3f\n", sprintf("sphere R0.25 wall %.3f", t), V/S, t, q, 1/(q*q)
      }
      a=0.30; b=0.20; c=0.10
      split("0.003 0.010 0.030", tb, " ")
      for (k=1; k<=3; k++) {
        t = tb[k] + 0
        V = a*b*c - (a-2*t)*(b-2*t)*(c-2*t); S = 2*(a*b+b*c+c*a); q = (V/S)/t
        printf "%-26s %10.6f %10.3f %8.4f %10.3f\n", sprintf("box .30x.20x.10 wall %.3f", t), V/S, t, q, 1/(q*q)
      }
    }'
    shell                           V/S m     wall m  V/S / t  Fo factor
    sphere R0.25 wall 0.003      0.002964      0.003   0.9880      1.024
    sphere R0.25 wall 0.010      0.009605      0.010   0.9605      1.084
    sphere R0.25 wall 0.050      0.040667      0.050   0.8133      1.512
    box .30x.20x.10 wall 0.003   0.002903      0.003   0.9676      1.068
    box .30x.20x.10 wall 0.010   0.008945      0.010   0.8945      1.250
    box .30x.20x.10 wall 0.030   0.021164      0.030   0.7055      2.009

At three millimetres the volume over surface area is within 1.2 per cent of the
wall on the sphere and 3.2 per cent on the box, which is why the two definitions
can sit next to each other without a discontinuity where a body stops being
thin. At the thick end it reads 0.8133 and 0.7055 of the wall, and the last
column is what that does to a Fourier number, which is a factor of 1.512 and
2.009. Both are in the direction that makes the lumped model look more valid
than it is, and that is the direction a criterion must not err in, so the wall
thickness wins wherever it exists.

The criterion is two dimensionless numbers, computed per fragment per step, and
neither of them replaces the other.

The Fourier number over the heating time, `Fo = alpha t / L^2`, with the thermal
diffusivity `alpha = k / (rho c)` built from the thermal conductivity, the
density and the specific heat capacity that record 0016 already requires of
every material:

    git grep -n 'The required properties are density, specific heat capacity, thermal' origin/main -- docs/decisions/0016-material-record.md
    origin/main:docs/decisions/0016-material-record.md:102:The required properties are density, specific heat capacity, thermal

It asks whether conduction has had time to even the wall out over the heating so
far, and `t` is the time since the fragment entered rather than a fixed number,
so `Fo` grows along the trajectory and a fragment can begin outside the lumped
range and end inside it.

The through-thickness temperature ratio, `Theta = q L / (k T_s)`, the steady
temperature drop across the conduction length under the absorbed flux `q`,
divided by the surface temperature itself. It asks whether the difference the
lumped model discards is large compared with the temperature the radiation and
the melt criterion read.

Both are needed because they answer different questions and either can be small
while the other is not. A fragment late in a long flight has a large `Fo` and,
under a heavy flux through a poor conductor, still carries a steady gradient, so
`Fo` alone would call it isothermal. A thick fragment early in the flight has a
small `Fo` and, before the flux builds, a negligible gradient, so `Theta` alone
would call it isothermal too.

No threshold is fixed here, and the record does not carry one. What the tool
does instead:

Every run reports the maximum of `Fo` and of `Theta` reached over the flight,
per fragment, in the artefacts of record 0009 rather than in a log line.

A run classifies a fragment as inside or outside the lumped model's range only
against a threshold the scenario declares. Where the scenario declares none, the
run reports the two numbers and states that the lumped validity was not
classified. It does not substitute a conventional figure, because the widely
quoted one is a rule of thumb and issue #60 asks for the opposite of that, and
because a default threshold silently converts an unmeasured question into a
printed verdict.

That absence is a disclosure and it is not restated later as a bound. What would
replace it is a measurement rather than a citation, and it is the comparison
issue #60 already asks for: the same case run under both models, with the demise
altitude and the surviving mass recorded per fragment against the two numbers
above. The threshold is the value of `Fo` or `Theta` at which that comparison
first moves a demise altitude by more than the tolerance issue #64 holds. Until
that run exists, this record fixes the criterion and not where it bites.

## Reasons

The lumped model is the default because it is the limit the other one has to
reproduce, and a default that is a limit of the alternative is a default whose
error has a sign somebody can reason about. It is also what the comparable codes
offer as their zero-dimensional option, so a disagreement with a published
demise altitude computed that way is a disagreement about physics rather than
about which model was on.

Both models exist from the start rather than one now and one later. Record 0006
took the same order for the atmosphere and states the value of it directly:

    git grep -n 'Fixing the interface before the model' origin/main -- docs/decisions/0006-atmosphere.md
    origin/main:docs/decisions/0006-atmosphere.md:142:Fixing the interface before the model is the whole of this record's value. Every

Issue #42 puts the second half of the argument in a sentence this record borrows
rather than invents, that an interface with one implementation is not an
interface. The specific failure it prevents here is a core that grew a lumped
temperature of its own because nothing ever asked it not to.

Exporting only the surface temperature is the narrowest interface that serves
every consumer, and narrowing it is what makes the lumped and conduction models
substitutable. A mean temperature exported alongside would be read by whichever
consumer found it first, and for the lumped model the two are equal, so the
mistake would be invisible in every test written against the default.

The criterion is two numbers because one number cannot carry both questions, and
the alternative to two numbers is one number plus a caveat nobody reads. Both
are dimensionless, both are computed from properties record 0016 already
requires, and neither needs a material the library does not have.

Refusing to classify without a declared threshold follows the rule record 0010
sets for a missing input, and record 0016 takes the same line for a material
property. The tool would rather report two numbers and say it did not judge them
than print a verdict resting on a figure this project never measured.

Preferring the wall thickness to the volume over surface area where both exist
is chosen off the measurement above rather than off which is more usual. The
alternative errs by a factor of up to 2.009 in the Fourier number for the bodies
checked, and it errs towards calling a thick shell isothermal.

## Reasons against

The largest weakness is that this record fixes a criterion and no number, so
today it changes nothing an operator would see. A criterion reported without a
threshold is a pair of columns in an artefact, and a reader who wants to know
whether the lumped model was appropriate still cannot tell. The honest reading
is that the answer is not known yet rather than that it is being withheld, and
this record is what makes the missing measurement nameable.

The two definitions of the conduction length are a seam, and seams are where
this kind of model goes wrong. A solid body and a shell of the same outline get
their length from two different formulas, so a scenario that changes a component
from solid to a thick shell moves the Fourier number by more than the geometry
change alone, and nothing in the output separates the two effects.

`Theta` as written uses the steady temperature drop, which is an upper bound on
the transient one and therefore over-reports the gradient early in the flight,
exactly when `Fo` is also small. The two numbers are not independent, and a
reader taking them as independent evidence takes more than they give.

The interface exports one temperature, and a conduction model has more to say
than that. A reader wanting to know how hot the back face of a panel got cannot
ask, and for a shell whose inner surface faces a cavity record 0029 leaves
empty, that is the temperature somebody will eventually want. The narrowness
bought here is paid for there.

The flux `q` in `Theta` is the absorbed flux, and how the heat load is
distributed over a body is issue #59 rather than this record. So one of the two
criterion numbers depends on a distribution nobody has fixed, and its value will
move when #59 lands. That is a dependency written down rather than a defect, and
it does mean no number computed from `Theta` before #59 should be quoted
afterwards.

Nothing in this record is measured against a real material. The survey found no
openly redistributable source carrying the demise property set for any alloy
class this tool needs, so there is no row in the tree to put into `alpha`, and
every number above is geometry. A record about a thermal model that contains no
thermal property is a record whose arithmetic has not been exercised.

## What would change this

The two-model comparison issue #60 asks for, run on a case from the demise
regression set of issue #64, showing where the demise altitude first departs by
more than that suite's tolerance. That is the measurement that turns the two
criterion numbers into a threshold, and it is the one thing this record is
missing.

A material library under issue #49 whose conductivities put a common panel
material outside the lumped range over most of its flight. That would make the
default wrong for the ordinary case rather than for the thick one, and the
default would move.

A demise validation case under issue #78 whose disagreement is in the direction
a lumped body errs, which is a surviving fragment the tool demised or a
component the tool survived that was recovered.

A decision under issue #62 that reads a temperature other than the surface one,
which would widen the interface fixed here and reopen the substitutability
argument.

A heat load distribution under issue #59 that makes the absorbed flux strongly
non-uniform over a body, which would make a one-dimensional through-thickness
model the wrong second model rather than the right one.

## What depends on this

Issue #60, which planned this record and stays open on the clauses it does not
discharge: both models existing behind the interface, the conduction model
reproducing the lumped result in the limit where they should agree, and the
validity criterion computed and reported per fragment.

Record 0005, whose thermal state field is the state these two models own, and
whose sentence about conduction entering behind the same interface is the
interface this record names the directions of.

Issue #62, the demise criterion, which reads the surface temperature this record
exports and no other, and which owns what happens once melting is reached.

Issue #61, radiative cooling, which is the only heat loss the model has and
which radiates at the same surface temperature.

Issue #58 and issue #59, which supply the absorbed heat flux and its
distribution over the body, and therefore supply the `q` in the second criterion
number.

Record 0029, whose wall thickness is the conduction length wherever one is
declared, and whose empty cavity is the assumption the back face of a shell sits
against.

Record 0016, whose density, specific heat capacity and thermal conductivity are
what the thermal diffusivity is built from, and whose refusal on a property
evaluated outside its declared temperature range reaches every step of both
models.

Record 0009, whose artefacts carry the two criterion numbers per fragment, and
which is where the sentence saying the validity was not classified is written.

Issue #64, whose tolerance is what the threshold above is defined against, and
whose sensitivity table would gain the conduction length definition as one more
uncertain input.
