# 0014. How much of the gravity field the trajectory is integrated against

## Date

2026-08-10

## Status

accepted

## Question

The flight this tool simulates starts at the atmospheric interface and ends on
the ground. Over that flight, how much of the Earth's gravity field enters the
equations of motion, and what does stopping there cost.

Two things are folded into the same question because separating them is how the
mistake is made. The first is the field itself, meaning the degree and order of
the spherical harmonic expansion. The second is where the Earth's rotation
enters, which issue #41 asked to have made explicit on the grounds that the
rotating frame terms are larger than most of the harmonics and are the ones that
get dropped by accident. That reasoning is right and its conclusion moves, because
record 0004 has since fixed the frame the equations are integrated in, and in
that frame there are no rotating frame terms to drop. Where the rotation does
enter is answered below, so nobody has to rediscover it.

What is not decided here is the altitude a run may start at. Issue #41 asks this
record to name the altitude above which the truncation stops being defensible.
Naming it needs the measurement this record does not have, so what is written
below is a ceiling and a refusal.

## Options considered

The central term alone, an inverse square field from a point mass. One
expression, no coefficients, no data set. It drops a perturbation whose scale
against the central term is measured below as 1.5645e-03 at the interface, and
the entry phase is where that perturbation acts with almost no drag against it.

The central term plus the second degree zonal harmonic, degree 2 and order 0.
Taken below. One additional coefficient, no file to read, no interpolation, and
nothing new for the manifest to identify beyond a number and its derivation.

The central term plus a zonal series to a higher degree, J2 through J4 or J6.
Each additional zonal costs a coefficient and buys a term already smaller than
the one below it. The gain is not zero and it is not measured here, which is the
same objection this record raises against itself further down.

A field to a stated degree and order from a published model, with the
coefficients read from a file. This is the arrangement that scales, and its cost
is not arithmetic. A coefficient file is a data set: record 0008 requires the
manifest to record its identity and version, record 0009 requires it in the
scenario or in the packaging, issue #89 requires an attribution entry with its
terms, and issue #93 requires it to be present on a clean machine with no
network. A field bought that way is bought at the price of a provenance surface,
and the argument for paying it has to be an integrated error.

Normal gravity from the WGS 84 ellipsoid, the Somigliana formula. Weighed and
dropped in a sentence: it is a closed form for gravity on the surface of the
ellipsoid, which is not a field to integrate through, and the physical content it
carries above the central term is the same second degree zonal term the option
above names directly.

## Decision

The field is the central term plus the second degree zonal harmonic. Degree 2,
order 0, and nothing else.

The coefficient is not written as a literal, here or in the source. It is
derived from the WGS 84 dynamic second degree zonal coefficient by that
standard's own relation, and both the input and the derivation belong to the
constants module that issue #35's invariant guards, so that the value cannot be
changed in one place and stay stale in another.

    awk 'BEGIN { C20 = -4.84165143790815e-04; printf "%.10e\n", -C20 * sqrt(5) }'
    1.0826261739e-03

The input is Table 3.2 of NGA.STND.0036_1.0.0_WGS84, "Department of Defense
World Geodetic System 1984", version 1.0.0, 2014-07-08, which gives the EGM2008
dynamic value as -4.84165143790815e-04, and the relation is equation (3-1) of
the same document. The dynamic coefficient is the one taken, and not the
geometric one implied by the ellipsoid, because what is being integrated is the
Earth's field and not the reference surface. The semi-major axis, the flattening
factor, the geocentric gravitational constant and the nominal mean angular
velocity are Table 3.1 of that document, and record 0004 already fixes the first
two as the reference surface for geodetic height.

