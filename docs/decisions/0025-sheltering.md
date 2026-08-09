# 0025. Sheltering, and the switch it sits behind

## Date

2026-08-09

## Status

accepted

## Question

The casualty area this tool computes treats every person as standing in the
open. Most people are indoors most of the time and a roof stops some fragments
and not others, so the unsheltered number is known to overstate the risk.

What has to be decided is whether the tool applies a sheltering credit, which
number it reports when it can do both, and what it requires before it will
produce a sheltered figure at all.

What is not decided here is a penetration criterion or a building stock data
set. Neither is chosen, and the record says why rather than choosing one badly.

## Options considered

No sheltering, ever, with no path to it. Simplest, and it is the position the
NASA standard takes for itself. `docs/survey/standards.md` records that the
standard states about 80 per cent of the world's population is unprotected or in
lightly sheltered structures and then declines to use that figure, treating
every person above the energy threshold as exposed. Taking the same position
costs the ability to express a question an operator legitimately has, and it
forecloses a later model that does have data behind it.

Sheltering on by default, with a model built in. It produces the number closer
to what actually happens, when the model is right. It costs the property that
makes the output comparable: `docs/survey/standards.md` records that a tool
applying sheltering by default produces a figure that cannot be set beside a
NASA compliance number, however defensible the model is. It also moves every
risk number this tool prints downward, which is the direction a reader is least
equipped to be sceptical about.

A constant factor taken from the 80 per cent sentence. It is arithmetically
trivial and it inverts its own source. That figure appears in the standard as
the reason for taking no credit, not as a factor to apply, so a tool that
multiplies by it is citing a document for the opposite of what the document
says.

A per-fragment penetration criterion applied wherever building data exists, and
the unsheltered number applied elsewhere. This is the physically honest form and
it produces a number whose meaning changes across the footprint, so two runs
over different oceans and coastlines are not comparable with each other either.

The unsheltered number as the default and always reported, with a sheltered
figure available only as an addition and only against named inputs. Taken below.

## Decision

The unsheltered number is what this tool computes and it is always reported. It
is never replaced, never adjusted, and never absent from an output that carries
a risk number.

A sheltered figure is opt-in and additional. Where it is produced, both numbers
appear together, each labelled with what it is, and the unsheltered one is the
one the tool presents as its answer. There is no mode in which the sheltered
figure appears alone.

Producing a sheltered figure requires two inputs, both named by the operator and
neither defaultable.

A penetration criterion, identified by name and version, which decides whether a
fragment of a given mass, shape and impact energy passes a given roof
construction.

A structure data set, identified by name, version, coverage and reference year,
which says what is under the footprint.

Both are recorded in the manifest under record 0008, alongside the population
basis, and both are printed with the sheltered figure. A sheltered number whose
criterion and data set are not visible next to it is not one this tool emits.

The absence of either, when a sheltered figure is asked for, is a pre-flight
refusal under record 0010 rather than a default. It is knowable from the
scenario and the data sets before anything is flown, which is where record 0010
puts the line. Asking for sheltering and receiving the unsheltered number with a
warning is exactly the substitution record 0010 refuses, so the run stops
instead.

Neither input ships. This tool carries no penetration criterion and no structure
data today, so the switch exists and refuses everything until something is put
behind it. That is the state this record describes rather than a gap in it: the
alternative was a criterion nobody sourced, applied to data nobody has, lowering
every number in the output.

The structure data is a second third-party data question and it is not the
population grid. Entry 5 of issue #1 covers the grid and says nothing about
building stock, its licence, its coverage or its reference year. Nothing on this
board has been asked about that, and this record does not answer it. What it
does is keep the tool honest until somebody is: the switch is defined, the
inputs it needs are named, and nothing is computed from data that does not
exist.

What the tool says about itself. A run that produced only the unsheltered number
says so in the output rather than leaving the reader to infer it from a missing
field, because a number with no label reads as the complete answer. The wording
belongs with issue #81, which owns the standing the tool prints, and this record
requires that it says the sheltering credit was not applied rather than being
silent.

## Reasons

