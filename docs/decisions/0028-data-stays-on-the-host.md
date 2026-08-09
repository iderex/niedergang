# 0028. Data stays on the host

## Date

2026-08-09

## Status

accepted

## Question

What may leave the machine this tool runs on, and where could data about people
enter it.

Both halves are settled before the code because both are cheap now and expensive
later. A network call on a calculation path is a call every later run inherits,
and a rule about personal data written after the artefact formats are fixed is a
rule the formats have to be changed to satisfy.

The question is not whether the tool is trustworthy. It is which of its
properties a reader can check without reading the whole source, since the people
who most need to check are the ones least able to audit a numerical core.

## Options considered

Fetch what a run needs when it needs it. Space weather indices, an Earth
orientation file, a population grid, all pulled on demand. Convenient, and it is
what a tool grows into if nobody decides otherwise. It costs reproducibility
first: the answer then depends on which server was reachable and on what it
returned that day, and record 0008 cannot record a provenance for a byte stream
nobody kept. It costs the operator second, because a machine on an isolated
network runs a different tool from a machine that is not.

Fetch on demand with a cache and an offline flag. The same shape with the failure
moved rather than removed. A cache is a fetch that already happened, so the first
run on a new machine is still a network run, and a flag that turns the fetching
off is a flag somebody has to know exists.

No network on the calculation path, with one separate command that fetches. Two
sources of input have to come from somewhere and record 0006 and record 0004
already place both behind an explicit command. Every other input is a file the
operator supplies. This costs an operator a step before a realistic run, and it
costs a refusal at the moment a model that needs indices is selected without
them.

For telemetry, three positions. Collect nothing ever. Collect behind a build flag
that ships off, which leaves a reader having to verify the absence in each build
rather than once. Collect with an opt-out, which puts the work on the person the
rule exists to protect.

For federation, meaning an exchange of scenarios, results or case data with
another installation, three positions. Never, which is a promise about a version
nobody has designed. Implicit, where an installation that knows about another
uses it, which is the shape that produces a leak nobody chose. Deliberate, off by
default, for a destination the operator names, with the documentation saying what
would leave before anything does.

## Decision

The calculation path makes no network request. Not for indices, not for a grid,
not for a schema, not for a version check.

The tool sends no telemetry, no usage report and no crash report, in any build.
There is no build flag, no environment variable and no configuration key that
turns any of that on, so the absence is a property of the program rather than of
how it was configured.

Anything the tool fetches is fetched by a separate command that does nothing
else. Two such inputs exist today, the space weather file of record 0006 and the
Earth orientation parameter file of record 0004, and both records already fix
that the fetch is explicit, never on a calculation path, never on a first run,
and never a side effect of a command whose name is about something else. That
command sends the request and nothing else: no scenario, no result, no
identifier, no count of anything the operator has done.

When an input a selected model needs is absent, the run is refused and the
refusal names the command that fetches it. Record 0006 fixes that for the
indices and this record does not weaken it. A refusal is the behaviour, not a
fetch and not a substituted default.

Where personal data could enter is named rather than dismissed.

The population grids are aggregated counts per cell. The survey in
`docs/survey/population-data.md` records, for each candidate, what its producer
says its own inputs are, and none of them describes data about an identifiable
person. That is a negative finding taken from the sources rather than an
assessment of the censuses underneath them, and this record points at it instead
of restating it.

An operator's scenario can name a satellite, an organisation and sometimes a
person. It is a file on the operator's machine, it is read from there, and
nothing in the tool copies it anywhere the operator did not name.

The output describes where people are. A footprint over inhabited ground is data
about places built from data about places, and it still says which places have
people under them. It is written to the run's output directory and to nowhere
else, and it is treated with the care the second sentence earns rather than the
care the first one would allow.

Federation is deliberate, off by default, for a destination the operator names,
and the documentation says what would leave and where it would go before
anything goes. It is never implicit in an update and never a side effect of a
command whose name is about something else.

Nothing in this repository refuses any of the above today, and that is the state
of the rule rather than a note about it. The tracked tree holds no source for a
check to read:

    git ls-tree -r --name-only origin/main | grep -cE '\.rs$|Cargo\.toml|rust-toolchain'
    0

and the invariants gate prints the network invariant among the ones it does not
carry, with what it waits on:

    git grep -n 'no network call outside the separate fetch command' origin/main -- .github/workflows/invariants.yml
    origin/main:.github/workflows/invariants.yml:202:            no network call outside the separate fetch command   waits on #24 and on source existing

