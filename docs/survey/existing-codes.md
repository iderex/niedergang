# The reentry breakup codes that already exist

Every architecture decision on this board is going to be argued against these
codes. An argument against a code whose behaviour is remembered rather than read
is worthless, so this document records what each one does from its own
documentation and from published papers about it, with a source on every claim.

Where a point is not published, the document says so rather than filling the gap
with a plausible sentence. Several of the entries below are mostly such
statements, and that is itself the finding: the codes that produce the numbers
regulators read are, with one exception, not obtainable and not described in
enough detail for an outsider to reproduce.

## The two families, and who defines them

The split every source uses is between object-oriented and spacecraft-oriented
codes, and the definition comes from Lips and Fritsche [1] by way of the review
that restates it:

> The main idea of the object-oriented method is to simplify the complicated
> object geometry into simple shapes like sphere, cylinder, box, etc. DAS, ORSAT
> and DRAPS are tools belonging to this method. The spacecraft-oriented method
> aims to simulate spacecraft reentry as real as possible. [2, section 1]

The same source states the cost of the second family plainly, and this is the
sentence this project has to answer rather than repeat:

> The spacecraft-oriented method should be more accurate than the
> object-oriented method, but it needs more modeling efforts and computational
> resources because of much more complex analysis strategy. [2, section 1]

The two families are not two implementations of one model. They differ on
whether breakup is predicted or assumed, which is the single largest modelling
difference between them, and it is recorded per code below.

## SCARAB

SCARAB is spacecraft-oriented in its own words. The 2025 review by the group
that develops it calls it "a spacecraft-oriented re-entry break-up simulation
code" and expands the name as Spacecraft Atmospheric Re-entry and Aerothermal
Breakup [3, abstract]. Development started under the lead of HTG in 1995, and
the same paper describes the goal as covering "all the multidisciplinary aspects
of a re-entry in one tool" with "relatively high level of detail with the
computational resources available back then" [3, abstract].

What it takes is a three-dimensional geometric model of the spacecraft. The tool
"constructs a 3D geometric model of the spacecraft, with volume panel grid which
serves as foundation for the computation of the aerodynamic forces and torques
experienced by the spacecraft during re-entry, as well as being the basis for
the thermal computation" [3, section 2]. The scale of that model is published
for one case: the BeppoSAX geometry in SCARAB has 859 primitives, 177,708
surface panels and 72,548 volume panels [2, section 2.2]. Structural analysis
needs an extra input that the object-oriented codes have no equivalent of: "some
'cuts' should be defined in the geometric modeling step, at which the stress
will be analyzed to judge whether breakup happens" [2, section 2.2].

What it emits is a fragment population and the risk that follows from it. The
review describes it as "an integrated software package (six degrees-of-freedom
flight dynamics, aerodynamics, aerothermodynamics, thermal- and structural
analysis) used to perform re-entry risk assessments (quantification,
characterization and monitoring of surviving fragments during re-entry)" [3,
section 2].

Where it sits relative to the other ESA tool is stated by the same authors. For
ESA projects compliance is assessed with DRAMA, whose SESAM module is object
oriented, and "if needed, ESA's more detailed spacecraft-oriented re-entry tool
SCARAB can be used for ground risk assessment" [3, section 1]. SCARAB is
therefore the escalation rather than the routine instrument.

Obtainability is the point this survey could not settle. HTG's own product page
describes the software and gives no version number, no licence, no price and no
download route [4]. Nothing read for this survey states terms on which a binary
or the source could be had by somebody outside an ESA contract.

Published validation exists and is stated in general terms rather than as a case
list: "The software has been validated with in-flight measurements, re-entry
observations and wind-tunnel experiments, and it has been compared to other
re-entry prediction tools of the international community" [3, section 2]. Named
applications include ATV, ROSAT, Ariane 5, BeppoSAX and the Mir space station
[2, section 2.2; 3, abstract].

Where it stops is not stated as a list of excluded effects in anything read for
this survey. The closest statement is about resolution rather than physics: the
HTG page says the full mathematical treatment "would need using super
computers", so simplifications are made to keep the tool on a personal computer
[4].

