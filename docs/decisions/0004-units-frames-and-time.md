# 0004. Units, frames and time, and the rule that stops a mix-up

## Date

2026-08-08

## Status

accepted

## Question

This tool mixes altitudes in kilometres, densities in kilograms per cubic metre,
heat fluxes in watts per square metre, angles that are sometimes degrees, and
several coordinate frames. Which units does it work in, which frames does it
name, which time scale does it keep, and what stops two incompatible quantities
from being added together.

The last part is the reason the first three are worth writing down. Unit and
frame confusion is the most boring way for a physics tool to be wrong and among
the hardest to find afterwards, because the answer looks plausible.

## Options considered

On units.

Work in whatever unit each source uses and convert where needed. This is what
happens by default and it is why the failure exists. Every conversion is a place
somebody forgets one, and the forgotten ones are invisible because the result is
still a number of about the right size.

SI internally with conversion only at the boundary. Taken below. It costs an
explicit conversion layer at input and output, which is work that feels like
overhead until the first time it catches something.

On the type-level rule.

A naming convention, so that a variable called `alt_km_geodetic` is not added to
one called `alt_m_geopotential`. Free, and it is enforced by attention, which is
the thing that fails.

A runtime check that compares a unit tag carried alongside each value. Catches
the mistake, at the cost of carrying the tag and of only failing when the line
executes, which for a rarely reached branch means in production.

A quantity type that carries its unit and its frame in the type, so that the
wrong combination does not compile. Costs conversion functions, some noise at
the boundaries, and a learning curve for a contributor. Taken below.

A dimensional analysis library that generalises over all units. More general
than this project needs, and the generality shows up as compile errors that are
hard to read, which is a real cost when the reader is a physicist and no
type theorist.

On the internal time scale.

UTC internally. Familiar and the format an operator will supply. It is not a
uniform scale: it has leap seconds, so a difference between two UTC timestamps
is not a duration, and integrating over one is wrong by the leap seconds it
contains.

TAI internally. Continuous, no leap seconds, and a difference between two TAI
timestamps is a duration. Taken below.

GPS time or a mission elapsed time. Both continuous and both a fixed offset from
TAI, which makes them equivalent for the integration and worse for the
provenance, because neither is what the offsets are published against.

## Decision

All internal arithmetic is in SI base units. Metres, kilograms, seconds,
kelvins, radians. There are no exceptions inside the tool, and there is no unit
that appears only in some parts.

Conversion happens at the boundary and only there. An operator writes kilometres
and degrees if they want to, the scenario reader converts once, and the artefact
writer converts back. Nothing between those two points sees anything but SI.
Which unit a scenario field is written in is part of the schema record 0009
fixes, so the conversion is driven by a declared unit and by no
convention in a field name.

The frames are named individually and not as a family, and there are four
plus one.

The inertial frame is the Geocentric Celestial Reference Frame, which is the
frame the trajectory is integrated in. Naming a realisation, where "an inertial
frame" would have passed, is the point: two codes that both say inertial can
differ by the precession and nutation model between them, and that difference is
a real ground track error over the minutes a reentry lasts.

The Earth-fixed rotating frame is the International Terrestrial Reference Frame,
which is what a ground position is expressed in and what the Earth rotates into.
For the purposes of geodesy the WGS84 ellipsoid is the reference surface, with a
semi-major axis of 6378137 metres and a flattening of 1/298.257223563.

The geodetic triple is latitude, longitude and height above that ellipsoid, and
its ellipsoid is named every time it is used. Geodetic latitude is not
geocentric latitude and the difference reaches about 0.19 degrees at mid
latitudes, which is roughly 20 kilometres on the ground. A footprint is measured
in exactly those units, so this is not a rounding question.

The local horizon frame is north, east, down, with its origin at the geodetic
sub-point. This is the frame the wind is supplied in and the frame the impact
geometry is expressed in.

The one that is not a coordinate frame is the body frame of a fragment, which
exists so that a shape's reference area and its aerodynamic coefficients have a
direction to be defined against. Where the attitude treatment is a tumbling
average, this frame carries the average, per
record 0005.

Altitude is the one that has to be stated at every boundary, because the models
and the data do not agree on it.

Inside the tool, altitude means geodetic height above the WGS84 ellipsoid.

At the atmosphere interface, the altitude an atmosphere model wants is the
model's business and it is converted explicitly. The 1976 standard atmosphere is
tabulated against geopotential altitude below its upper region and against
geometric altitude above it, so the conversion is part of the adapter for that
model and not part of the trajectory core. Record 0006 makes the model's
validity range part of what it returns; the altitude convention is the same kind
of per-model property.

At the population grid, the altitude is not used at all and the horizontal
position is what matters, which makes the ellipsoid and the grid's own datum the
thing that has to agree. Record for that agreement sits with issue #73.

Where an operator supplies an altitude above mean sea level, the conversion to
ellipsoidal height needs a geoid model, and that model is named in the scenario
and never assumed. An unnamed geoid is a refusal under issue #21.

Time. The internal scale is TAI, because a difference between two TAI instants
is a duration and the integrator needs that to be true.

An operator supplies UTC, because that is what an epoch is published in, and the
conversion needs the leap second table. The Earth rotation angle that takes a
position from the inertial frame to the Earth-fixed one needs UT1, which is UTC
plus a published offset that changes continuously.

