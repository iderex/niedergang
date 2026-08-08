# niedergang

Reentry breakup tools are agency-bound: SCARAB and DRAMA at ESA, ORSAT and DAS at NASA, DEBRISK and PAMPERO at CNES. SCARAB development started in 1995, and the group that develops it describes the original goal as covering every part of a reentry in one tool at the level of detail the computing resources of the time allowed. The same group writes of the rules of thumb this field still quotes, the breakup altitude of 78 km among them, that some are based on little evidence and others rely on old data. The reading behind both sentences is [docs/survey/existing-codes.md](docs/survey/existing-codes.md), which also records what it could not establish. This board builds an open object-oriented tool covering fragment geometries, transitional-regime aerodynamics, heating and ablation, the ground dispersion ellipse and the resulting casualty area and population risk, validated against documented reentries from ROSAT and Tiangong-1 to the routine Starlink deorbits. With tens of thousands of satellites in low orbit this is public safety, and it is currently computed exclusively by the parties that launch them.

Planning happens on the issue tracker first. Every decision that shapes
the architecture is written down there with its reasons before the code
that depends on it exists.

See [NOTICE.md](NOTICE.md) for the intended-use notice.