## DRAMA, and its reentry modules SESAM and SERAM

DRAMA is ESA's Debris Risk Assessment and Mitigation Analysis tool, and the part
that matters here is the Survival And Risk Analysis area, which contains SESAM
and SERAM. SESAM is object-oriented and the developer says so while stating the
assumption that goes with it: "The typical assumption for object oriented tools
like SESAM is the release of the particular simple objects from the compartment
at a certain altitude" [5]. That sentence is the whole difference from SCARAB,
because it means the breakup event is an input rather than a result.

Its purpose is stated as determining "those fragments of a satellite that survive
the atmospheric re-entry and may pose a risk to the world population" [5].

What it takes is a fragment list of primitives and an initial state: the model is
built from "simple geometric objects like spheres, cylinders, boxes or plates,
which are characterized by their measures, mass and material properties", and
"it's initial state (position, epoch and velocity) is defined" [5]. SERAM is the
second half: it "analyses the results of the SESAM simulation" to produce the
risk [5].

The stage it is prescribed for is stated directly, and it is the early one: the
object-oriented approach is "advantageous, if a first quick assessment for the
on-ground risk is needed, e.g. during the conceptual phase of spacecraft design
or at an earlier time before the expected re-entry" [5].

Obtainability is better than any other code here except one. DRAMA is downloaded
from ESA's Space Debris User Portal [5, 6]. An account is required, which is not
a matter of inference: requesting the DRAMA page on that portal returns a
redirect to a login route rather than the page.

    curl -s -o /dev/null -w '%{http_code} %{redirect_url}\n' https://sdup.esoc.esa.int/drama/
    302 https://sdup.esoc.esa.int/login?next=%2Fdrama%2F

That login route redirects onward to an OAuth authorisation endpoint.

The portal itself states an eligibility rule in the text read for this survey
only for DISCOS, where an applicant has to "belong to a research institute, to a
government organisation, or to an industrial company of an ESA Member State
(e.g., not as an individual)" [6]. Whether the same rule governs a DRAMA licence
was not established here, and it is not assumed.

Published validation specific to SESAM was not found in the sources read.

Where it stops follows from the family. Breakup is not predicted, and the shape
set is primitives, so a component whose geometry is not a primitive is
represented by a nearest primitive chosen by the person writing the input.

## ORSAT

ORSAT is object-oriented and is NASA's operational instrument. The Orbital Debris
Program Office describes it as "the primary NASA computer code for predicting the
reentry survivability of satellite and launch vehicle upper stage components
entering from orbital decay or from controlled entry" [7]. The shape set is
small and is published: in DAS and ORSAT "four types of object shapes can be
modeled, i.e. sphere, cylinder, flat plate and box. Only solid objects can be
modeled in DAS, while both solid and hollow ones can be analyzed in ORSAT" [2,
section 2.1].

What it takes is a component list with shapes, dimensions, masses and materials,
an entry state, and a motion type per component rather than a solved attitude.
The object-oriented family uses "Three degrees of freedom (DOF) ballistic model
... The attitude dynamic equation of object is not directly solved but predefined
as specific motion according to object shapes", and for a cylinder ORSAT offers
four motion types [2, section 2.1]. The atmosphere is selectable, and which one
is used is tied to the case: NRLMSISE-00 replaced MSISE-90 in the code, and "this
model is reserved for use in examining controlled reentries of spacecraft and
rocket bodies", while "Natural decay reentry trajectories are simulated using the
1976 US Standard Atmosphere" [8, section 2.5]. Entry interface is "typically
defined as 122 km altitude for ORSAT analysis" [8, section 2.2]. The material
database is described by the agency page as holding 80 materials [7].

What it emits is a debris casualty area and the expectation of casualty that
follows from it. Version 6.2.1 implements the Opiela-Matney formula, published in
the same paper as

    DCA = 0.278 + A_obj + 0.3 * P_obj

with a later simplification removing the dependence on perimeter [8, section
2.7].

The stage it is prescribed for is compliance with NASA-STD-8719.14, and the
threshold the number is compared against is a risk of less than 1 in 10,000 [7].

