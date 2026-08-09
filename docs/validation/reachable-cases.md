# Which validation cases this tool can reach, and what each one proves

This document sorts every case in `docs/survey/recovered-debris.md` into one of
three tiers and says, per tier, what a pass proves and what it does not. It is
written before the code exists on purpose. The standing a tool claims is easiest
to state honestly while there is nothing yet to defend.

The tiers are about what can be checked, not about how interesting a case is.

## Tier 1: recovered, identified debris and a public component list of the parent

Both halves of the chain are checkable. The tool is given the parent's
components and predicts what survives and where; the ground says what survived
and where it landed.

| Case | Why it is here |
| --- | --- |
| Delta II second stage, reentry 1997-01-22 | Four recovered items with published impact coordinates, and the parts that matter published with dimensions, wall thickness, material and mass. The trajectory was reconstructed from tracking data rather than assumed |
| Delta II third stage, Star 48B, reentry 2001-01-12 | One recovered item, 67 kg, with published dimensions, thickness and material, and a reconstructed trajectory with a state vector at reentry and at breakup |

That is the whole tier, and it is the finding this document exists to state. End
to end validation of this tool is possible against two upper stages of one
launch vehicle family, both of them simple, both of them mostly empty metal
shells, and neither of them a spacecraft.

What a pass in tier 1 proves. That the tool, given a small metal body of known
material entering at a known state, gets the surviving mass and the impact point
close to what was recovered. That is the tool's core loop working end to end on
a real case.

What it does not prove. Nothing about a spacecraft. A satellite is a box of
boxes with electronics, batteries, composite panels, tanks and reaction wheels
inside it, and the two cases above contain none of that. It also proves nothing
about breakup, because a stage that stays largely intact never tested the
fragment tree. And with two cases there is no statistics: agreement could be two
lucky cancellations of errors, and the tool cannot tell the difference.

## Tier 2: one half of the chain is checkable

Either debris was recovered but the parent's component list is not public, so
the tool cannot be given the case as an input, or the reentry was observed with
nothing recovered, so there is no ground truth to compare a footprint against.

| Case | Which half is checkable | Which half is not |
| --- | --- | --- |
| Kosmos 954, 1978-01-24 | That massive, dense reactor components survive to the ground, and roughly where along the track | The input. No component list exists publicly, and the recovered fraction was a sample of an incomplete search |
| Skylab, 1979-07-11 | The gross scale of the footprint, and that large tanks survive | The input, and the positions, which are accounts rather than survey data |
| Columbia, 2003-02-01 | The post-breakup half: fragment count, surviving mass fraction, and footprint length and width, at a scale no other case reaches | The entry, which was controlled and off nominal rather than an uncontrolled decay, and the input, since no pre-flight component list with masses and materials is published |
| Long March 5B core stage, 2020-05-11 and 2022-07-30 | That large stage structures reach populated ground, and the approximate along-track position | The input, the fragment identities, and any material data |
| Dragon trunk, reentries 2022-07-09, 2024-02 and 2024-05 | That a composite structure of this class survives, repeatedly, and that models predicting full burn-up were wrong | The input, and any quantitative surviving mass |
| Flight support equipment pallet stanchion, 2024-03-08 | The demise question for one identified part: Inconel, 0.7 kg, about 10 cm by 4 cm, which survived when the prediction said it would not | Everything else on the pallet, and the footprint, since only the one piece that hit a building was found |
| ATV-1 Jules Verne, 2008-09-29 | Breakup altitudes and the sequence of separation, measured optically | What reached the sea, which nobody looked for |
| UARS, 2011-09-24 | The entry point and time against a prediction | The predicted 26 surviving components and about 540 kg, which went into open water and were never checked |
| ROSAT, 2011-10-23 | The entry point and time | Everything after it |
| Tiangong-1, 2018-04-02 | The entry point and time, against a prediction campaign that published a window beforehand | Survival, footprint and the input, none of which are public |
| Instrumented reentries carrying a breakup recorder, from 2011 | Conditions through the breakup as measured from inside it, where the device returned data | Ground truth, since these are ocean reentries, and the input, which is not published |

