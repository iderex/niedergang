# 0011. The model boundary, and what is deliberately outside it

## Date

2026-08-08

## Status

accepted

## Question

Every reentry code leaves things out. Which effects does this one leave out, on
purpose, and what does each omission cost.

The record is written before the code exists so that a later reader can tell a
deliberate exclusion from an oversight. That distinction is the whole point: an
effect that is absent and named is a stated limitation, and an effect that is
absent and unnamed is a defect nobody has found yet.

## Options considered

Model everything the reference codes model, and say nothing about the rest. This
is what most of the field does. It costs the reader the ability to tell what was
weighed, and it is the state that makes two codes' disagreement unarguable,
because neither says where it stops.

Name the exclusions but not their cost. Cheap and better than nothing. It is
still not usable: a reader who knows that oxidation is excluded and does not know
what that is worth cannot decide whether the answer is fit for their case.

Name each exclusion with an error estimate, and refuse to state an estimate that
is not either a citation or a labelled guess. More work per entry, and the work
is mostly in admitting how few of the numbers are citable.

Name each exclusion with an error estimate and require every one to be a
citation. Rejected because it would produce a short and dishonest list. Most of
these effects have no published error bound for the case this tool computes, and
requiring one would mean quietly dropping the entries that have none, which
inverts the purpose.

## Decision

The list below is the model boundary. Each entry names the effect, the regime
where it matters, an order of magnitude for the error from leaving it out, and
whether that number is a citation or an estimate. An estimate labelled as an
estimate is acceptable here. An unlabelled estimate is not, and it is the one
thing this record forbids.

Nothing in the list is permanent by default. Each entry says whether it is a
permanent exclusion or a later milestone.

Six degree of freedom attitude dynamics. The tool propagates three degrees of
freedom and stands a tumbling or fixed attitude assumption in place of solving
the attitude equations. Matters most for elongated bodies and plates, where the
projected area between two attitudes differs by the aspect ratio of the body, and
least for spheres, where it does not matter at all. The reference object-oriented
codes do the same thing: attitude "is not directly solved but predefined as
specific motion according to object shapes", and for a tumbling object the
aerodynamic forces and heating are weighted averages over non-tumbling models
[1, section 2.1]. Error from leaving it out: for a body with an aspect ratio near
one, small; for a plate or a long cylinder, the projected area can vary by the
full aspect ratio over a tumble cycle, so the instantaneous heat flux varies by
the same factor and the time-averaged heat load by considerably less. The bound
is a citation for the modelling approach and an estimate for the magnitude. Later
milestone, behind the attitude decision of issue #56, not permanent.

Oxidation and other exothermic surface chemistry. Left out, and this is the
entry that departs furthest from the reference codes. ORSAT exposes oxidation
efficiency as a parameter an analyst varies [2], and DEBRISK lists oxidation
among the phenomena it models [3]. Matters for alloys that form an exothermic
oxide under a hot boundary layer, which is where a demise prediction can move
from survives to demises. Error from leaving it out: not established. No number
is quoted here because none was found, and the direction is the only thing that
can be stated, which is that omitting an exothermic term underestimates the heat
into the body and therefore overpredicts survival. Estimate, and a weak one:
direction only. This is issue #63 and it is a later milestone, not permanent.

Fragment to fragment shielding and wake interaction. Left out permanently at
this level of the model. Record 0003 already fixes that each fragment flies alone
and sees the free stream, and this record does not reopen it. Matters
immediately after a breakup, while fragments are still close enough for one to
sit in another's wake, and stops mattering as they disperse. Error from leaving
it out: a shielded fragment receives less heat than the model gives it, so the
model underpredicts survival for the shielded one, for the part of the
trajectory where the shielding lasts. No number. Estimate, direction only.

Ablation products changing the flow. Left out. Mass blown off a hot surface
thickens the boundary layer and reduces the heat transfer to it, which is the
blowing or transpiration effect. Matters where ablation is vigorous, which is the
same window where the demise decision is being made. Error from leaving it out:
the model gives the body more heat than it would receive, so it overpredicts
demise, which is the opposite direction from the oxidation entry above. No number
is quoted. Estimate, direction only. Permanent at this level of fidelity, and a
candidate for a higher fidelity mode rather than for this one.

