# The thermophysical material data that is publicly redistributable

Whether a fragment survives is decided by its material. The models can be right
and the answer still wrong if the numbers behind them are wrong, and the numbers
are somebody else's data set with somebody else's licence on it.

The result of this survey is negative and it is worth stating before the detail.
For none of the alloy classes this tool needs was an openly redistributable source
found that carries the demise properties over the temperature range that matters.
The good values are in four places, and all four are sold, licensed per seat, or
released only under an agreement: the CINDAS databases, MMPDS, the ASM handbooks,
and the material databases inside the reentry codes themselves. What is freely
reachable is either about the wrong materials, or is reachable to read without
being redistributable, and those two are different things that are easy to
conflate.

## What the demise model reads

The property list is taken from this tree rather than from the general literature.
`docs/decisions/0005-the-fragment.md` makes a fragment homogeneous, of one
material, carrying a reference into a material library rather than a copy of its
values, and makes a missing required property a refusal rather than a default.
`docs/decisions/0011-model-boundary.md` fixes which physics is in and which is
out, and each entry below is read by something that record keeps.

Density. Read by the mass properties, by the ballistic coefficient and by the
impact energy. Needed at ambient; its variation with temperature matters only
through volume expansion, which this model does not carry.

Specific heat capacity. Read by the lumped thermal model to turn absorbed heat
into a temperature rise. Needed as a function of temperature from ambient to the
melting point, because for the alloys in question it is not flat over that range.

Thermal conductivity. Read where conduction enters behind the thermal model's
interface, which `0011` places behind the same interface as the lumped model.
Needed as a function of temperature over the same range. A lumped model that never
grows a conduction mode would not read it, and the survey records it because the
interface exists.

Melting temperature. Read by the demise criterion. An alloy melts over a range
rather than at a point, so a single number is already a modelling choice, and
which of solidus, liquidus or a nominal value a source reports has to be recorded
with the value.

Heat of fusion. Read by the ablation rate once the melting temperature is reached.
This is the property most often absent from a general engineering data sheet, and
its absence is the most common reason a material that has a full mechanical
characterisation still has no usable demise record.

Emissivity. Read by the radiative cooling term, which is the only heat loss the
model has. Needed as a function of temperature, and needed for the surface
condition the fragment is actually in rather than for a polished coupon, because
an oxidised or roughened surface radiates differently from a virgin one.

Two properties the model does not read, recorded so that a source is not rejected
for lacking them and not accepted for carrying them. Mechanical strength, because
`0011` leaves structural failure loads out and that is what makes the breakup
altitude a convention. Heat of vaporisation, because the demise criterion stops at
melt and `0011` assumes melted material leaves the body immediately.

## What redistributable has to mean here

Two questions are separate and are answered separately below for every source.

Can it be read. Whether a person at this repository can obtain the document or the
data set at all, and at what cost.

Can it ship. Whether the values may be copied into a file this project
distributes. A permission to read, a public release approval, or an unlimited
distribution statement on a report is not a permission to redistribute the
content, and the two are easy to confuse. The clearest case of the confusion is
below, in the TPRC entry.

## The sources, and what each one permits

### NIST Standard Reference Data

Reachable, and not redistributable. This is the one most likely to be assumed open
because it is a United States government institute, and the assumption is wrong in
writing:

    Standard Reference Data (SRD) are copyrighted by the U.S. Secretary of
    Commerce on behalf of the United States of America. All rights reserved.
    None of our SRD may be reproduced, stored in a retrieval system or
    transmitted, in any form or by any means, electronic, mechanical,
    photocopying, recording or otherwise, without prior permission. [1]

The authority is the Standard Reference Data Act, Public Law 90-396, which
authorises the Secretary to secure copyright on behalf of the United States and to
authorise reproduction by others [1]. So the NIST Chemistry WebBook and the
JANAF thermochemical tables are readable, are the best available values for pure
elements, and may not be copied into a shipped material library without asking.

### The TPRC data series, Touloukian

Reachable, and not redistributable, and the two facts sit in the same file. The
thirteen volumes plus master index cover more than 5,000 materials for thermal
conductivity, specific heat, thermal expansion, thermal diffusivity and
emittance, which is most of the property list above. The Defense Technical
Information Center scans are mirrored on the Internet Archive and download without
an account:

    curl -sL -A "Mozilla/5.0" https://archive.org/download/DTIC_ADA129117/DTIC_ADA129117.pdf -o t14.pdf -w "http=%{http_code} size=%{size_download}\n"
    http=200 size=9624240