Neither the leap second table nor the UT1 offset is compiled into the tool. Both
are read from an Earth orientation parameter file supplied the same way the
space weather indices are, by a separate explicit command, never fetched on the
calculation path, and recorded in the manifest by identity and version. A run
whose epoch falls outside the range the file covers is refused, naming the file
and the epoch, and never extrapolated. The reason is arithmetic: the Earth turns
roughly 460 metres per second at the equator, so an offset wrong by a second is
a ground track wrong by a few hundred metres, and that is the same order as the
accuracy the whole tool is trying to claim.

The rule that makes this enforcement. A quantity carries its
unit in its type and a vector carries its frame in its type. Adding a geodetic
altitude to a geopotential one does not compile. Adding a position expressed in
the inertial frame to one expressed in the Earth-fixed frame does not compile.
Conversion is a named function and never an implicit coercion, so every place a
frame or a convention changes is a place a reader can find by searching for the
function's name.

The proof that this bites is a compile-fail test. A test that asserts a
particular wrong expression fails to compile, and which fails if that expression
ever starts compiling, is the only form of proof available for a rule whose whole
value is that the code does not build. It is named here as an obligation rather
than written, because no code exists yet. Issue #14 stays open until it exists,
and that is the one part of its definition of done this record cannot discharge
on its own.

## Reasons

SI internally is not a preference, it is the only arrangement in which the type
rule above is cheap. If two parts of the tool disagreed about which unit a
quantity is in, the type would have to carry a scale as well as a dimension, and
the conversion would be arithmetic.

Naming the frames individually, where a family name would have been shorter, is
what makes a disagreement with another code investigable. Two results that
differ, both computed in "an inertial frame", give a reader nowhere to start.
Two results that differ, one in GCRF and one in something else, give them the
first thing to check.

Making altitude a per-boundary statement is the honest arrangement, because the
disagreement is real. An atmosphere model and a population grid genuinely mean
different things by altitude and by latitude, and a tool that pretended
otherwise would be papering over the mismatch inside itself.

TAI internally is decided by the integrator. A scale in which a difference is
not a duration cannot be integrated over without special-casing the leap
seconds, and a special case in the time arithmetic is the kind of thing that
works until the year one is inserted.

The type-level rule is chosen over a runtime check because of when each one
fires. A runtime check on a rarely reached branch fires the first time that
branch is reached, which is in front of a user. A compile error fires before the
code exists.

## Reasons against

The type rule costs readability at the boundaries, and the boundaries are where
a physicist reading this code will start. A conversion that would be one
multiplication becomes a named function and a wrapped type, and the first
contributor from the field will experience that as ceremony and not as
safety.

It also has a hole, and the hole is the interesting part. A type distinguishes a
geodetic altitude from a geopotential one only where somebody remembered to make
them different types. A quantity that is modelled as a plain length because
nobody noticed it had a convention attached is exactly as unprotected as it would
be under a naming convention. The type rule catches the mistakes somebody
anticipated, and the ones nobody anticipated are the ones that reach production.

Reading Earth orientation parameters from a file, where a compiled-in leap
second table was open, means a fresh install cannot compute a trajectory until
the operator has fetched something. That is a real obstacle at the worst moment,
and the counter-argument, that a compiled table silently goes stale, is one an
impatient user will not find persuasive.

Fixing four frames before any of them is used risks fixing the wrong four. A
frame that turns out to be needed and is not here will be added, and the record
will read as though it were a complete list when it was a first list.

Against TAI specifically: every artefact an operator reads will be in UTC, so the
tool converts on the way in and on the way out, and there will be a moment when a
timestamp in a debugging output is in the wrong one and somebody loses an
afternoon.

## What would change this

A compile-fail test that turns out to be impractical to maintain, which would
move the proof from the compiler to a lint or a review checklist and would weaken
this record to a convention. That is checkable as soon as the workspace of issue
#26 exists.

An atmosphere or a data source whose altitude convention cannot be converted
from geodetic height without a model this tool does not have. That would add a
conversion with its own uncertainty and would not change the internal convention.

A precision requirement that makes the difference between two realisations of
the inertial frame matter, which would move the record from naming a frame to
naming a specific transformation model and its coefficients.

A decision under issue #25 that the ground track accuracy this record justifies
its time handling with is not achievable anyway, which would make the Earth
orientation parameter requirement heavier than the accuracy it buys.

## What depends on this

Issue #39, the time scales and frame transforms with published test vectors,
which is where this record becomes code and where the transformations are
checked against values somebody else published.

Record 0005, whose fragment carries a trajectory state in a stated frame at a
stated epoch and which explicitly defers that choice to this record.

Record 0006, whose atmosphere adapter owns the conversion from geodetic height
to whatever altitude a given model wants.

Record 0009, whose scenario schema carries the declared unit of every field and
whose artefacts carry positions in a named frame.

Record 0008, whose manifest records the identity and version of the Earth
orientation parameter file.

Issue #44, the entry state, which is where an operator's orbit becomes a state
in the frame named here.

Issue #70 and issue #73, the impact points and the population grid, which are
where the geodetic convention meets somebody else's datum.

Issue #21, the refusal rule, which owns the refusal on an unnamed geoid and on an
epoch outside the Earth orientation file.

Issue #35, the invariants gate, which is the only place a rule like the one in
this record could be refused by something other than the compiler.