Obtainability could not be established. The agency page describes ORSAT without
offering a route to a copy [7], and this is worth contrasting with the DAS page
of the same office, which sets out a Software Usage Agreement route explicitly
[9]. Nothing read here states terms on which ORSAT could be obtained by somebody
outside NASA.

Published validation is a list of comparison cases rather than a validation
report: hollow sphere reentry compared with SCARAB, a barium fuel rod, the
SPARTAN spacecraft, Delta second stages and UARS [2, section 2.1].

Where it stops is stated by its own developers, and the clearest statement is
about breakup. Standard ORSAT analysis "has assumed breakup to occur at 78 km
(originally 42 nmi) altitude, regardless of spacecraft mass, shape, or size",
and the paper quotes the source of that number, a report written in support of
the Office of Commercial Space Transportation, which said only that
"Magnesium/aluminum structure consistently exhibits catastrophic failure at 42
nmi altitude" [8, section 2.3]. The same paper then says what that costs: "it may
be introducing bias into standard ORSAT analysis to always use 78 km as the final
altitude of the parent body" [8, section 2.3]. The general form of the limitation
is stated for the whole family: "The structural analysis model is always omitted
in the object-oriented method, so the breakup event cannot be directly predicted"
[2, section 2.1].

## DAS

DAS is NASA's Debris Assessment Software, and it is a compliance tool rather than
a physics tool. The agency page describes it as verifying "spacecraft, upper
stages, and payloads for compliance with NASA's debris mitigation requirements,
including limiting debris generation, spacecraft vulnerability, postmission
lifetime, and entry safety", against NASA Technical Standard 8719.14C [9].

Its survivability model is the object-oriented one, restricted further than
ORSAT's: four shapes, solid objects only [2, section 2.1]. It uses the 78 km
breakup convention [2, section 2.1].

The casualty arithmetic it implements is published. The equivalent casualty area
of one fragment combines the fragment's cross section with a projected human
cross section of 0.36 square metres, the total is the sum over surviving
fragments, and the threshold is "1:10 000 per reentry event" [2, section 2.3].

Obtainability is the best documented of any code here and is still not open.
"Although approved for public release, NASA regulations require that a Software
Usage Agreement must be obtained to acquire a copy", requested through the NASA
Software Catalog [9]. The current version stated on that page is 3.2.7, released
10 April 2026, and it is described as optimised for one desktop operating system
[9].

Published validation of DAS as such was not found in the sources read. Its
standing rests on the standard it implements rather than on cases.

Where it stops is the same place ORSAT stops, one step earlier: no structural
analysis, an assumed breakup altitude, four shapes, and no hollow objects.

## DEBRISK

DEBRISK is CNES's object-oriented tool and the one code in this survey that an
outsider can actually obtain. CNES calls it "the CNES reference tool for ablation
computation during re-entry", says it "evaluates the survivability of fragments
issued from a satellite entering the Earth's atmosphere", and states the family
in one sentence: "This software uses an object oriented approach" [10].

What it takes is a satellite described as a set of objects, each with "shape, its
size, its mass and its material" [10]. What it emits is "a list of the surviving
objects and their characteristics upon ground arrival" [10]. The modelled
physics is listed as trajectory and ablation with "thermal heat, ablation,
conduction, oxidation" [10], and the presence of oxidation is worth noting
against the boundary this board is drawing in issue #63.

The stage it is prescribed for is named by CNES's own people as certification:
in a 2024 presentation the CNES tool set is described with DEBRISK as the
"certification tool", PAMPERO as the "spacecraft-oriented tool" and BLIZZARD as
the "Inviscid CFD code" [11].

Obtainability is the finding. "Since version 3.5, distribution is no more subject
to a preliminary demand", the licence is described as a free owner licence, the
current version is 3.7.1, and it runs on Windows and Linux on a Java runtime of
1.8 or later [10]. This is the only code in this survey that can be downloaded
and run by somebody with no institutional standing.

Published validation exists under its own name: a 2013 conference paper on
validation and sensitivity analysis [12], and a 2020 journal paper on extending
the tool to complex shapes whose method was checked against a wind tunnel
campaign [13]. Neither full text was read for this survey; the entries are
catalogue records and abstracts.