The report control form at the front of that file gives its distribution statement
as "Unlimited". Further into the same file is the publisher's page:

    Copyright: (c) 1979, Purdue Research Foundation
    IFI/Plenum Data Company is a division of Plenum Publishing Corporation

An unlimited distribution statement on a technical report is not a licence to
redistribute a copyrighted book that the report reproduces. The same data is sold
today: CINDAS LLC lists a Thermophysical Properties of Matter Database among its
products, alongside an Aerospace Structural Metals Database and a High Performance
Alloys Database [2]. So this source is the right data, readable at no cost, and
closed to a shipped artefact.

### CINDAS, MMPDS and the ASM handbooks

Not redistributable, and this is the class the issue predicted. CINDAS sells
databases by licence and publishes a multi-user licence agreement [2]. MMPDS is
the primary source of statistically based design allowables for aerospace metals
and is sold [3]. The ASM handbooks are sold per volume or per seat. Between them
they hold the best characterised values for exactly the aerospace alloys this tool
cares most about, and none of it can ship.

The cost of that is not only legal. A property taken from a source the reader
cannot open cannot be checked by the reader, so a material library sourced this
way fails this repository's own standard for evidence even where the licence
allowed it.

### MatWeb

Terms not established from this machine. MatWeb is the source the reentry
literature reaches for when a property is missing elsewhere, and one published
demise study states plainly that emissivity values absent from the NASA database
"were taken from the MatWeb database" [4, section 2.5]. Its terms of use page was
refused here:

    curl -sL -A "Mozilla/5.0" https://www.matweb.com/services/termsofuse.aspx -o /dev/null -w "http=%{http_code}\n"
    http=403

So what MatWeb permits is not recorded in this document, and any value that
reaches this project through it inherits an unread licence. That is a reason to
treat a MatWeb-sourced number as unusable here until the terms are read, not a
finding that it is forbidden.

### The NASA Debris Assessment Software database

Obtainable under an agreement, and not established as redistributable. DAS is the
tool NASA requires its own programs to use for compliance, and its material
database is what several published studies build on. The agency page states:

    Although approved for public release, NASA regulations require that a
    Software Usage Agreement must be obtained to acquire a copy of the
    NASA-developed software, DAS. [5]

The agreement itself was not read here, so whether it permits extracting the
material database and redistributing the values is not established. Approved for
public release and freely redistributable are not the same statement, which is the
same distinction as the TPRC entry above in a different form.

### ORSAT's material database

Not established as obtainable. The agency page records that thermal properties for
80 materials are included in a database in ORSAT, "with temperature-varying
properties listed for thermal conductivity, specific heat, and surface
emissivity" [6]. That is the shape this tool wants, temperature-varying rather
than constant. The page says nothing about whether ORSAT or its database is
distributed, and nothing read here says otherwise.

### NASA TPSX

Reachable, terms not established, and about a neighbouring material set. The
database advertises more than 1,500 materials in 32 categories with over 150
properties including density, thermal conductivity, specific heat and emissivity,
drawn from "multiple NASA and Industry Databases" [7]. The industry half is what
makes the licence question real rather than formal, and it was not answered here.
Its categories are thermal protection materials, which reach the ceramics and the
carbon-phenolics below and not the structural alloys.

### NASA Reference Publication 1289

Reachable, redistributable, and about the wrong materials for most of this list.
It is a United States government work, 236 pages, and downloads without an
account:

    curl -sL -A "Mozilla/5.0" https://ntrs.nasa.gov/api/citations/19930009576/downloads/19930009576.pdf -o rp1289.pdf -w "http=%{http_code} size=%{size_download}\n"
    http=200 size=3847743

Its contents are ablators and thermal protection materials: carbon-phenolics,
silica-phenolics, silica-teflon, silica-silicones, carbon materials with ablation
data, and elastomeric ablators [8]. It is the one large compendium found in this
survey that could be copied into a shipped library, and it holds almost nothing
for aluminium, titanium, steel or nickel. That mismatch is the honest summary of
the whole licensing position.

### The ESA Space Materials Database

Reachable, terms not stated. The site covers metals, alloys, glasses, polymers,
ceramics and composites with physical and thermal properties [9]. Its landing page
carries no licence, no terms of use and no redistribution statement, and none was
found elsewhere on it here.

