# What the standards require of a reentry risk number

The output of this tool exists because standards ask for it. What they actually
require, in their own words and with their clause numbers, decides what the tool
has to emit for the number to be usable at all.

Three documents were read in full enough to quote clause numbers: the NASA
technical standard, the ESA requirements standard, and the United States federal
regulation the FAA licenses launch and reentry under. A fourth, the ISO standard
several of the others derive from, is behind a paywall and is recorded as such
rather than paraphrased.

The most useful thing in this document is not the thresholds. It is the section
on where two documents use the same word for different quantities, because that
is the failure this survey exists to prevent, and it turned out to be real.

## NASA-STD-8719.14C

Process for Limiting Orbital Debris, revision C, 5 November 2021, 77 pages [1].

The threshold is Requirement 4.7-1, and it is four separate requirements rather
than one. Its opening sentence fixes the energy filter for all of them: "The
potential for human casualty is assumed for any object with an impacting kinetic
energy in excess of 15 joules" [1, 4.7.2.2].

Requirement 4.7-1a, uncontrolled reentry: "the risk of human casualty from
surviving debris shall be less than 0.0001 (1:10,000)".

Requirement 4.7-1b, controlled reentry, is a geometric requirement rather than a
probability: "the selected trajectory shall ensure that no surviving debris
impact with a kinetic energy greater than 15 joules is closer than 370 km from
foreign landmasses, or is within 50 km from the continental U.S., territories of
the U.S., and the permanent ice pack of Antarctica".

Requirement 4.7-1c, controlled reentry again: "the product of the probability of
failure to execute the reentry burn and the risk of human casualty assuming
uncontrolled reentry shall be less than 0.0001".

Requirement 4.7-1d, long-term reentry: "For long-term reentry of space structures
in MEO, Tundra orbits, highly inclined GEO, and other orbits: Surviving debris
shall have less than 7 m2 total debris casualty area or 0.0001 (1 in 10,000) risk
of human casualty". The trailing "and other orbits" is part of the clause and is
what stops it being read as a list of three orbit classes.

The definition of casualty is by energy rather than by injury model, and the 15
joule figure carries its own source: it comes from the U.S. Range Commanders
Council Standard 321-20 Supplement, section 6.2, May 2020, and the standard
quotes the reasoning, that "Injury is demonstrated to be highly improbable from
any contact below this energy with any part of a human" [1, 4.7.1.2].

The population basis is stated and it is the part most likely to surprise
somebody. The standard records that "approximately 80% of the world's population
is unprotected or in lightly-sheltered structures providing limited protection
against falling debris", and then declines to use that: "For the purpose of this
standard, any debris with an impacting kinetic energy greater than 15 joules will
be considered potentially hazardous for all of the world's population" [1,
4.7.3.4]. So no sheltering credit is taken, deliberately, and the 80 per cent
figure is the reason given rather than a factor applied.

The debris casualty area is defined by Equation 4.7-1 as a sum over surviving
objects of a term combining each object's average cross-sectional area with a
0.6 metre allowance for the person. The filter is part of the definition of the
total rather than a separate step: "The total debris casualty area for a reentry
event is the sum of the individual debris casualty areas for all debris pieces of
greater than 15-joule terminal energy that survive atmospheric reentry" [1,
4.7.4.1]. The standard says the 0.6 metre value
"creates a casualty area curve very closely matched to a more complicated and
more accurate formula derived by Opiela and Matney" [1, 4.7.4.2]. The equation
itself is reproduced in the source as a formula whose symbols did not survive
text extraction cleanly; the 0.6 metre value, the sum over surviving objects and
the Opiela and Matney provenance are stated in prose in the same paragraph and
are what is relied on here.

Applicability has a floor that is worth noting against the tool's own entry
interface: "Requirement area 4.7 applies to all spacecraft and launch vehicles
returning to the surface of the Earth from an altitude of greater than 130 km"
[1, 4.7.1.4]. The reference implementation of the same agency defines its entry
interface at 122 km [2, section 2.2], so the applicability altitude and the
analysis altitude are different numbers and neither is the other.