Where it stops is the family limit again, and the 2020 paper exists because of
it: the object-oriented approach is limited to primitive shapes, which is what
that work sets out to widen [13].

## PAMPERO

PAMPERO is CNES's spacecraft-oriented code, developed with R.Tech. Its own
authors state the aim: "PAMPERO aims to simulate the complete atmospheric reentry
of an entire satellite, launcher or the associated fragments due to the breakup
process" [14]. The modelling is multidisciplinary, covering flight dynamics,
aerodynamics, thermal analysis, mechanical stress and fragmentation, with six
degrees of freedom and multi-material objects [14].

Its role beside DEBRISK is stated in the same place as DEBRISK's, and it is the
same shape as the SCARAB and SESAM pairing at ESA: the certification instrument
is the object-oriented one and the spacecraft-oriented one is the deeper tool
[11].

Obtainability, published validation as a case list, and a statement of what it
does not model were not established from the sources read. A 2024 presentation
applies the CNES tools to the January 1997 Delta II second stage reentry with
the stated aim of validating "our re-entry risk verification tools and
procedures" [11], which is a validation activity rather than a published result
this survey has read.

## DRAPS

DRAPS is the Debris Reentry and Ablation Prediction System, developed at Tsinghua
University with the China Academy of Space Technology, and its authors place it
in the object-oriented family: "DRAPS developed by present authors falls within
the category of the object-oriented method" [2, section 3].

Its numerical core is stated to be the same as ORSAT's: "DRAPS adopts 3 DOF
ballistic model for trajectory prediction and zero-dimensional or
one-dimensional heat conduction approach for ablation analysis, which is the same
as ORSAT" [2, section 3]. What it changes is the shape set and the treatment of
uncertainty. The object shapes are "extended to 15 types in DRAPS as well as 51
predefined motions", including half spheres, cones, and cylinder and box
assemblies [2, section 3.1], against the four shapes of DAS and ORSAT.

The uncertainty point is the one this board should read closely, because it is
the same argument issue #18 makes. Its authors write that the established tools
"are most deterministic other than probabilistic. But there exist a lot of
uncertainty sources which may affect the analytical results significantly, such
as initial conditions, atmospheric models, aerodynamic models and so on. In
DRAPS, a simple Monte Carlo method has been integrated to account for these
uncertainty effects and assess reentry risk in a probabilistic manner" [2,
section 1].

Validation of its aerodynamic and aerothermal models is by direct simulation
Monte Carlo rather than by flight cases, stated in the paper's own abstract [2],
and a comparison against ORSAT is cited as confirming "reasonable agreement
between different tools" [2, section 1].

Obtainability was not established. Nothing read for this survey offers a route to
a copy.

## Where each code stops

The one effect excluded by the whole object-oriented family, stated by a source
rather than inferred, is structural analysis, and with it the prediction of
breakup itself [2, section 2.1]. Everything downstream of a breakup altitude in
DAS, ORSAT, SESAM, DEBRISK and DRAPS rests on a number the operator supplies or
inherits.

That number has a traceable and thin provenance. The 78 km convention descends
from a single sentence about magnesium and aluminium structure in a report
supporting commercial launch licensing, and the group that uses it most has
written down that always using it may bias their own analyses [8, section 2.3].
The remedy they describe is to compute a breakup altitude per parent body from
the altitude at which radiative equilibrium surface temperature reaches the
parent material's melting point, which makes low ballistic coefficient objects
such as CubeSats break up higher than 78 km and dense ones lower [8, section
2.3].

For the spacecraft-oriented family, no source read for this survey states a list
of deliberately excluded effects. That absence is recorded as an absence.

Three engineering rules of thumb are in circulation in the community and are
quoted, in these words, in the 2025 SCARAB review [3, section 1]:

> Spacecraft below ~400 kg are fully demisable
>
> 10-40% of the re-entering mass survives to ground
>
> The main breakup of a spacecraft occurs at ~78 km

The same paper immediately says of them: "Some of these statements are based on
little evidence, others rely on old data" [3, section 1]. That sentence is
written by the group with the largest archive of these simulations, and it is
the most direct published support for this project's premise that exists in the
sources read.