The scale of the term this buys, and the scale of the largest thing degree 2
still leaves out, at 120 kilometres above the semi-major axis:

    awk 'BEGIN {
      a = 6378137.0; GM = 3.986004418e14; w = 7.292115e-5
      C20 = -4.84165143790815e-04; C22 = 2.43938357328313e-06
      J2 = -C20 * sqrt(5); r = a + 120000; g = GM / (r * r)
      v = sqrt(GM / r); u = w * r
      printf "J2 scale against central term: %.4e\n", 1.5 * J2 * (a / r)^2
      printf "C22 against C20:               %.4e\n", C22 / (-C20)
      printf "central acceleration:          %.4f m/s2\n", g
      printf "circular speed at r:           %.1f m/s\n", v
      printf "co-rotation speed at r:        %.1f m/s\n", u
      printf "drag factor, air overtaking:   %.4f\n", ((v - u) / v)^2
      printf "drag factor, air opposing:     %.4f\n", ((v + u) / v)^2
      printf "those departures over J2 scale: %.1f and %.1f\n", \
        (1 - ((v - u) / v)^2) / (1.5 * J2 * (a / r)^2), \
        (((v + u) / v)^2 - 1) / (1.5 * J2 * (a / r)^2)
    }'
    J2 scale against central term: 1.5645e-03
    C22 against C20:               5.0383e-03
    central acceleration:          9.4397 m/s2
    circular speed at r:           7832.0 m/s
    co-rotation speed at r:        473.9 m/s
    drag factor, air overtaking:   0.8827
    drag factor, air opposing:     1.1247
    those departures over J2 scale: 75.0 and 79.7

The first line is the factor multiplying the central acceleration in the leading
J2 term, with the latitude factor of order one left out, so it is a scale and not
a bound. The second uses the degree 2 sectorial coefficient from the same Table
3.2 as the largest non-zonal term at the degree being kept, and it says that
order 0 leaves out something two hundred times smaller than what order 0 keeps.

The field sits behind an interface with one implementation, in the arrangement
record 0006 uses for the atmosphere. A higher degree field is then a second
implementation and a scenario field, not a change to the integrator.

The equations of motion carry no centrifugal term and no Coriolis term. Record
0004 integrates the trajectory in the Geocentric Celestial Reference Frame, and
those terms exist only in equations written in the rotating frame. This is stated
so that adding one reads as the error it would be, and never as a correction
somebody remembered.

The Earth's rotation enters in two other places and neither is gravity. The
transformation to the Earth-fixed frame for the ground track is record 0004, and
it is the reason that record reads Earth orientation parameters from a file. The
atmosphere co-rotates with the Earth, so the velocity the drag is computed
against is the velocity relative to that co-rotating air, and dropping the
difference is the mistake issue #41 was pointing at. Its size is on the last
three lines printed above. For an equatorial track at that radius, computing the
drag against an inertial velocity, where a relative one belongs, changes it by a
factor of 0.8827 where the air overtakes the fragment and 1.1247 where it
opposes it, and the departures of those factors from one are 75.0 and 79.7 times
the J2 scale on the first line. That term belongs to issues #43 and #52. It is
named here because it is the larger one, it is not a gravity term, and this
record is where a reader would otherwise go looking for it.

Third body attraction from the Sun and the Moon, and solar radiation pressure,
are excluded. The exclusion is stated and it is not quantified. No magnitude for
either was computed for this record, and issue #45 is where an unquantified
exclusion becomes a measured one.

The truncation error is not measured. What is fixed here instead is the shape of
the measurement, so that it is produced once and the argument after it exists
has something fixed to meet. It is a comparison of two runs of the same case
that differ only in the field: the field decided here against a field of a
stated higher degree and order, with the entry state, the seed, the atmosphere,
the space weather input and the fragment set held identical. The quantities
reported are the along-track and cross-track separation of the impact point in
metres and the difference in demise altitude in metres, per fragment and never
aggregated, on a case from the regression set of issue #45. Until that
comparison exists, a run states that the gravity truncation error is unmeasured.
That sentence is a disclosure. It is not restated later as a bound.

A scenario whose start altitude is above the atmospheric interface is a
pre-flight refusal under record 0010, naming the start altitude, the interface
and issue #44. The refusal exists because the argument above is an argument about
a flight of a few minutes, and it is silently stretched by a run that begins well
above the interface and propagates down. Issue #44 is where the highest state an
operator may supply is decided, and this record does not decide it.

## Reasons

J2 is kept because it is not small where it acts. Its scale
against the central term at the interface is 1.5645e-03, and the entry phase is
the part of the flight where the drag that dominates everything later is still
negligible, so the perturbation acts almost unopposed for the part of the
trajectory that decides where the rest of it goes.