The threshold is per event and the standard says so historically: "In 1995 NASA
established a policy of limiting the risk of world-wide human casualty from a
single, uncontrolled reentering space structure to 1 in 10,000", and records that
ESA and the IADC adopted the same threshold [1, 4.7.3.1].

## ESA, ESSB-ST-U-007

ESA Space Debris Mitigation Requirements, Issue 1, 30 October 2023, 83 pages [3].

Section 5.5, Re-entry, carries the threshold normatively. Clause 5.5 b: "The
expected number of casualties per re-entry of a spacecraft or launch vehicle
orbital stage, including elements thereof, shall be less than 10-4 through
adopting design and operations practices in the following order of preference: 1)
Design for demise 2) Controlled re-entry 3) Approval of any other design or
operations practice by the approving agent". That is the same number as NASA's
and a different quantity: an expected number of casualties rather than a risk of
human casualty.

The scope of the sum is fixed in a note and it is wider than one object. The
calculation "takes into account all objects released in orbit from the spacecraft
or launch vehicle orbital stage, including elements thereof", with a dispenser
detached by a launch vehicle stage given as the example, "even when not
re-entering at the same moment in time" [3, 5.5 b NOTE 4].

Clause 5.5 c requires that the expected number of casualties "shall be assessed
probabilistically", which no other document read here asks for.

The same 10-4 appears a second time as a condition rather than a requirement, and
the two should not be confused. Clause 5.5 b is what a reentry has to satisfy;
clause 5.4.1.3 a. 2) uses the same number to decide whether a LEO spacecraft has
to be prepared for external removal, exempting one whose "on-ground re-entry
casualty risk is lower than 10-4 in case of uncontrolled re-entry" [3, 5.4.1.3
a. 2)].

The second threshold is the one that has no NASA counterpart and it is two orders
of magnitude tighter. "Spacecraft part of a large constellation in Earth orbit
re-entering shall either: 1) Have an expected number of casualties per re-entry
below 10-6 2) Perform a controlled re-entry" [3, 5.5 d]. The rationale section
names why: the standard sets out to mitigate "the constellation aggregate
re-entry casualty risk in view of expected large number of re-entry events,
especially from large LEO constellations" [3, 4.5].

The definition of casualty risk is by outcome rather than by energy: "risk that a
person is killed or seriously injured", with a note that the risk "is determined
through the probability to cause death or serious injury, as specified in
ESSB-ST-U-004" [3, 3.2.4]. That referenced document was not read, and clause
5.5 a applies its requirements wholesale to reentry, so what it adds on top of
section 5.5 is not established here.

Demise is defined, and the definition is a casualty definition rather than a
thermal one: "result of an ablation processes acting on elements, equipment,
parts or components of a space object during an atmospheric re-entry event to the
extent that the resulting fragments no longer pose a casualty risk", with a note
naming tanks, reaction wheels and magnetorquers as the common equipment that does
pose one [3, 3.2.11].

The verification expectation is stated in a way no other document read here
matches. A 95 per cent confidence level "is often adopted" for demonstrating the
demise of a component, and a set of minimum uncertainties for reentry casualty
risk analysis is specified in ESSB-HB-U-002 Issue 2 [3, 5.5 c NOTE 2]. Where a
probabilistic assessment produces a multi-modal distribution with a mode above the
threshold, a bespoke assessment is applied, and where it does not, a median
approach is used to validate the threshold at system level [3, 5.5 c NOTE 1].
This is the only place read that asks an assessment to say how uncertain it is.

## The FAA, 14 CFR 450.101

The reentry criteria are in paragraph (b), Reentry risk criteria, and they are
not one number [4].