## The age of the core routines, and what has moved since

This is the argument the project rests on, so it is recorded with more care than
the rest, and it does not come out as one-sided as the board's own framing.

ORSAT's provenance is old and its models are not frozen. Its developers write
that the tool "has been used in the NASA Orbital Debris Program Office for over
25 years", that version 6.0 was released in 2005 with 6.1, 6.2 and 6.2.1
following, and that "The ORSAT software was originally written in Fortran 77;
over the last 25 years of its development, features and external source codes
have been added with different code standards", with a port to Fortran 95 made
for version 6.2.1 [8, sections 1 and 2.6]. The models have moved substantially
and recently. The same paper adds a Tauber-Sutton radiation correlation and the
QRAD equilibrium radiation code, replaces MSISE-90 with NRLMSISE-00, replaces
the latitude-averaged population basis with a statistical model of initial
conditions, and updates the casualty area formula [8, sections 2.2 to 2.7]. A
2023 record for version 7.1 describes "five years of new thermal and aerodynamic
model development" and states that "The thermal demise model was completely
rewritten" with a new pyrolysis model for fibre reinforced plastics [15].

SCARAB's provenance is comparable and its trajectory is the same. Development
started in 1995 and the 2025 review covers three decades [3]. Its archive holds
analyses for 37 satellite and launcher projects with more than 90 unique design
iterations and over 1,000 simulations, the earliest from 2001, and the majority
of those simulations were performed with SCARAB 3.0 or 3.1L [3, sections 2 and
3]. A successor exists: a 2021 paper describes SCARAB 4 as an activity to
"update and extend the current SCARAB (3.1L) to improve the calculation of the
re-entry casualty risk", adding an advanced demise and ablation model covering
six material types including composites, local heating based on local radius of
curvature, shock impingement, radiative shock heating, break-up triggers at
component interfaces, and a direct interface to DRAMA [16].

So the honest form of this project's premise is narrower than "the codes are
old". The provenance is genuinely old, the numerical cores of the
object-oriented tools are a 3DOF ballistic integration with lumped or
one-dimensional conduction that has not changed in family since ORSAT, and the
breakup assumption at the centre of all of them rests on one sentence from a
licensing support document. The models around that core have been actively
worked on through 2021 to 2026 by the groups that own them. A claim that the
field is standing still is refuted by the sources above; a claim that the
assumption at the centre of it is thin is supported by them, in the developers'
own words.

## What this survey did not establish

The board's issue asks which of two assessment stages each code is prescribed
for. The two stages are not defined anywhere on this board, and no source read
here names a two-stage scheme in those terms. What is recorded above instead is
what each code's own documentation says about where in a project it is used,
which is a weaker answer to a question that has to be fixed before it can be
answered properly. This belongs in the reconciliation of issue #10.

Availability terms for SCARAB, ORSAT, PAMPERO and DRAPS were not found. In each
case the sources describe the tool without offering a route to a copy. That is
not the same as a statement that no route exists, and it is not written here as
one.

Validation for SESAM, DAS and PAMPERO was not found as published case results.

No full text was read for references 1, 12, 13 and 15. They are cited from
catalogue records and abstracts, and any claim resting on them is marked in the
text where it is made.

The README of this repository carried two claims about other codes that this
survey could not source, and both have since been repaired. It stated that
SCARAB 3.1L "is a decade old and its core routines go back twenty-five years".
Nothing read for this survey gives a release date for 3.1L, so the first half of
that sentence was unverified here; the second half is consistent with the 1995
development start [3]. It also named STELA alongside PAMPERO as a CNES reentry
tool; the CNES material read here names DEBRISK, PAMPERO and BLIZZARD as the
reentry tool set [11], and STELA was not encountered in that role in any source
read. Issue #109 replaced both sentences with what this survey does establish
and linked this document from the paragraph that held them, so a reader
checking a sentence on the front page now reaches the reading behind it. The
finding stays recorded here because it is the one place that says the front
page was checked against this survey and what that check could not source.

## References

1. Lips, T., Fritsche, B. "A comparison of commonly used re-entry analysis
   tools." Acta Astronautica 57 (2005) 312-323. Cited here through reference 2,
   which restates its classification. Full text not read.
   <https://www.sciencedirect.com/science/article/abs/pii/S0094576505000767>
