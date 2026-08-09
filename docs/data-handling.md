# What this tool reads, what it writes, and what never leaves your machine

This document is for the person who has to answer for the tool rather than for
the person reading its source. It says what the program does with data, and it
says for each statement whether something refuses a violation of it or whether it
is a claim you are being asked to take on trust.

Read the status line under each statement. Today every one of them says the same
thing, and that is the honest state of this repository rather than an oversight.
The rule itself is written down in
[docs/decisions/0028-data-stays-on-the-host.md](decisions/0028-data-stays-on-the-host.md).

## Everything a calculation reads is a file on your machine

You supply the scenario, the material data, the population grid, the space
weather indices and the Earth orientation parameters. The tool reads them from
the paths you give it and writes its artefacts to the output directory you name.

Status: a claim. There is no source in this repository yet, so nothing refuses a
file read from anywhere else. Issue #35 is where that refusal is owed and issue
#26 is the workspace it would read.

## A calculation makes no network request

Not for reference data, not for a schema, not to check for a newer version. If
the machine has no route to a network, a calculation behaves exactly as it does
on one that has.

Status: a claim, and the same two issues hold it. You can check the state of the
enforcement yourself rather than taking this paragraph's word for it. The
invariants gate prints the invariants it does not carry and what each one waits
on:

    git grep -n 'no network call outside the separate fetch command' -- .github/workflows/invariants.yml

## Nothing is sent anywhere about you or your use of the tool

No telemetry, no usage counts, no crash reports, no identifiers. There is no
build flag, no environment variable and no configuration key that turns any of
that on, so this is not something you have to configure correctly.

Status: a claim. The absence of a flag is checkable by reading the tree once
there is a tree to read; today there is none.

## One command fetches, and it is not part of a calculation

Two inputs come from somebody else's server: the space weather file and the Earth
orientation parameter file. Fetching either is a separate command that does
nothing else. It is never run for you, never on a first run, and never as a side
effect of a command whose name is about something else.

That command sends the request and nothing else. No scenario, no result, no
identifier, no count of anything you have done.

If you select a model that needs one of those inputs and it is absent, the run is
refused. The refusal names what is missing, the epoch it was needed for, and the
command that fetches it. It is not filled in with a default, because a substituted
value that does not appear in the output is how a confident wrong number is
produced.

Status: a claim. The refusal on a missing input is fixed in
[docs/decisions/0006-atmosphere.md](decisions/0006-atmosphere.md) and the fetch
route in that record and in
[docs/decisions/0004-units-frames-and-time.md](decisions/0004-units-frames-and-time.md);
neither is code yet.

## Where personal data could enter

This section is the one worth reading twice, because the easy part is the part
usually written and the rest is usually left out.

The population grids are aggregated counts per cell. Every candidate grid was
surveyed and none of the producers describes an input that is data about an
identifiable person: they publish census, register or administrative counts
spread onto cells using layers about buildings, land cover and light. The record
of that, per grid and with its sources, is
[docs/survey/population-data.md](survey/population-data.md). It is what those
producers say about their own inputs rather than an assessment of the censuses
underneath, and it is written that way there.

Your scenario file can name a satellite, an organisation and sometimes a person.
It stays on your machine. Nothing in the tool copies it anywhere you did not
name, and the artefacts a run writes go to the output directory you gave it.

The output describes where people are. A footprint over inhabited ground is built
from data about places and it still tells a reader which places have people under
them. Treat it as you would treat any other document about a populated area near
an incident, and note that this is a statement about what the output is rather
than about what the tool does with it.

Status: claims, with one exception. The survey document exists and can be read
today, so the sentence about the grids is a citation rather than an assertion.
Nothing refuses a copy of your scenario going anywhere, for the reason given
above.

## If a future version ever exchanges anything with another installation

Federation means sending scenarios, results or case data to another installation
of this tool. Nothing does that today.

If a version ever does, it is something you switch on deliberately, for a
destination you name. It is off by default, it is never introduced by an update,
and the documentation says what would leave and where it would go before anything
goes.

Status: a claim about work that does not exist. It is written down now so that
the default is decided before there is a reason to argue for a different one.

## What this document does not tell you

It does not tell you that the tool has been audited, because it has not.

It does not tell you that these properties hold in the code, because there is no
code. Every status line above says which issue owes the refusal, and the
enforcement they name arrives with the workspace rather than with this document.

It does not cover what the number this tool produces means or what standing it
has. That is issue #90 and its own document.