Collective risk, 450.101(b)(1): "The collective risk, measured as expected number
of casualties (EC), consists of risk posed by impacting inert and explosive
debris, toxic release, and far field blast overpressure". It is capped twice, over
two different sets of people rather than over one set and a superset of it. "The
risk to all members of the public, excluding persons in aircraft and neighboring
operations personnel, must not exceed an expected number of 1 x 10-4 casualties"
[4, 450.101(b)(1)(i)], and "The risk to all neighboring operations personnel must
not exceed an expected number of 2 x 10-4 casualties" [4, 450.101(b)(1)(ii)].
Persons in aircraft are excluded from the first cap and carried by (b)(3) below.

Individual risk, 450.101(b)(2): "The risk to any individual member of the public,
excluding neighboring operations personnel, must not exceed a probability of
casualty of 1 x 10-6 per reentry", and 1 x 10-5 per reentry for any individual
neighbouring operations personnel.

Aircraft, 450.101(b)(3): the operator "must establish any aircraft hazard areas
necessary to ensure the probability of impact with debris capable of causing a
casualty for aircraft does not exceed 1 x 10-6".

Critical assets, 450.101(b)(4), is the criterion that is not about people at all
and it belongs in the list because it is inside the same paragraph. The risk is
"measured as the probability of loss of functionality" and is capped at 1 x 10-3
for each critical asset and 1 x 10-4 for each critical payload. The assets are
named by federal agencies rather than by the operator, and the Administrator may
impose a more stringent probability.

The definitions are in 14 CFR 401.7 and they are worth quoting because they are
the sharpest of the three sets. "Casualty means serious injury or death."
"Casualty area means the area surrounding each potential debris or vehicle impact
point where serious injuries, or worse, can occur." And, separately, "Effective
casualty area means the aggregate casualty area of each piece of debris created
by a vehicle failure at a particular point on its trajectory", described in the
same definition as "a modeling construct in which the area within which 100
percent of the population are assumed to be a casualty, and outside of which 100
percent of the population are assumed not to be a casualty" [5]. The same section
defines the collective quantity: "Expected casualty means the mean number of
casualties predicted to occur per flight operation if the operation were repeated
many times" [5].

No kinetic energy threshold appears in the text of 450.101 read here. The phrase
used is "debris capable of causing a casualty", and where that is quantified is
not established from these two sections.

## ISO 24113

Not read. The standard is sold rather than published, and the request for its
catalogue page from this machine was refused:

    curl -sIL https://www.iso.org/standard/83494.html
    HTTP/1.1 403 Forbidden

What can be said without it is that the 2023 edition is normatively referenced by
the ESA standard, which quotes its definitions of approving agent and break-up
verbatim and attributes them "[ISO 24113:2023]" [3, 3.2.2 and 3.2.3]. So some of
ISO 24113's text is reachable through a free document, which is a useful fact for
anybody building on it, and it is not a substitute for the standard.

## Where two documents mean different things by the same word

This is the section the issue asked for and it is not empty.

The number 1 in 10,000 appears in all three documents read and does not denote
the same quantity in any two of them. NASA's is the risk of human casualty from
surviving debris, meaning falling objects and nothing else [1, 4.7.2.2]. The
FAA's collective risk is expected casualties from "impacting inert and explosive
debris, toxic release, and far field blast overpressure" [4, 450.101(b)(1)],
which is a strictly larger hazard set, so an assessment that satisfies NASA's
threshold has not thereby satisfied the FAA's. ESA's is an expected number of
casualties [3, 5.5 b], which is a mean rather than a probability, and it is summed
over every object released by the vehicle including ones reentering at another
time [3, 5.5 b NOTE 4]. So the three carry the same digits over a probability, an
expectation over a wider hazard set, and an expectation over a wider object set.

The population the number is taken over is not the same population either. NASA's
is all of the world's population within the orbit extreme latitudes, every person
treated as exposed [1, 4.7.3.4]. The FAA's is "the public", which its definitions
exclude launch and reentry participants from and which splits again into
neighbouring operations personnel with their own cap [5]. An expected casualty
number computed over everybody underneath is not the FAA's collective risk.