Comparability is the first reason and it is external. The one standard
`docs/survey/standards.md` reads on this point takes no sheltering credit
deliberately, so a default that applied one would produce a figure nobody can
set beside a compliance number. A tool whose output cannot be compared against
the thing operators are actually judged by has lost most of its use.

The direction of the error decides the default. Sheltering can only lower the
risk number. An error in a number that lowers risk is the one nobody
investigates, and this project exists to avoid a plausible figure that reads
exactly like an earned one. Where the tool must be wrong, it is wrong in the
direction that gets checked.

The data is thin and the tool cannot make it thicker. Published penetration
criteria come mostly from the same small set of tests, and building stock data
at a resolution a footprint could use is thinner still. A model that looks
precise on top of that produces a number whose apparent precision is entirely in
the arithmetic.

Refusing rather than defaulting follows from the rest of the tree. Record 0010
makes a missing input a refusal, and a sheltering switch that quietly returned
the unsheltered number when its data was missing would be a default wearing the
name of an option.

The switch is defined now rather than later because the shape of the output is
what the switch changes. A sheltered figure that arrives after the risk summary
schema exists arrives as a second number nobody planned a place for, and the
first thing somebody does with an unplanned second number is put it where the
first one was.

## Reasons against

The default is known to be wrong. Most people are indoors and the unsheltered
number overstates the risk for every footprint over inhabited land. A record
that chooses the number it knows to be too high, and says the reason is that the
error is easier to see, is making a presentational argument about a physical
quantity.

An operator in a jurisdiction that accepts a sheltering credit gets nothing from
this tool for that purpose, and will compute it outside the tool with worse
inputs and no manifest. Refusing to help is not the same as the help not
happening.

A switch with nothing behind it is a promise. It will read on the command
surface as a feature, and every operator who finds it and gets a refusal has
learned that the tool advertises what it cannot do.

Requiring the operator to name a structure data set makes the sheltered path
available in practice only to somebody who already has proprietary building
data, which is the class of user least in need of an open tool.

This record decides against a built-in model without having read the penetration
literature. The thinness of that literature is taken from
`docs/survey/standards.md` and from the issue that planned this record rather
than from the papers themselves, and if the criteria are better than that
suggests, the argument above is weaker than it reads.

## What would change this

A published penetration criterion with test data behind it, readable and
citable, that says for a fragment of a given mass, shape and energy what a
named roof construction stops. That is the input this record says does not
exist, and finding it changes what the switch can carry rather than the default.

An openly licensed building stock data set with stated coverage and a reference
year, at a resolution a footprint crosses. Together with the criterion above, it
would make the sheltered figure something the tool could ship rather than only
accept.

A standard that requires a sheltering model rather than declining to apply one.
`docs/survey/standards.md` records that the ESA requirements have the Earth
population model declared as an assumption without fixing one, and that ISO
24113 and ESSB-ST-U-004 were not read. Any of the three could carry something
that contradicts this record.

A measurement of how far apart the two numbers are on a real case. Nothing here
quantifies what the sheltering credit is worth, and a case where it moves the
answer across a threshold is a different argument from one where it does not.

An answer to the structure data licensing question that entry 5 of issue #1 does
not cover, which would replace this record's statement that nothing ships with a
statement about what does.

## What depends on this

The casualty area of issue #72 and the expected casualty number of issue #74,
which are the quantities a sheltering credit would modify and which this record
leaves unmodified.

Issue #73, the population exposure, whose grid identity, projection and
reference year the structure data set would sit alongside rather than inside.

Record 0009, whose risk summary is where a second number and its labels would
have to fit, and whose schema under `schema/` decides whether a sheltered figure
can appear without its criterion and data set.

Record 0008, which carries the criterion and the structure data set identity in
the manifest when one is used.

Record 0010, which owns the refusal when a sheltered figure is asked for and its
inputs are absent.

Issue #81, the standing the tool prints about itself, which is where the run
says the credit was not applied.

Issue #95, the report an operator reads, which is where two numbers become one
sentence and where the unsheltered one has to survive that step.

Entry 5 and entry 9 of issue #1. The first is the population grid, which this
record distinguishes from structure data. The second is what the output says
about thresholds, which decides what a reader is invited to compare either
number against.