Structural failure loads. Left out, and its absence is what makes the breakup
altitude a convention rather than a result. Stated for the whole family this tool
belongs to: "The structural analysis model is always omitted in the
object-oriented method, so the breakup event cannot be directly predicted" [1,
section 2.1]. The cost is not an error bar on a heat flux, it is that the moment
the parent comes apart is an input. Error from leaving it out: whatever the
breakup altitude assumption is wrong by. The reference convention of 78 km
descends from one sentence in a launch licensing support document, and the group
that uses it most has published that always using it "may be introducing bias
into standard ORSAT analysis" [4, section 2.3]. Citation for the provenance and
for the bias statement; no number for the magnitude. Permanent for this tool.
Issue #65 is where the convention is fixed and issue #66 is where a thermally
triggered alternative enters.

Propellant and pressure vessel rupture. Left out. A pressure vessel that
bursts is a different physics from a fragment that comes apart under aerodynamic
load, and residual propellant is a chemical energy source the model has no term
for. Matters because these are exactly the components that survive: composite
overwrapped pressure vessels have been reported surviving reentry to the ground,
and the published rule of thumb from the same work is that "the overwrap on
composite-overwrapped pressure vessels (COPV) is not likely to demise if it is
thicker than about 1 mm" [4, section 2.1]. Error from leaving it out: a tank
modelled as an intact primitive that in reality burst high and scattered is a
single high energy impact predicted where several lower energy ones occurred.
Citation for the survival of these components, estimate for the consequence.
Later milestone, not permanent.

Melt layer removal. Left out, in the specific sense that melted material is
assumed to leave the body immediately. Matters for materials whose melt is
viscous enough to be retained, where the retained layer both insulates and keeps
its mass in the impact energy. Error from leaving it out: assuming immediate
removal takes mass and heat capacity out of the body earlier than reality, which
overpredicts demise. Where the melt is retained the same assumption underpredicts
impact mass. No number. Estimate, direction only. The reference field has moved
here: the advanced demise and ablation model added in SCARAB 4 covers six
material classes including composites and ceramics [5], which is the state of the
art this omission is measured against. Later milestone, not permanent.

Radiative heating from the shock layer. Left out, and this is the one
exclusion with a citable bound rather than a direction. Shock layer radiation
becomes significant at entry speeds well above orbital decay. The Tauber-Sutton
correlation used for it in the reference code is stated as valid for velocities
from 10 km/s to 16 km/s [4, section 2.4], and an uncontrolled reentry from low
Earth orbit arrives at roughly 7.5 to 7.9 km/s, which is below that range. So the
omission is defensible for the case this tool computes and would not be for a
lunar or interplanetary return. Citation for the correlation's validity range,
and the inference that orbital decay sits below it is an inference stated as one.
Permanent for the entry regime this tool serves, and the record is wrong the day
somebody points it at a higher speed case.

Non-spherical Earth effects on the trajectory beyond the gravity choice. The
gravity field is issue #41 and this entry is about everything else the shape of
the Earth does. The reference ellipsoid enters the altitude definition, the
atmosphere lookup and the impact point, and record 0004 is where those
conventions are fixed. What is left out here is any coupling beyond that.
Matters over the length of a footprint, which is hundreds of kilometres. Error
from leaving it out: the difference between geodetic and geocentric latitude
reaches about 0.19 degrees at mid latitudes, which is of order 20 km on the
ground, so this is not a small term and the answer is to get the conventions
right rather than to exclude the effect. That number is a property of the WGS84
ellipsoid rather than a measurement of this model. Not an exclusion so much as a
pointer, and it is here because leaving it unstated is how it becomes one.

The list above is the boundary as it stands today. An effect that is not on it
and not modelled is an oversight, and finding one is a defect report against this
record.

The same list, in shorter form and without the citations, is carried in
`docs/model-boundary.md`, which is the operator-facing half of this decision.
Two documents saying the same thing is a drift risk, and it is accepted here for
one reason: an operator who reads only the short form has to see the boundary,
and an operator who never opens `docs/decisions/` is the normal case rather than
the exception. The short form points at this record as the authority.

## Reasons