Both at `e84095215431d40b9a7eff5932cf4ed6f71ff580`. Issue #35 is where the
refusal is owed, and record 0013 is what makes it a reading of one directory
rather than of the whole tree, since it places every call that could open a
socket in `data`.

## Reasons

A reentry risk number is produced by one party and read by another, and the
second party cannot repeat the run. What they can do is check that the run could
not have depended on anything outside the machine it happened on. That check is
worth more than any assurance, and it only exists if the property is absolute.
An exception for a version check is an exception for everything, because the
reader then has to establish which requests are the allowed ones.

Reproducibility is the same argument in a different register. Record 0008 fixes
that a run records what it was, and an input fetched during the run is an input
whose bytes nobody kept. The separate command makes the fetched file an artefact
on disk with a date, which is a thing a manifest can name.

Telemetry has no version of this that survives the standpoint. A crash report
from a tool that reads a scenario naming an organisation carries that scenario's
shape whether or not it carries its contents, and the operator who would most
want a defect fixed is the one least able to authorise sending it.

Naming the personal data question rather than answering it with a sentence about
aggregation is the difference between a statement a reader can check and one they
have to accept. The grids are the easy part and the survey already carries the
sources. The scenario file and the footprint are the parts that would be missed,
and they are the ones an operator can act on.

Federation is decided now because the expensive version of it is the one that
arrives as a default in an update. Deciding it while nothing federates costs a
paragraph.

## Reasons against

The rule is prose. The section above says so with the commands, and a rule
nothing refuses is a rule that survives exactly as long as everybody remembers
it. The honest form of the objection is that this record is worth having only if
#35 gets its invariant, and until then a reader who takes it as enforcement has
been misled by its confidence rather than by its content.

The refusal costs an operator at the worst moment. Somebody selecting a model
that needs indices for the first time meets a refusal instead of a result, and
record 0006 already records that as the cost of its own position. Doing it again
here compounds it: the same person also cannot be told about an updated file,
because nothing checks for one.

No telemetry means no crash reports, which means a defect that only appears on an
operator's machine is one only that operator can report, through a route they
have to find. For a tool whose failure mode is a confident wrong number, giving
up the channel that would surface such a failure is a real loss and not a
formality.

The federation position is a promise about work nobody has scoped. A design that
does not exist cannot be held to a default, and writing the default down now may
produce a record that the eventual design has to be argued around rather than
one that shapes it.

Naming the scenario file and the footprint as places personal data touches
invites the reading that this tool processes personal data, which is a stronger
statement than the one made here and is not the one intended. The wording is
chosen against that reading and a reader in a hurry may still take it.

## What would change this

The invariant landing in the gate. That is the condition under which this record
stops being prose, and it is the only change that would alter what the record is
worth rather than what it says. It is held by issue #35 and it needs source to
read, which is issue #26.

A legal obligation to report something outward. Nothing read for this record
imposes one, and that is a negative finding about what was read rather than a
survey of every jurisdiction an operator might sit in. An obligation of that kind
would supersede this record rather than be handled as an exception inside it.

A measured cost to the separate fetch that outweighs what it buys. No number is
quoted, because there is nothing to measure yet.

A population grid that turns out to be personal data under some reading of its
terms. The survey records what each producer says about its own inputs, which is
weaker than an assessment of the censuses underneath, and a finding against one
of them would change entry 5 of issue #1 and this record with it.

## What depends on this

Issue #24, which this record answers, and whose remaining clause is the
invariant.

Issue #88, the data handling statement in the words an operator reads. It carries
this rule outward and marks each part as a claim for as long as it is one.

Issue #35, which owes the refusal, and issue #26, which owes the tree that
refusal reads.

Issue #30, the separated network suite, which is where the fetch command is
tested and where a test that reaches the network is kept out of the default run.

Records 0004 and 0006, each of which fixes a fetch this record generalises, and
record 0008, whose manifest is what makes a fetched file into recorded
provenance. Record 0009 places the artefacts this record says are written to the
run's output directory and nowhere else. Record 0013 places every call that could
open a socket in `data`, which is what makes the check a reading of one
directory.

Entry 5 of issue #1, which chooses the population grid, and entry 6, which
decides what the project says about dual use. Neither is decided by this record.
