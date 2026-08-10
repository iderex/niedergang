# 0023. Oxidation and other exothermic surface chemistry

## Date

2026-08-10

## Status

accepted

## Question

A hot alloy surface in a reentry boundary layer oxidises, the reaction releases
energy at the surface, and that energy is heat the body did not receive from the
flow. Does the first implementation carry a term for it, and if it does not, what
does leaving it out cost.

Issue #63 asks the second half in a specific form: which materials it matters
for, the altitude and temperature range where the flux is significant, the
published estimates of its effect on demise altitude for a representative
component, and whether those estimates agree. Two of those four are answerable
from the published record and two are not, and which is which is the useful part
of this record rather than a gap in it.

Record 0011 already places this effect outside the model boundary and says of it
that the error from leaving it out is not established, that no number was found,
and that the direction is the only thing that can be stated. That record's `What
would change this` names a sensitivity run over the oxidation efficiency
parameter of a code that has one as the measurement that would move it. This
record is the attempt to find that measurement, and it reports what the attempt
produced rather than what it was looking for.

## Options considered

A term now, in the form the one openly described implementation uses. ORSAT
computes the net heat rate as the hot wall rate plus an oxidation rate minus the
reradiation rate, and

    curl -sL -o orsat1997.pdf https://ntrs.nasa.gov/api/citations/19970040121/downloads/19970040121.pdf
    pdftotext orsat1997.pdf - | grep -o 'The oxidation heat rate is based on an empirical constant times the cold wall heat rate times the oxide heat of formation as used in ORSAT.'
    The oxidation heat rate is based on an empirical constant times the cold wall heat rate times the oxide heat of formation as used in ORSAT.

The cost is the empirical constant. It is dimensionless, it runs from zero to
one, and the measurements below put a real recovered component on both sides of
that range. Shipping a value inside it is shipping a tuning parameter as physics,
which is the substitution this project exists to refuse.

A term now, with the constant as a required operator input and no default. This
looks like the honest version of the option above and it is worse. Record 0010
makes a missing input a refusal, so every run would stop until the operator
supplies a number the field has not measured, and the operator is the person
least equipped to supply it. It converts an unknown in the model into an obstacle
in front of the user without reducing the unknown.

The mechanism the recovered hardware actually shows, which is not the same thing.
The metallurgy of the two Delta II tanks points at molten aluminium deposited on
a steel or titanium surface, oxidising, having its oxide layer stripped by the
shear of the flow, and burning. Modelling that needs material from one component
to arrive on another, which record 0003 excludes by flying every fragment alone.
It is a change of architecture rather than a surface term, and it is weighed here
only so that a later reader does not mistake the option taken for a fix to it.

Out silently. Listed so that it is visibly rejected rather than absent. The
effect is the entry where this tool departs furthest from the reference codes,
and an operator who does not know that cannot judge the number.

Out, disclosed, and costed as far as the literature allows, with the heat balance
written to take an additional surface source so that adding a term later is an
implementation. Taken.

## Decision

Oxidation and other exothermic surface chemistry are not modelled in the first
implementation. No oxidation term enters the heat balance, and no oxidation
efficiency, oxide heat of formation or equivalent parameter appears in the
scenario or in the material record.

The heat balance is written as a sum of surface sources and sinks in which an
additional source term can be added without changing its shape. That constraint
belongs to issue #60 and to the thermal model record it plans, and it is stated
here because this record is what makes it load bearing.

The exclusion is printed, not only documented. It is one of the entries record
0011 governs, `docs/model-boundary.md` already carries it in the words an
operator reads, and this record adds nothing to those two lists.

What the published record supports, and what it does not.

**Which materials.** Alloys forming an exothermic oxide under a hot boundary
layer, and the case in the recovered debris is aluminium. The 1997 Delta II and
2001 Delta II reentries in `docs/survey/recovered-debris.md` are the source, and
the reading of them is:

    curl -sL -o sdc4-44.pdf https://conference.sdo.esoc.esa.int/proceedings/sdc4/paper/44/SDC4-paper44.pdf
    pdftotext sdc4-44.pdf - | grep -o 'this augmented heating from burning aluminum is generally not included in reentry breakup models.'
    this augmented heating from burning aluminum is generally not included in reentry breakup models.

The same paper reconstructs where that happened:

    pdftotext sdc4-44.pdf - | grep -o 'major breakup occurred at 77.8 and 71.8 km for the objects.'
    major breakup occurred at 77.8 and 71.8 km for the objects.
    pdftotext sdc4-44.pdf - | grep -o 'Maximum temperatures between 1000 and 1300'
    Maximum temperatures between 1000 and 1300

The temperature line ends with a degree sign and a C in the source. The
metallurgy is of AISI 410 stainless steel and Ti-6Al-4V, and the aluminium was
deposited on them rather than being what they were made of.

**The altitude and temperature range where the flux is significant.** Not
answered, and the question as issue #63 poses it does not have an answer in the
implementation that exists. That question assumes a term driven by an atomic
oxygen flux, which would carry an altitude window with it. The ORSAT term quoted
above is driven by the cold wall convective heat rate and a dimensionless
constant, so it has no flux in it and no altitude window to state. No atomic
oxygen flux model and no number density profile was read for this record, and
nothing about one is asserted here.

**Published estimates of the effect on demise altitude.** None was found. What
was found is an estimate of a different shape, and it is stronger for this
decision than an altitude difference would have been. In the 1997 ORSAT analysis
of the two recovered Delta II second stage fragments, the titanium pressurisation
sphere was run at the top of the parameter range:

    pdftotext orsat1997.pdf - | grep -o 'An oxidation heating factor of 1.0 (maximum value) was used in this analysis.'
    An oxidation heating factor of 1.0 (maximum value) was used in this analysis.

and the stainless steel propellant tank was not:

    pdftotext orsat1997.pdf - | grep -o 'An oxidation heating factor of only 0.4 could be used before the cylinder survived.'
    An oxidation heating factor of only 0.4 could be used before the cylinder survived.

That sentence is terse and the reading taken here is stated rather than assumed:
0.4 is the largest factor at which that analysis still produced survival, and
above it the code demised a tank that was recovered on the ground near
Georgetown, Texas. The reading is supported by the same paper concluding that
both fragments were predicted to survive as the actual objects did, and by the
same figure showing the tank sitting at its melt temperature of 1728 K for about
a hundred seconds, which is the state in which a small addition to the heat rate
decides the outcome.

So the cost of the exclusion, quantified: at the top of the one published
parameterisation the term is large enough to move a recovered component from
survives to demises. It is not a small correction to a demise altitude. It is a
parameter with an outcome flip inside its range, and its value for a given alloy
is not measured anywhere this record found.

**Whether the published treatments agree.** They do not, and the disagreement is
about whether to have the term at all rather than about its size. ORSAT carries
it and exposes the efficiency as a parameter an analyst varies. Its smaller
sibling drops it deliberately:

    pdftotext orsat1997.pdf - | grep -o 'currently not in the MORSAT code (to provide for a conservative situation or survivability of the object).'
    currently not in the MORSAT code (to provide for a conservative situation or survivability of the object).

The reference cross comparison of the two standard codes, 120 cases over spheres,
boxes and cylinders in aluminium, titanium and graphite epoxy, switched it off on
both sides to make the comparison possible:

    curl -sL -o sdc4-45.pdf https://conference.sdo.esoc.esa.int/proceedings/sdc4/paper/45/SDC4-paper45.pdf
    pdftotext sdc4-45.pdf - | grep -o 'oxidation heating was not considered'
    oxidation heating was not considered

DEBRISK lists oxidation among the phenomena it models, per record 0011 and the
survey behind it. And the paper on the recovered hardware says the mechanism it
observed is generally not in reentry breakup models at all. Four positions, from
four sources, on one term.

One consequence of the material above is easy to miss and is stated so that it is
not. The strongest physical evidence that oxidation matters is evidence for
deposited molten aluminium burning on the surface of a component made of
something else. A per fragment oxidation term over a fragment's own alloy does
not represent that, however carefully it is calibrated. Record 0003 excludes it
first by flying every fragment alone, and this record excludes it second. Adding
an oxidation term later closes one of those two gaps and not the other.

## Reasons

The direction of the error is known and it is the direction that does not hide a
hazard. Omitting an exothermic surface source gives the body less heat than it
would receive, so the model demises less than reality would and reports more
surviving mass, a larger casualty area and a higher risk number. The one code
that drops the term states that as its reason for dropping it, in the sentence
quoted above. A tool whose output is a casualty expectation is the wrong place to
carry an error whose sign is the other way.

The magnitude is not known, and the measurement that exists argues against
shipping a value rather than for one. A parameter whose range contains an outcome
flip for a real component is a parameter that decides the answer. Choosing a
number inside it and calling the result physics would produce exactly the
plausible number from an unsupported input that record 0010 exists against.

The interface consequence is cheap now and expensive later, so it is taken now. A
heat balance written as a sum of surface sources absorbs an oxidation term as one
more entry. A heat balance written as a convective term minus a radiative term
has to be reopened, and it will be reopened by whoever adds the term rather than
by whoever wrote it.

Two of the four things issue #63 asked for are not in the published record, and
saying so is worth more than filling them. A record that answered all four would
have had to invent an altitude window and a demise altitude difference, and the
next reader would have quoted them.

## Reasons against

This is the entry where the tool departs furthest from the codes it will be
compared against, and record 0011 already says so. ORSAT has carried an oxidation
term since at least 1997. A reviewer who knows the field will read its absence as
the tool being less complete rather than more honest, and on this point they have
a case.

Conservative in the risk direction is not conservative in every direction. An
operator designing for demise is being told their component survives when the
real atmosphere may destroy it, so the tool systematically under credits demise
and pushes the design towards mass and cost it may not need. The word
conservative in this record means only that the casualty number does not come out
too low, and it should not be read as safe in general.

The quantification rests on one sentence in one 1997 conference paper about one
component, read from a scanned copy whose text layer renders one word of a
neighbouring sentence as `tern` rather than `term`. That is a thin base for the
central claim of this record, and the claim is stated no more strongly than the
base supports.

The 0.4 figure bounds a code's tuning parameter and not the physics. Whatever the
true exothermic contribution to that tank was, ORSAT's factor is the number that
made ORSAT reproduce the recovery, and the two are the same only if the rest of
ORSAT was right. A reader who carries 0.4 into this tool as an error bar is
carrying more than the source produced.

Excluding the effect while the operator page says some alloys burn as well as
melt invites the reading that the missing piece is a refinement. On the evidence
above it can be the difference between a fragment on the ground and no fragment
at all, and no wording in a boundary list makes that small.

There is also a case that this record should not exist yet. Its subject is a term
in a heat balance that has not been written, in a thermal model that is not
decided, in a workspace that does not exist. It is written now because issue #63
placed the interface constraint before the implementation deliberately, and
because the literature question is the same question whenever it is asked.

## What would change this

A published sensitivity run over an oxidation efficiency parameter reporting a
demise altitude difference for a stated component. Record 0011 named this as the
measurement that would put a number on its oxidation entry, and this record
looked for it and did not find it. Finding one supersedes this record rather than
extending it, because the decision here rests on the magnitude being unknown.

A measurement of the exothermic contribution for a named alloy from a plasma or
high enthalpy facility, which is a bound on the physics rather than on a code's
tuning parameter and is the thing that would let a term be shipped with a value
somebody can check.

A validation case under issue #78 or #79 whose residual is in the survival
direction, on a component of an alloy this effect is claimed for, and of a size
consistent with a missing exothermic term. Issue #81 is the register that entry
would live in, and issue #77 is what decides whether a residual of that size
counts as a disagreement at all.

A decision under issue #48 on whether the material record carries an oxide heat
of formation. Without that field no oxidation term can be added whatever is
decided here, and with it the cost of reversing this record falls to the term
itself.

Debris recovered from a further reentry that characterises the effect, which is
what the 2005 analysis recommended and has not happened in the corpus behind
`docs/survey/recovered-debris.md`. Issue #78 is where such a case would enter.

## What depends on this

Issue #63, which planned this record. It stays open on the clause this record
does not discharge, which is that the heat balance accepts an additional surface
source term.

Issue #60, the thermal model, which owns the heat balance and therefore owns the
constraint stated in the decision above. If the balance it decides cannot take an
additional surface source, this record is what that decision contradicts.

Record 0011 and `docs/model-boundary.md`, whose oxidation entry this record sits
under. Neither is corrected by it. Record 0011 says no error bound was found and
that remains true, because a bound on a code's parameter is not an error bound on
an answer. What this record adds is the first published figure attached to the
entry and the reason it is not the figure that entry asked for.

Record 0003, which excludes fragment to fragment interaction and therefore
excludes the mechanism the recovered debris shows, independently of this record
and before it.

Issue #48, the material record, which is where an oxide heat of formation would
have to be a field before any later term could read one, and issue #49, which
would have to source it per row.

Issue #62, the demise criterion, which is what an oxidation term would move, and
issue #64, the demise regression set, which is where the movement would show.

## Sources

1. Ailor, W., Hallman, W., Steckel, G., Weaver, M. "Analysis of Reentered Debris
   and Implications for Survivability Modeling." 4th European Conference on Space
   Debris, ESA SP-587, 2005.
   <https://conference.sdo.esoc.esa.int/proceedings/sdc4/paper/44/SDC4-paper44.pdf>
2. Rochelle, Wm. C., Kinsey, R. E., Reid, E. A., Reynolds, R. C., Johnson, N. L.
   "Spacecraft Orbital Debris Reentry: Aerothermal Analysis." Proceedings of the
   Eighth Annual Thermal and Fluids Analysis Workshop, September 1997. NASA NTRS
   19970040121.
   <https://ntrs.nasa.gov/api/citations/19970040121/downloads/19970040121.pdf>
3. Lips, T., Wartemann, V., Koppenwallner, G., Klinkrad, H., Alwes, D.,
   Dobarco-Otero, J., Smith, R. N., DeLaune, R. M., Rochelle, W. C., Johnson,
   N. L. "Comparison of ORSAT and SCARAB Reentry Survival Results." 4th European
   Conference on Space Debris, ESA SP-587, 2005.
   <https://conference.sdo.esoc.esa.int/proceedings/sdc4/paper/45/SDC4-paper45.pdf>

Source 1 is the paper `docs/survey/recovered-debris.md` was read from, and the
Delta II rows there carry the rest of what it supplies. Sources 2 and 3 were read
for this record. The commands above were run against copies fetched from those
URLs on 2026-08-10, and each prints the line quoted beneath it.