## The material classes

The property list is the same for every class, so it is not repeated. What differs
is what exists.

### Aluminium alloys

The class that decides most reentries, because most structure is aluminium and
because it melts low enough that the demise question is usually about everything
else. AA7075 is the representative the reentry field has settled on.

Measurements exist and they are recent. The ESA CHARDEM project measured specific
heat capacity, heats of solid state phase transitions, heat of fusion, density,
volume expansion and thermal conductivity for aluminium alloy 7075 among others
[10]. Where the resulting values are published, and under what terms, is not
established here.

No redistributable source of the full property set was found. Pure aluminium is
covered by NIST SRD, which cannot ship; 7075 as an alloy is covered by MMPDS,
CINDAS and ASM, none of which can ship.

### Titanium alloys

The class that decides whether a tank reaches the ground. ESA's own standard names
tanks, reaction wheels and magnetorquers as the equipment that commonly poses a
casualty risk after reentry [11, 3.2.11], and a titanium tank is the canonical
survivor. Ti6Al4V is the representative.

Measurements exist. CHARDEM covers Ti6Al4V and a 3D printed titanium alloy, and
its own comparison plot for titanium specific heat names eight prior literature
sources against which the new measurement was placed, running from 1977 to 2012
[10]. So for this one alloy the published literature is genuinely deep. Whether
any of those sources is redistributable was not checked, and the plot is a picture
rather than a table, so no values are taken from it here.

No redistributable source of the full property set was found.

### Stainless and other steels

AISI 316L is the representative and is in CHARDEM alongside the aluminium and the
titanium [10]. The same position holds: measurements exist, the values sit in
sold handbooks, and nothing redistributable carrying the demise set was found.

### Nickel superalloys

Nothing was established for this class. No source read here carries the demise
property set for a named nickel superalloy, redistributable or otherwise, and no
measurement campaign covering one was found. Recorded as an absence rather than as
a negative result, because the search for it was shallower than for the three
classes above.

### Magnesium

Nothing was established. Magnesium matters out of proportion to its mass share
because the 78 km breakup convention this board inherited descends from a single
sentence about magnesium and aluminium structure, which
`docs/survey/existing-codes.md` records with its provenance. No source carrying its
demise properties was read here.

### Beryllium

Nothing was established, and this class has a second problem the others do not.
Beryllium is toxic in the forms produced by machining and by combustion, so its
data tends to sit in restricted defence literature rather than in open handbooks.
That is a claim about where the data sits, not a measurement of it, and it was not
checked.

### Glass and fused silica

Partly covered, from an unexpected direction. NASA RP-1289 carries silica-phenolic,
silica-teflon and silica-silicone systems with thermophysical property data and can
be redistributed [8]. Those are ablators rather than the fused silica of a window
or a lens, so the coverage is adjacent rather than direct. Optical fused silica
itself was not established from any redistributable source here.

### Carbon fibre composites

This class needs its own paragraph because the demise criterion does not apply to
it. A composite does not melt in the way the criterion assumes, and the codes
handle it by convention.

What the reference codes do is already recorded on this board, in
`docs/survey/existing-codes.md`, from the codes' own publications. ORSAT 7.1 is
described as having rewritten its thermal demise model completely and added a
pyrolysis model for fibre reinforced plastics. SCARAB 4 added an advanced demise
and ablation model covering six material types including composites. Neither
source read for that survey states the numerical assumptions behind either, so
what each code assumes is named as an area of work rather than as a number, and
this document does not improve on it.

What is different from that reference set is what the published demise studies do,
which is simpler and worth knowing before adopting it. The study cited above
derives its material database from DAS 2.0 and states that "All the material
properties are assumed temperature independent" [4, section 2.5]. A composite
treated that way is a lumped body with a melting temperature that has no physical
referent.

Measurements exist. CHARDEM includes a CFRP among its eight test articles,
alongside a high pressure tank and a solar panel, and put them through the same
standard demisability test procedure of thermophysical measurement, high enthalpy
wind tunnel testing and numerical rebuilding [10].

### The ceramics in optics and reaction wheels

Partly covered and partly absent. Silicon carbide is in CHARDEM [10], and the
ESA standard's own note names reaction wheels among the equipment that survives
[11, 3.2.11]. The optical ceramics of a telescope or a star tracker were not
established from any source here.

