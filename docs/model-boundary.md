# What this tool does not model

Every reentry code leaves things out. This page is the list for this one, in the
words an operator needs rather than the words a developer does. If an effect
matters to your case and it is on this page, the number this tool gives you does
not account for it.

`docs/decisions/0011-model-boundary.md` is the authority. It carries the same
list with the sources behind each entry and with each error estimate marked as
either a citation or a guess. Where this page and that record disagree, that
record is right and this page is stale.

One thing to read first. Most of the entries below say which way the answer is
wrong and not by how much. That is not an oversight in the writing. For most of
these effects no published error bound exists for the kind of case this tool
computes, and a number invented to fill the gap would be worse than the gap.

## The list

Attitude is assumed, not solved. The tool flies a fragment with a tumbling or
fixed attitude rather than working out how it actually turns. For a roughly
spherical fragment this changes little. For a plate or a long cylinder the area
facing the flow varies over a tumble, and so does the heating.

Oxidation is not modelled. Some alloys burn as well as melt, and burning puts
heat in. Leaving it out means the tool gives a fragment less heat than it would
really get, so it predicts that more fragments survive than really would.

Fragments do not shield each other. Every fragment flies alone and sees the
undisturbed flow. Just after a breakup, while pieces are still close together,
one piece can sit in another's wake and stay cooler than this tool says.

Material blown off a fragment does not change the flow around it. In reality
that outflow thickens the boundary layer and reduces the heating. Leaving it out
means the tool gives a fragment more heat than it would really get, which is the
opposite direction from the oxidation entry above.

Structural failure is not computed, so the moment a spacecraft comes apart is
an input. The tool does not work out when the structure fails. You give it a
breakup altitude, or you take the default, and everything after that depends on
that number being about right. The conventional value used across the field
comes from a single sentence in an old licensing document, and the group that
uses it most has published that always using it may bias their results.

Pressure vessels and residual propellant are not modelled as such. A tank is
flown as an ordinary fragment. A tank that in reality burst high and scattered
will be shown here as one intact piece hitting the ground.

Melted material is assumed to leave immediately. Where a melt is viscous
enough to stay on the body it keeps its mass and insulates what is underneath,
and this tool does not represent that.

Heating by radiation from the shock layer is not modelled. This is
deliberate and it is the one omission with a clean justification: it matters at
entry speeds well above the roughly 7.5 to 7.9 kilometres per second of a decay
from low Earth orbit. If you are assessing anything faster, such as a return from
beyond Earth orbit, this tool is the wrong tool.

The shape of the Earth enters only through the stated conventions. Which
altitude and which latitude are meant is fixed elsewhere and matters more than it
sounds: the difference between geodetic and geocentric latitude reaches about
0.19 degrees at mid latitudes, which is roughly 20 kilometres on the ground, and
a footprint is measured in exactly those units.

## What is not on this list

An effect that is neither on this list nor modelled is an oversight rather than a
decision. Reporting one is worth more to this project than most things, and
`CONTRIBUTING.md` says where a report goes.
