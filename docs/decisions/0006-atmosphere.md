# 0006. The atmosphere interface, the default model, and where space weather comes from

## Date

2026-08-08

## Status

accepted

## Question

Density is the single most influential input to a reentry trajectory, and the
models available disagree with each other. That disagreement is a stated
uncertainty that has to reach the output, not noise to be averaged away.

Three things have to be decided together. What the rest of the tool asks an
atmosphere for. Which model answers by default, and under what licence it can
be shipped. Where the solar and geomagnetic indices such a model needs come
from, and what happens when they are absent.

## Options considered

On the order of the decision.

Choose the model first and let the interface follow from what that model
happens to return. This is what a tool does when nobody decides, and it costs
the property that matters most here: if the interface is the shape of one
model's output, running a second model over the same case is a port rather than
a configuration change, and the comparison that would give the answer an honest
error bar never gets made.

Fix the interface first and the model second. Taken below.

On the default model.

NRLMSIS 2.x. The current model of the family every reentry tool uses, and the
one an analyst would name without thinking. It is ruled out as a shipped
default by its licence, which is an academic research agreement rather than an
open one: the grant is for academic, non-commercial purposes, and it withholds
the right to modify the software, to disseminate it for commercial purposes, or
to copy it beyond personal, archival and non-commercial academic use without
written consent [1]. A tool that ships it makes every operator's use a question
for somebody's legal department, and this project's stated requirement is a
default whose licence permits redistribution.

NRLMSISE-00 through the widely used C port. Older, well characterised, and the
version the reference reentry codes moved to [2, section 2.5]. The package
carries a warranty disclaimer and a request that patches be sent back to its
maintainer, and no grant of rights was found in the documentation read [3]. An
absent licence is not a permissive one, so it cannot be shipped on the strength
of what has been checked.

U.S. Standard Atmosphere 1976. A work of the United States government, public
domain, distribution unrestricted, with tables to 1000 km [4]. It is the model
the reference code uses for natural decay trajectories [2, section 2.5]. Its
cost is severe and it is the reason this option is uncomfortable: it is a static
atmosphere with no solar or geomagnetic dependence at all, so it cannot
represent the density variation that decides a decay.

A model written here from published coefficients. Removes the licence question
entirely and replaces it with a validation obligation this project cannot meet
early, since validating an atmosphere model is a larger piece of work than the
rest of this milestone.

On the behaviour when indices are missing.

Fall back to a static atmosphere and label the output. Convenient, and the label
is the only thing standing between an operator and a number computed against an
atmosphere they did not choose.

Refuse and name what is missing. Costs the operator a step and cannot be
misread.

## Decision

The interface is fixed first and the model is a choice behind it.

An atmosphere is anything that answers, for a position and a time, with the
density, the temperature, the pressure and the number densities of the neutral
species the free molecular heating needs. It also answers with its own identity,
meaning a name and a version, so that record 0008's manifest can record which
atmosphere produced a result, and with the altitude range over which it claims
validity.

The tool can run the same case against two atmospheres and report what changed.
This is a requirement on the interface rather than a feature, because the
difference between two models over one case is one of the few honest error bars
available on the answer, and record 0007 names the atmosphere as the uncertainty
it is most waiting on.

The default model shipped inside the artefact is the U.S. Standard Atmosphere
1976, on the grounds of its licence and nothing else. It is a work of the United
States government, in the public domain, with distribution unrestricted, and it
covers 0 to 1000 km [4].

That is a weak default and the record says so rather than presenting it as a
choice on the physics. A static atmosphere has no solar or geomagnetic
dependence, so two reentries at opposite ends of a solar cycle are the same
reentry to it. Every artefact produced with it is labelled with the model
identity, and the validation standing of issue #81 states plainly that a result
computed against a static atmosphere is not a result to plan against.

A thermospheric model is selected by the operator and supplied by the operator,
and the interface above is what it is supplied through. Two are named as the
ones worth supporting first, NRLMSISE-00 and NRLMSIS 2.x, and neither ships. The
reason is recorded above and it is a licence reason rather than a technical one.
If a redistributable model of that class is confirmed, it becomes the default
and this record is superseded rather than quietly amended.

The space weather input is F10.7 for the day, the 81-day average of F10.7, and
the geomagnetic Ap index [3]. The source is the CelesTrak space weather file,
which carries all three in one record along with Kp, and which states its own
upstream sources as GFZ Potsdam for the geomagnetic data, the Canadian Space
Weather Forecast Centre for F10.7, SIDC for sunspot number, and NOAA SWPC and
NASA for the short and long term predictions [5]. No terms of use were stated on
the page read, and that absence is recorded rather than read as permission.

Fetching that file is a separate explicit command. It is never done during a
calculation, never on a first run, and never as a side effect of a command whose
name is about something else. The test that exercises it belongs to the
separated network suite of issue #30. This is the same position record 0009 takes
on implicit migration and the same one issue #24 takes on network access: the
calculation path makes no request.

When a model that needs indices is selected and the indices are absent, the run
is refused. The refusal names the model, the index that is missing, the epoch it
was needed for, and the command that fetches it. There is no fallback, no
substitution and no flag that turns the refusal into a warning. The reasoning is
the one issue #21 holds: a substituted default that is invisible in the output is
the mechanism by which a confident wrong number is produced, and this is the
place it would most easily happen, because an operator who has never fetched an
index file has no reason to know one exists.

Outside a model's stated validity range the answer is a refusal rather than an
extrapolation. The range is part of what the interface returns, so this is
checked against the model's own claim rather than against a constant written
here.

