# 0009. The scenario an operator writes, and the artefacts a run leaves behind

## Date

2026-08-08

## Status

accepted

## Question

What does an operator hand this tool, in what format, and what does a run leave
behind afterwards.

Both halves are one question because they are the tool's actual interface. The
physics will be rewritten several times and these two surfaces will outlive every
rewrite, so getting them wrong is expensive in a way that getting a coefficient
wrong is not.

The input is also untrusted. A scenario file arrives from somewhere, often by
email, and it is parsed before anything else happens.

## Options considered

On the input format.

YAML. The most comfortable to hand-write of the candidates and the most widely
used for configuration of this kind. It is rejected on the untrusted input
constraint rather than on taste. Its specification carries anchors, aliases and
type tags, which give a parser reasons to expand, allocate and construct on the
strength of the document's own content, and the well-known amplification attack
against it exists because of that. Its implicit typing is a second problem in a
physics tool specifically: a value that reads as a number in one implementation
and a string in another is exactly the class of defect this project cannot
afford.

JSON. Simple, universally parseable, and the parsers are small. It has no
comments, which matters more here than it usually does, because a scenario is a
record of modelling choices and the reason for a choice belongs next to it. It
is also poor to diff by hand, and an operator keeping a scenario in version
control diffs it constantly.

TOML, taken below. Text, comments, unambiguous typing, a specification small
enough to read in one sitting, no aliases, no tags, no include mechanism and no
construction of arbitrary types. It costs deeply nested structures, which are
awkward, and a component list is exactly the shape that gets awkward.

A binary or a general-purpose serialisation format. Faster to parse and smaller.
Rejected outright by the second constraint: a format whose parser is a
general-purpose object deserialiser turns a scenario file into a program, and the
scenario file is the untrusted input.

A bespoke text format. Total control of the grammar and the smallest possible
parser. It costs every operator an unfamiliar syntax and this project a parser
and its fuzzing harness, for no gain over TOML.

On the versioning rule.

Guess the version from the fields present. Convenient for the operator, and it
makes the tool's behaviour depend on a heuristic that changes silently as the
schema grows.

Accept an unversioned file as the oldest version. Also convenient, and it means
a file written yesterday against the current schema and missing its version line
is read as something else.

Refuse an unversioned file, taken below. Costs the operator one line and one
error message the first time.

On the outputs.

One document containing everything. Easiest to write and easiest to lose track
of, and it forces every consumer to parse the whole thing to read one part.

Named files with documented schemas, taken below.

Whatever the code happens to print, with the format defined by the code. This is
the state most tools of this kind are in and it is why their results get
screen-scraped.

## Decision

The scenario is one TOML file. It describes exactly one reentry and carries the
parent object with its component list, each component's shape, dimensions, mass
and material, the entry state, the epoch, the atmosphere and space weather
selection, the sampling configuration and the seed.

The first key in the file is a schema version, and it is not optional. Three
rules follow from it and there is no fourth.

A file with no schema version is refused, naming the missing key. The tool does
not guess.

A file whose version the tool does not know is refused, and the refusal states
the version it read and the versions it supports. This covers the newer-file
case, which is the one that would otherwise be read as a corrupt file.

A file whose version is older than the current one is refused too, and the
refusal names the command that converts it. Conversion is a separate explicit
command that reads the old file and writes a new one, and it never happens
implicitly during a run. The reason is that an implicit migration changes what
the operator believes they ran without leaving a trace in the file they keep in
version control, and the converted file, being a file, is diffable and
reviewable.

The artefacts are named files in a run's output directory, each with a schema
that ships with the tool. There are five and every run writes all of them.

The fragment outcome table is CSV with a header row. One row per fragment,
carrying the identity and the parent link that record 0005 requires, the
terminal state, and, according to that state, the demise altitude or the impact
point with its velocity and energy. CSV because this is the artefact somebody
opens in a spreadsheet and pastes into a report, and because a table with a
fixed column set is the one shape CSV is actually good at.

The footprint is GeoJSON. A mapping tool has to read it without a conversion
step, and GeoJSON is the format every mapping tool already reads. It is text and
it diffs, badly but readably.

The risk summary is JSON. It carries the expected casualty number, the casualty
area, the counts in each of the four terminal states of record 0005, and the
population basis the number was computed against. It is machine readable because
the thing that most wants to read it is a comparison between two runs.