What a pass in tier 2 proves. Exactly the half named in the row and nothing on
the other side of it. A tool that reproduces the ATV-1 breakup altitudes has a
breakup model that is not obviously wrong for a large pressurised cargo vehicle.
A tool that survives a stanchion of Inconel where the accepted models demised it
has one data point that its heating and demise path is not making the same
mistake.

What it does not prove. That the numbers an operator actually wants, surviving
mass and casualty area, are right. In every tier 2 row the unchecked half
contains at least one of those, and a document that reports a tier 2 pass
without naming the unchecked half is reporting an end to end validation it does
not have.

The pallet stanchion row deserves its own sentence, because it is the only case
in this document where a published model was checked against the ground and lost
in a direction that matters. Anything that reproduces that failure is worth more
than several cases that agreed, and any tool claiming better demise physics
should be asked this case first.

## Tier 3: cases that exist only as another code's published output

Agreement here is agreement with a model, not with the world. It is still worth
having, because a disagreement is informative and cheap to find, and because a
tool that disagrees with every established code on a benchmark object has
something to explain. It must never be presented as validation against a
reentry, in the README, in the tool's output, or in a report.

This tier is empty today because the search for its cases was run and came back
empty. `docs/survey/existing-codes.md` read the codes' own documentation and the
published papers about them, and it records that validation for SESAM, DAS and
PAMPERO was not found as published case results. The nearest thing it found is a
2024 presentation applying the CNES tools to the January 1997 Delta II second
stage reentry, which it marks as a validation activity rather than a published
result it read. So nothing read for that survey carries one object with its
geometry, materials, masses, entry state and the results a code produced for it,
which is the whole of what a tier 3 row needs.

That is a finding about what this field publishes rather than a note about work
outstanding here, and the two read differently: a tier empty pending work is a
gap that will close, and a tier empty after a search is a statement about the
sources. What the search does not cover is worth naming with it. It read the
codes' own documentation and papers about them, and the reentry conference
proceedings were not searched for a reproducible case specifically. If such a
case is found or published, its object and the demise altitudes and casualty
areas a code computed for it enter here and this section names them.

What a pass in tier 3 would prove. That this tool sits inside the spread of
codes already in use, so a user who has to justify a number can say it is not an
outlier.

What it would not prove. Anything about reality. The codes it agrees with share
assumptions, most visibly the convention of a single breakup altitude near
78 km, and a tool that agrees with them inherits those assumptions rather than
testing them. Where the codes agree with each other and disagree with a
recovered fragment, the fragment wins.

## The sentence the README will carry

Drafted here so that it is fixed before there is pressure to soften it, and so
that the pull request that puts it in the README is a move of an agreed sentence
rather than a new argument.

> This tool is checked end to end against two recovered upper stages, whose
> construction is public and whose debris was recovered with published impact
> points. Every other case reaches only half the chain: either the debris was
> recovered and the object's component list is not public, or the reentry was
> observed and nothing was recovered. No public case validates this tool against
> the reentry of a complete spacecraft with a published component inventory,
> because no such case exists. Read `docs/validation/reachable-cases.md` before
> quoting a number from this tool, and do not present agreement with another
> code as validation against a reentry.

## What would move a case between tiers

A component list becoming public for any parent in tier 2 moves that case to
tier 1 and is the single most valuable thing that could happen to this tool's
standing.

A recovery from a reentry that was also observed instrumentally would create a
case stronger than anything in tier 1, and none exists publicly today.

A case that is quietly promoted, from an inventory that is not public but was
seen, is a different tool with different standing. Whether such a case may be
used at all is entry 7 of issue #1 and is not decided here.