2. Wu Ziniu, Hu Ruifeng, Qu Xi, Wang Xiang, Wu Zhe. "Space Debris Reentry
   Analysis Methods and Tools." Chinese Journal of Aeronautics 24 (2011)
   387-395. doi:10.1016/S1000-9361(11)60046-0.
   <https://web.xidian.edu.cn/rfhu/files/20140306_211807.pdf>
3. Kärräng, P., Lips, T., Breslau, A., Fritsche, B. "Review of 30 Years of
   SCARAB Re-entry Break-up Analysis." Proc. 9th European Conference on Space
   Debris, Bonn, 1-4 April 2025, ESA Space Debris Office.
   <https://conference.sdo.esoc.esa.int/proceedings/sdc9/paper/144/SDC9-paper144.pdf>
4. HTG Hyperschall Technologie Göttingen GmbH, SCARAB product page.
   <https://www.htg-gmbh.com/en/htg-gmbh/software/scarab/>
5. HTG Hyperschall Technologie Göttingen GmbH, DRAMA / SESAM product page.
   <https://www.htg-gmbh.com/en/htg-gmbh/software/dramasesam/>
6. ESA Space Debris User Portal. <https://sdup.esoc.esa.int/>
7. NASA Orbital Debris Program Office, ORSAT page.
   <https://orbitaldebris.jsc.nasa.gov/reentry/orsat.html>
8. Ostrom, C., Greene, B., Smith, A., Toledo-Burdett, R., Matney, M., Opiela, J.,
   Marichalar, J., Bacon, J., Sanchez, C. "Operational and Technical Updates to
   the Object Reentry Survival Analysis Tool." First International Orbital Debris
   Conference, 2019. NASA NTRS 20190033904.
   <https://ntrs.nasa.gov/api/citations/20190033904/downloads/20190033904.pdf>
9. NASA Orbital Debris Program Office, Debris Assessment Software page.
   <https://orbitaldebris.jsc.nasa.gov/mitigation/debris-assessment-software.html>
10. CNES, DEBRISK page on Connect by CNES.
    <https://www.connectbycnes.fr/en/debrisk>
11. Galera, S., Constant, E., Annaloro, J., Spel, M. "Re-Entry Survival Analysis
    Using The CNES Re-Entry Tools." Clean Space Days, 8-11 October 2024.
    <https://indico.esa.int/event/516/contributions/10004/>
12. Omaly, P., Magnin Vella, C., Galera, S. "DEBRISK, CNES Tool for Re-Entry
    Survivability Prediction: Validation and Sensitivity Analysis." ESA SP-715,
    2013. Catalogue record only; full text not read.
    <https://ui.adsabs.harvard.edu/abs/2013ESASP.715E..76O/abstract>
13. "Aerothermodynamics modelling of complex shapes in the DEBRISK atmospheric
    reentry tool: Methodology and validation." Acta Astronautica, 2020.
    Abstract only; full text not read.
    <https://www.sciencedirect.com/science/article/abs/pii/S009457652030134X>
14. Annaloro, J., Galera, S., Prigent, G., Thiebaut, C., Omaly, P. "Latest
    Improvements on the CNES Spacecraft-Oriented Tool: PAMPERO." Clean Space
    Industrial Days 2018, ESTEC, 23-25 October 2018.
    <https://indico.esa.int/event/234/contributions/4048/>
15. "Recent Updates to the Object Reentry Survival Analysis Tool (ORSAT) Version
    7.1." NASA NTRS 20230003426. Catalogue record only; full text not read.
    <https://ntrs.nasa.gov/citations/20230003426>
16. Kanzler, R., Lips, T., Fritsche, B., Breslau, A., Kärräng, P., Spel, M.,
    Pagan, A., Herdrich, G., Lemmens, S. "SCARAB 4 - Extension of the
    high-fidelity re-entry break-up simulation software based on new measurement
    types." Proc. 8th European Conference on Space Debris, 2021.
    <https://conference.sdo.esoc.esa.int/proceedings/sdc8/paper/13>