The manifest is JSON, and it is the artefact record 0008 fixes. This record
places it among the others and does not restate its contents.

The validation standing the tool prints about itself is written as its own file
rather than as a field inside another, so that it cannot be dropped by a
consumer reading only the summary. Issue #81 owns what it says.

Two constraints hold across both halves and they are the reason for every choice
above. Everything a human reads or diffs is text, because a scenario belongs in
an operator's version control and so does a result they intend to defend. And no
format here has a parser that is a general-purpose object deserialiser, in
either direction, because the input is untrusted and because an output format
with that property invites a consumer to become vulnerable through this tool's
artefacts.

Schemas live under `schema/` in this repository, one file per artefact per
version, named so that the version is in the file name. They ship inside the
artefact so that a run can name the schema it wrote against and an offline
reader can resolve it.

## Reasons

The untrusted input constraint decided the input format on its own. Everything
else was a preference between comfortable formats, and comfort loses to a parser
that cannot be talked into constructing something. TOML gives up nested
structure and gives up nothing else that matters here.

Refusing rather than guessing is the same rule the rest of this project already
took. A tool that guesses a version produces an answer nobody can attribute to an
input, and this tool's entire claim is that its answers can be attributed to
inputs. The cost of refusing is one clear error at the start of a run, which is
the cheapest moment in the whole process to be told something is wrong.

Separate named artefacts rather than one document is what makes a result
checkable by somebody who did not run it. The person who wants the footprint on
a map, the person who wants the number, and the person who wants to know which
component caused it are three different people, and handing each of them a file
they can open is the difference between a result that is examined and a result
that is quoted.

Choosing three formats rather than one is deliberate and it is the choice most
likely to be questioned. Each is the format its consumer already reads: a
spreadsheet reads CSV, a map reads GeoJSON, a program reads JSON. A single
format would be tidier for this project and worse for all three.

## Reasons against

TOML for a component list is genuinely awkward. A satellite with fifty
components is fifty array-of-table blocks, and an operator maintaining that by
hand will want an include mechanism or a generator, neither of which this record
gives them. The first serious scenario will make this argument and it is a fair
one.

Refusing an old file rather than migrating it silently costs the operator a step
every time the schema moves, and the schema will move often early on. There is a
real risk that the conversion command becomes something people run without
reading, which would deliver the convenience of an implicit migration along with
the ceremony of an explicit one.

Five files per run is a lot of output for somebody who wanted one number. A
directory of artefacts is heavier than a line on standard output, and it will be
described as bureaucratic by the first person who runs the tool in a loop.

CSV has no types and no schema of its own. Naming a schema for it is a
convention this project maintains, not a property of the format, so a consumer
reading it wrongly is not refused by anything. JSON for the fragment table would
have removed that, at the cost of the one consumer most likely to exist.

Against the whole shape: this is a lot of interface fixed before a single number
has been computed, and some of it will turn out to be wrong in a way that is
expensive to change precisely because it is the interface. The counter is that
the interface is expensive to change whenever it is decided, and deciding it
after the physics means deciding it under pressure.

## What would change this

A component list that TOML cannot express without a generator. That is
checkable as soon as a real spacecraft is described, and if it happened the
answer would be a second input file for the component list rather than a
different format for the whole scenario.

An operator constraint requiring an existing exchange format, because the
scenario has to be produced by something that already exists. That is a forced
means, and it would be held to the smallest surface: an importer rather than a
change to this record.

A footprint that GeoJSON cannot represent. A footprint is a polygon or a set of
points and this is unlikely, but a gridded probability surface is not a GeoJSON
shape, and if the footprint becomes one, the footprint artefact gains a second
form rather than changing its first.

A measurement showing that writing five artefacts is a material fraction of the
cost of a cheap run. Nothing is measured. If it were true the answer is a faster
writer, not fewer artefacts.

## What depends on this

Record 0008, which fixes the manifest and is placed among the other artefacts
here.

Record 0005, whose fragment identity and parent link the fragment outcome table
is required to carry.

Issue #86, the parser fuzzing, whose target is the scenario reader chosen here.

Issue #92, the command surface, which owns the conversion command this record
requires to exist.

Issue #95, the report an operator reads, which is built from these artefacts and
not from a parallel output path.

Issue #81, the validation standing, which is written as its own artefact by this
record.

Issue #21, the refusal rule, which shares the position that a missing or unknown
input is refused rather than defaulted.