The reason for having the list at all is that this project's claim to be worth
using is that a reader can check it. A tool whose omissions are undiscoverable
cannot be checked, only trusted, and the codes this one is measured against are
mostly in that state: no source read for the survey of issue #2 states a list of
deliberately excluded effects for the spacecraft-oriented family at all.

The reason for the citation-or-estimate rule is narrower. An error estimate that
looks like a measurement and is a guess is worse than no estimate, because it
gets quoted. Labelling every entry forces the writer to notice how few of these
have published bounds, and that noticing is itself a finding: most of this list
is direction only, and the honest reading of the list is that the model's
uncertainty from what it leaves out is not quantified.

The reason for stating a direction where no magnitude exists is that direction is
usable even when magnitude is not. Two of the entries push toward overpredicting
survival and two push the other way, and knowing that much tells a reader that
the errors do not obviously accumulate in one direction, which is more than a row
of blanks would say.

## Reasons against

A list of exclusions written before any code exists is a list of intentions. The
model that gets built will not match it exactly, and the failure mode is a record
that reads as a specification of the shipped tool while describing a plan. The
repair is that each entry names the issue that owns it, so a divergence has a
place to be argued.

Most entries carry a direction and no magnitude, which is thin. A reader who
wants to know whether the answer is fit for their case gets told that omitting
oxidation overpredicts survival and not by how much, and that is not enough to
decide with. Presenting that as a boundary risks giving the impression of more
rigour than there is.

Publishing the list is also an argument against the tool, and it is the strongest
one available to anybody who wants to dismiss it. Nine named omissions read worse
than silence, and the codes this one competes with do not publish theirs. That
asymmetry is real and this record accepts it.

Against the duplication specifically: two documents carrying the same list is
exactly the drift this project criticises elsewhere. The mitigation is a pointer
rather than a mechanism, and nothing refuses the two going out of step.

## What would change this

A published error bound for any entry currently carrying a direction only. The
oxidation entry is the most valuable one to fix and the most likely to be
fixable, because the reference codes expose an oxidation efficiency parameter,
so a sensitivity run over that parameter in a code that has it would produce a
number this record could cite.

A case outside the entry regime this tool serves. The radiative heating entry is
justified by orbital decay arriving below 10 km/s, and a higher speed case makes
that entry wrong rather than incomplete.

A validation case under milestone 10 that does not reproduce, where the residual
points at one of these entries. That is the intended way for this list to be
corrected, and issue #81 is the register it would be recorded in.

A demise criterion that turns out to depend on a fragment's neighbours, which
would contradict record 0003 before it contradicts this one, and both would be
superseded together.

## What depends on this

Issue #63, oxidation, which owns the second entry and is where it is either
brought inside the boundary or confirmed outside it.

Issue #56, attitude, which owns the first entry.

Issue #65 and issue #66, the breakup convention and its thermally triggered
alternative, which own the consequence of leaving structural failure out.

Issue #62, the demise criterion, and issue #59, the distribution of heat over the
body, which are where the melt removal and blowing entries become code.

Record 0003, which this record depends on rather than the other way round for the
shielding entry.

Issue #81 and the validation milestone, which is where an entry on this list gets
its first real error bound.

`docs/model-boundary.md`, which restates this list for an operator and cites this
record as the authority.

## Sources

1. Wu Ziniu, Hu Ruifeng, Qu Xi, Wang Xiang, Wu Zhe. "Space Debris Reentry
   Analysis Methods and Tools." Chinese Journal of Aeronautics 24 (2011)
   387-395. doi:10.1016/S1000-9361(11)60046-0.
2. NASA Orbital Debris Program Office, ORSAT page.
   <https://orbitaldebris.jsc.nasa.gov/reentry/orsat.html>
3. CNES, DEBRISK page on Connect by CNES.
   <https://www.connectbycnes.fr/en/debrisk>
4. Ostrom, C. et al. "Operational and Technical Updates to the Object Reentry
   Survival Analysis Tool." First International Orbital Debris Conference, 2019.
   NASA NTRS 20190033904.
5. Kanzler, R. et al. "SCARAB 4 - Extension of the high-fidelity re-entry
   break-up simulation software based on new measurement types." Proc. 8th
   European Conference on Space Debris, 2021.

Full entries for all five are in `docs/survey/existing-codes.md`, which is where
they were read.