Nothing beyond J2 is kept because the next term costs a category of work, where
J2 cost a line. The measured ratio above puts the largest degree 2 non-zonal
coefficient two hundred times below the zonal one, and a field bought from a
published model arrives as a data set with an identity, a version, a licence, an
attribution entry and a file that has to exist on a machine with no network.
Paying that for an unmeasured gain is the shape of decision this project exists
to avoid, and it is cheap to reverse once the measurement exists.

Deriving the coefficient, where writing it down would be shorter, is the same
argument as record 0012's single module for elementary functions. A physical
constant that appears as a literal is a value that can be changed in one place
and stay stale in another, which is one of the invariants issue #35 already
carries.

Stating what is absent from the equations of motion alongside what is present
is what makes this record readable by somebody checking an
implementation against it. A record that lists the terms it includes cannot be
used to find a term that should not be there.

## Reasons against

The strongest argument against this record is that it decides an integrated
question with an instantaneous number. Every figure above is a ratio of
accelerations at one radius. What matters is where the fragment lands, which is
that ratio integrated over the flight, and a perturbation three orders below the
central term can still move a footprint if it acts long enough in one direction.
Until issue #45 runs the comparison this record specifies, the decision is an
argument and the tolerance it implicitly claims is unstated.

The scale printed above is not a bound. It drops the latitude factor and it is
computed at the equatorial radius, so a reader who takes 1.5645e-03 as a maximum
is taking more than the command produced.

J2 alone is an ellipsoid, and the interesting part of the real field is exactly
its departure from one. A footprint over a region with a large geoid undulation
is where a reader most wants the answer, and it is where this truncation gives
them the least. The record has no measurement to say how much less.

Excluding third body attraction and radiation pressure without a magnitude is
weaker than the rest of this record, and it is weaker in the direction that
matters, because an exclusion nobody quantified is indistinguishable from one
nobody thought about.

Refusing a start above the interface is a real obstacle for the operator most
likely to want this tool, who arrives with an orbit. It is
chosen over a warning because a warning on the calculation path is read once and
then not read, but it does mean the first thing some operators meet is a refusal
where they expected an answer.

Fixing the coefficient to the EGM2008 dynamic value ties one number in this tool
to one gravitational model, in a record that otherwise argues the model does not
matter at this degree. If it does not matter, the choice of source for it is
decoration; if it does, the record is wrong about something larger.

## What would change this

The comparison specified above, run on a case from issue #45, showing an impact
point separation larger than the tolerance issue #77 fixes for an impact point.
That is the measurement this record is missing and it is the one that would move
it, in either direction.

A decision under issue #44 admitting a start well above the interface. The
argument here is about a flight of a few minutes and it does not survive an arc
of hours without being remeasured.

A validation case under issue #78 or #79 whose disagreement lies along the track
and not across it, which is the direction a field truncation would move an
impact point.

A population grid decided under issue #73 whose resolution makes a sub-kilometre
track error the difference between two cells with different densities, which
would raise the accuracy this record has to hold and leave the physics in it
alone.

## What depends on this

Issue #41, which planned this record and stays open on the clauses it does not
discharge: the implementation, the measured truncation error with the command
that produced it, and the behaviour above the altitude where the choice stops
holding.

Issue #40, the state vector and the integrator, which is where this field becomes
an acceleration and where the absence of a rotating frame term is checkable.

Issue #45, the propagation regression set, which owns the comparison this record
specifies and the exclusions it leaves unquantified.

Record 0004, which fixes the frame this record integrates in, the reference
surface its constants come from, and the Earth orientation input the ground track
needs.

Record 0010, whose list of refusable inputs gains the start altitude above the
interface. That record states that an absent input on neither of its lists is a
defect report against it, so the list growing here is the mechanism working
and no exception to it.

Record 0008, whose manifest carries the degree and order used and the derived
coefficient, so that a run computed against this field is distinguishable from
one computed against a larger one.

Issues #43 and #52, which own the relative velocity the drag is computed against
and therefore own the co-rotating atmosphere term this record measures and hands
over.

Issue #35, whose constants invariant is what keeps the coefficient out of the
call sites, and issue #26, whose workspace layout decides where a force model
lives. The Scope line of issue #41 places this under a root beginning `core`,
and record 0013 places the models a step above the core, so the placement is a
question for #26 and this record does not settle it.