## What has no usable public data at all

Stated as the issue asks, and stated as the state of this survey rather than as
the state of the world. On the evidence gathered here, no material class in the
list has a complete, redistributable, temperature-resolved property set behind it.
Three classes have at least a recent measurement campaign that is named and
findable, which are aluminium, titanium and stainless steel, plus silicon carbide
and one CFRP through the same project. Four classes have nothing established at
all here, which are nickel superalloys, magnesium, beryllium and optical ceramics.

## What this feeds

The licensing position above is the input to entry 4 of #1. The choice it forces
is not a preference. Either this project ships no material library and requires an
operator to supply one, or it ships a small library built from sources that permit
it and states for every row where the value came from and what it is worth, or it
ships nothing until a licence is bought. The first two are compatible with what
this repository already decided in `docs/decisions/0005-the-fragment.md`, which
makes a missing required property a refusal rather than a default.

The schema that carries provenance per row is #48, and the licence position of
every shipped row is #49. This document is their input and does not pre-empt
either.

## What this survey did not establish

The terms of the NASA Software Usage Agreement for DAS. Whether the material
database inside it may be extracted and redistributed is the single question that
would most change the answer above, and it was not read.

The terms of use of MatWeb, refused from this machine as recorded.

The terms of use of NASA TPSX and of the ESA Space Materials Database.

Where the CHARDEM measured values are published. The project is named, its
material list and its measured property list are recorded from its own
presentation, and no data table from it was read here. The presentation shows a
comparison plot rather than a table, so no number in this document comes from it.

Whether any of the eight literature sources named on that plot is itself
redistributable.

Nickel superalloys, magnesium, beryllium and optical ceramics were searched less
thoroughly than the other classes. Their entries above are absences in this survey
rather than established absences in the literature.

No value for any property of any material is quoted anywhere in this document.
That is deliberate. Quoting one would mean copying it out of a source whose licence
this document has just recorded as unread or restrictive.

## References

1. NIST, "Standard Reference Data Act," on the copyright and licensing of NIST
   Standard Reference Data. <https://www.nist.gov/srd/public-law>
2. CINDAS LLC product listing, naming the Thermophysical Properties of Matter
   Database, the Aerospace Structural Metals Database, the High Performance Alloys
   Database and a multi-user licence agreement. <https://cindasdata.com/>
3. MMPDS, Metallic Materials Properties Development and Standardization.
   <https://www.mmpds.org/>
4. Trisolini, M., Lewis, H. G., Colombo, C. "Demisability and survivability
   sensitivity to design-for-demise techniques." Preprint, arXiv:1910.06397.
   <https://arxiv.org/pdf/1910.06397>
5. NASA Orbital Debris Program Office, Debris Assessment Software page.
   <https://orbitaldebris.jsc.nasa.gov/mitigation/debris-assessment-software.html>
6. NASA Orbital Debris Program Office, ORSAT page.
   <https://orbitaldebris.jsc.nasa.gov/reentry/orsat.html>
7. NASA TPSX Materials Properties Database. <https://tpsx.arc.nasa.gov/>
8. Williams, S. D., Curry, D. M. "Thermal Protection Materials: Thermophysical
   Property Data." NASA Reference Publication 1289, December 1992.
   <https://ntrs.nasa.gov/api/citations/19930009576/downloads/19930009576.pdf>
9. Space Materials Database. <https://spacematdb.com/>
10. Schleutker, T., Gülhan, A., Lips, T., Kaschnitz, E., Bonvoisin, B. "CHARDEM:
    Experimental investigations on the demisability of space relevant materials."
    Presentation, ESA Indico event 128.
    <https://indico.esa.int/event/128/attachments/736/894/02_DLR___CSID_CHARDEM.pdf>
11. ESSB-ST-U-007 Issue 1, "ESA Space Debris Mitigation Requirements," 30 October
    2023. Recorded in `standards.md`.
    <https://technology.esa.int/upload/media/ESA-Space-Debris-Mitigation-Requirements-ESSB-ST-U-007-Issue1.pdf>
12. Touloukian, Y. S. et al. "Thermophysical Properties of Matter, the TPRC Data
    Series." Purdue Research Foundation, IFI/Plenum. Volume 14, master index, read
    as the DTIC scan mirrored at
    <https://archive.org/details/DTIC_ADA129117>