The FAA requires something neither of the others does: an individual probability
of casualty, capped at 1 x 10-6 per reentry for a member of the public
[4, 450.101(b)(2)(i)]. A collective expectation can meet 1 x 10-4 while a single
person under the footprint exceeds 1 x 10-6, so the two are independent
constraints and a tool that emits only the collective number cannot answer the FAA
at all.

The threshold is not always per event. NASA's is explicitly per single
uncontrolled reentering structure [1, 4.7.3.1]. ESA's constellation clause is
also written per reentry, at 10-6, but its stated purpose is the aggregate over
many reentries [3, 4.5], so the same form of words is doing a different job.

Casualty area is defined twice and computed once. The FAA's definition is
physical, the area where serious injuries or worse can occur, with the effective
casualty area named as a modelling construct [5]. NASA's is a formula with a 0.6
metre human allowance [1, 4.7.4.2]. They are compatible in intent and only one of
them tells an implementer what to compute.

The total debris casualty area limit is not a general limit on any reentry. The
current standard states 7 m2, inside Requirement 4.7-1d, which is the long-term
reentry case: MEO, Tundra orbits, highly inclined GEO "and other orbits" [1,
4.7.2.2]. It is an alternative there rather than a condition, satisfied by either
the 7 m2 or the 1 in 10,000. None of 4.7-1a, 4.7-1b or 4.7-1c names a casualty
area limit at all, so quoting 7 m2 as the limit an uncontrolled reentry has to
meet is quoting the wrong requirement. The 8 m2 figure that appears in the
literature is attributed to the older NASA safety standard NSS 1740.14 by [6,
section 2.3]. NSS 1740.14 was not read here, so what this document asserts is
that the figure is not in the current standard, not what the superseded one says.

The energy threshold is NASA's and the Range Commanders Council's, not
everybody's. 15 joules is stated by NASA with its source [1, 4.7.1.2]. It does
not appear in the FAA sections read. Attributing it to a document that does not
carry it is the exact error this section exists to catch.

Sheltering is where the documents differ most quietly. NASA records that about 80
per cent of the world's population is unprotected or lightly sheltered and then
applies no sheltering credit at all, treating every person as exposed [1,
4.7.3.4]. ESA requires the Earth population model to be declared as a model
assumption [3, Annex B, page 72] without fixing one. So a tool that applied
sheltering by default would produce a number that is not comparable with a NASA
compliance figure, however defensible the sheltering model is.

## What the tool must emit

This section is the acceptance criterion for milestone 09. It is assembled from
what the three documents require an assessment to present, and the largest single
contribution is the ESA space debris mitigation report annex, which lists the
contents of a reentry casualty risk analysis explicitly [3, Annex B (normative),
page 72].

Per surviving fragment, the tool emits its physical properties, meaning size,
shape, mass and material; its dynamic properties, meaning impact velocity and
kinetic energy; and its casualty area. All three are named in the ESA list.

Whether each fragment floats. This is in the ESA list as "Floating or
non-floating fragments" and it is the one required output that nothing else on
this board has an issue for.

The total casualty area, as a number separate from the risk, because NASA
Requirement 4.7-1d is satisfiable by either one.

The casualty risk as an expected number of casualties, and separately the
probability of casualty for an individual, because the FAA caps both and they are
independent.

What the expected number was summed over, as a list of the objects included rather
than as a total alone. ESA's clause counts every object released by the vehicle,
including ones that reenter at a different time [3, 5.5 b NOTE 4], so a total
whose object set is not printed cannot be checked against it.

Which fragments were counted and which were filtered out by the 15 joule
threshold, since the total is defined over objects above it and a reader cannot
check the total without the filter.