## Reasons

Fixing the interface before the model is the whole of this record's value. Every
atmosphere in this class returns roughly the same quantities, so the interface is
not hard to design; what is hard is retrofitting one after a model's output shape
has spread through the trajectory core. Doing it first is cheap now and expensive
later, and the comparison it makes possible is a real error bar rather than a
feature nobody uses.

The licence finding decided the default and it was not the expected answer. The
model an analyst would name is not one this project can ship, and the honest
consequence is a default that is weak on the physics rather than a default whose
licence is glossed over. Labelling every output produced with it, and saying in
the validation standing what it means, is what stops that weakness from being
invisible.

Refusing rather than falling back follows from where this project thinks wrong
numbers come from. The failure is not a tool that stops, it is a tool that
answers using something the operator did not choose. Indices are the clearest
case of that in the whole design.

Keeping the fetch out of the calculation path is a security and a provenance
decision at once. An operator inside an organisation that does not permit
outbound requests can still run every calculation, and a run that made a request
would have a result depending on what a server returned that day, which is not
reproducible under record 0008 whatever else is pinned.

## Reasons against

The default is bad physics and no amount of labelling makes it good. A tool
whose out-of-the-box atmosphere cannot represent solar activity will produce
first results that are wrong in the direction of whatever the standard
atmosphere assumes, and first results are the ones people remember. There is a
fair argument that shipping no default at all and refusing until the operator
supplies a model would be more honest than shipping a weak one.

The licence position on NRLMSISE-00 may be too cautious. The absence of a grant
in the documentation read is not the same as a prohibition, and a package that
has been redistributed widely for two decades may well carry terms that were not
found here. That is a question for somebody who can read the actual licence file
rather than a decision, and treating it as settled would be the same mistake in
the other direction.

Requiring an operator to fetch indices before a realistic run adds a step at
exactly the moment they are least patient, and the refusal will be experienced
as the tool being difficult. Some of them will reach for the static atmosphere to
make it stop, which is the failure this record is trying to prevent, arrived at
by a different route.

Naming a single upstream for space weather makes that upstream a dependency of
every realistic run. The file format is stable and long-lived, and it is still
one organisation's file, and no terms of use were found for it.

## What would change this

A redistributable thermospheric model, or a reading of the NRLMSISE-00 package's
actual licence text showing a grant that permits shipping it. That is the single
change that would most improve this record, it is cheap to check, and it was not
checked here.

A measurement of how far two atmosphere models actually diverge over a
representative reentry. Nothing is measured, and the claim that they disagree is
carried from the issue rather than from a run. As soon as the interface exists
the measurement is one command, and record 0007 is waiting on the number to turn
the density uncertainty from an assumption into something with a source.

A validity range that turns out to be stated differently by a model than by its
documentation, which would move the refusal boundary.

An operator environment where the separate fetch is impossible, which would make
the index file an input the operator supplies by hand rather than one the tool
retrieves. The interface already permits that and the record would not need to
change, but the documentation would.

## What depends on this

Record 0007, which names the atmosphere density as the uncertainty it is most
waiting on and which inherits the model comparison described here as the way to
give it a source.

Record 0008, whose manifest records the identity and version of every data set
read, including the atmosphere model and the index file.

Record 0009, which carries the atmosphere and space weather selection in the
scenario and the model identity in the artefacts.

Issue #21, the refusal rule, which owns the position that a missing index is a
refusal rather than a substitution.

Issue #24, which owns the statement that the calculation path makes no network
request, and issue #35, the invariants gate, which is where that becomes
enforcement rather than prose.

Issue #30, the separated suites, which is where the fetch is tested.

Issue #42, the atmosphere behind the interface, which is where this becomes code
and where the space weather input becomes a file.

Issue #43, winds, which sits behind the same interface.

Issue #81, the validation standing, which has to say what a result computed
against the shipped default means.

## Sources

1. Licence terms distributed with NRLMSIS 2.1, as summarised from the release
   the unofficial mirror at <https://github.com/jacobwilliams/NRLMSIS2.1>
   carries. The grant is academic and non-commercial and withholds modification
   and dissemination without written consent from Naval Research Laboratory IP
   Counsel. The licence file itself was not opened; this is read from the
   release description and is marked here as such.
2. Ostrom, C. et al. "Operational and Technical Updates to the Object Reentry
   Survival Analysis Tool." First International Orbital Debris Conference, 2019.
   NASA NTRS 20190033904. Recorded in `docs/survey/existing-codes.md`.
3. NRLMSISE-00 C package documentation, which names Mike Picone, Alan Hedin and
   Doug Drob as the model's authors and Dominik Brodowski as the author of the C
   port, states the five inputs including F10.7 daily, the F10.7 81-day average
   and Ap, and carries a warranty disclaimer with no grant of rights.
   <https://raw.githubusercontent.com/magnific0/nrlmsise-00/master/DOCUMENTATION>
4. "U.S. Standard Atmosphere, 1976." NOAA, NASA and USAF. NOAA-S/T-76-1562,
   NASA-TM-X-74335. Tables to 1000 km, public domain as a work of the United
   States government. <https://ntrs.nasa.gov/citations/19770009539>
5. CelesTrak space weather data format documentation, which lists the fields
   including the eight three-hourly Kp and Ap values, F10.7 observed and
   adjusted, and the 81-day centred and trailing averages, and which names its
   upstream sources. <https://celestrak.org/SpaceData/SpaceWx-format.php>