For a controlled reentry, the distance from every surviving impact above 15
joules to the nearest landmass, because NASA Requirement 4.7-1b is a geometric
test at 370 km and 50 km rather than a probability, and no risk number answers
it. The ESA list adds the Declared Re-entry Area and the Safety Re-entry Area.

The model assumptions, as declared values rather than as defaults buried in a
configuration. The ESA list names them individually: atmospheric density, Earth
gravitational attraction, solar activity as solar flux and geomagnetic index,
Earth population model, ground impact probability, fragmentation model, and
whether the reentry is controlled or uncontrolled.

The initial conditions, which the ESA list gives as orbital parameters and epoch
at the last orbit before fragmentation, and attitude at reentry.

The tool and methodology used, and a justification for the methodology and the
assumptions.

Everything in that list except the justification is a machine-writable artefact,
and every item of it is already carried by something this board has decided.
The per-fragment properties and the terminal states are record 0005. The declared
model assumptions and the input after defaults are the manifest of record 0008.
The artefacts that carry them are record 0009. What is new here is the specific
set, and two items that are not yet anywhere: the floating or non-floating flag,
and the distance to landmass for a controlled reentry.

## What this survey did not establish

ISO 24113 was not read. Its clause numbers, its threshold and its definitions are
not in this document except where the ESA standard quotes them and attributes
them.

ESSB-ST-U-004 was not read. The ESA standard states its own threshold normatively
in clause 5.5 b, so that number does not depend on the unread document, but clause
5.5 a applies ESSB-ST-U-004's requirements to reentry wholesale and clause 3.2.4
sends the determination of casualty risk there. What it adds, and whether anything
in it is tighter than section 5.5, is not established here.

ESSB-HB-U-002 Issue 2, which the ESA standard names as specifying a set of
minimum uncertainties for reentry casualty risk analysis, was not read. That
document is the most directly relevant thing found in this survey to record 0007,
which lists every uncertainty in this tool as an assumption, and it may contain
the numbers that would turn some of them into citations.

The U.S. Range Commanders Council Standard 321-20 and its Supplement were not
read. The 15 joule threshold is recorded from NASA's quotation of them.

NSS 1740.14, the superseded NASA safety standard, was not read. Its 8 m2 total
debris casualty area figure is recorded here as attributed to it by [6] and is
not quoted from it.

National licensing rules other than the United States were not examined. The
issue asks for the national rules that have started to name a threshold directly,
and only the FAA's were read.

The NASA casualty area equation is quoted from its surrounding prose rather than
from the equation itself, because the formula's symbols did not extract cleanly.
The 0.6 metre value and the sum over surviving objects are what is relied on.

## References

1. NASA-STD-8719.14C, "Process for Limiting Orbital Debris," 5 November 2021.
   <https://standards.nasa.gov/sites/default/files/standards/NASA/C/0/nasa-std-871914c.pdf>
2. Ostrom, C. et al. "Operational and Technical Updates to the Object Reentry
   Survival Analysis Tool." First International Orbital Debris Conference, 2019.
   NASA NTRS 20190033904. Recorded in `existing-codes.md`.
3. ESSB-ST-U-007 Issue 1, "ESA Space Debris Mitigation Requirements," 30 October
   2023.
   <https://technology.esa.int/upload/media/ESA-Space-Debris-Mitigation-Requirements-ESSB-ST-U-007-Issue1.pdf>
4. 14 CFR 450.101, "Safety criteria." Read from the eCFR renderer API for the
   current edition.
   <https://www.ecfr.gov/current/title-14/part-450/section-450.101>
5. 14 CFR 401.7, "Definitions." Read from the same source.
   <https://www.ecfr.gov/current/title-14/part-401/section-401.7>
6. Wu Ziniu, Hu Ruifeng, Qu Xi, Wang Xiang, Wu Zhe. "Space Debris Reentry
   Analysis Methods and Tools." Chinese Journal of Aeronautics 24 (2011)
   387-395, which is where the older NSS 1740.14 figure of 8 m2 is quoted.
