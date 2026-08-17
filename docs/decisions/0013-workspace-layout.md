# 0013. Workspace layout, the dependency direction, and the text attributes

## Date

2026-08-08

## Status

accepted

## Question

How is this tool divided into parts, which part may call which, and what may
version control rewrite on the way in.

The two halves are in one record because they share a deadline and no
subject. Both have to be settled before the thing they govern exists. A
dependency direction agreed after the code is written is a refactor, and a text
attribute agreed after the first data file lands is a fixture that already
changed.

## Options considered

One crate. Everything in one library with modules for the parts. Cheapest to
start, and it is what this repository would produce by default. It costs the
rule: a module boundary in Rust is not a dependency boundary, `pub(crate)` makes
every module reachable from every other, and nothing refuses a physics core that
reaches into file parsing. The layering would exist as an intention and the
compiler would not hold it.

A crate per file type. A parsers crate, a maths crate, a types crate. It divides
along how the code is written and not along what it is about, so a change to
the atmosphere touches the maths crate, the types crate and the parsers crate,
and every reviewer of that change is reviewing a third of it.

A crate per physics layer, with a one-way dependency order. The split the layers
already have in the model, so a change to the aerodynamics is a change inside one
crate. It costs a workspace to set up, a name per crate, and the discipline to
put a new type in the lowest crate that needs it.

For the text attributes, three options. No `.gitattributes`, which leaves the
behaviour to each clone's `core.autocrlf` and makes the bytes in a working tree a
property of the machine. A blanket `* text=auto`, which lets git guess whether a
file is text and normalise it, and whose failure mode is that it guesses right
almost always and silently deletes a carriage return from the fixture that
existed to prove one. An explicit list, with the default set to leave bytes
alone.

## Decision

The tool is a Cargo workspace of six crates. The dependency direction is one way,
in this order, and nothing may depend on anything later in it:

    core -> model -> propagation -> risk -> data -> cli

`core` knows about quantities, frames, fragments and the errors the rest raise.
It depends on nothing in this workspace. The type-level unit and frame rule of
record 0004 lives here, so every crate above it inherits that rule by using these
types.

`model` is the physics of the environment and of a body in it: atmosphere,
aerodynamics, thermal. Each is behind a trait declared here, which is what record
0006 already assumes for the atmosphere and record 0011 assumes for the thermal
model.

`propagation` is the state vector, the integrator and its step control, and the
breakup event that interrupts it. It calls `model` through those traits and never
reaches for a concrete one.

`risk` is what happens after the fragments stop flying: impact points, the
footprint, the casualty area, the population exposure and the number itself. It
declares the trait a population grid satisfies.

`data` reads and writes: the scenario, the material library, the population grid,
the space weather file, the run artefacts. It is the only crate that touches the
filesystem, and it implements the traits declared by `model` and `risk` rather
than declaring its own.

`cli` is the binary. It wires a concrete atmosphere and a concrete population
grid into the layers that took them as traits, and it is the only crate allowed
to print.

Two rules follow and are stated as rules because a check can read them.

Nothing outside `cli` writes to standard output or standard error. Not a
diagnostic, not a progress line, not a warning. A layer that wants to say
something returns it or raises it.

Nothing outside `data` opens a file or makes a network call. Record 0006 already
fixes that the only fetch is a separate command; this places the code that could
make one in a single crate so that the claim is checkable by looking at one
directory.

Both are greppable and not structural, and that is a weakness stated here
and not left for a reader to notice. Cargo refuses the dependency direction
because the direction is what a manifest declares. Nothing in Cargo refuses a
`println!` in `risk`. Issue #35 is where these two become refusals.

For the text attributes, the default is `* -text`, so a path matching no rule is
stored and checked out byte for byte, and the text classes are listed
explicitly. The list is in `.gitattributes` with the reason at each entry, and it
is not repeated here, because a second copy in a document is a copy that can go
stale while still reading as authoritative.

The rule the list expresses is one sentence. Prose, source and configuration are
normalised to LF in the repository and checked out as LF everywhere. Data,
fixtures and published text carried unchanged are never normalised.

## Reasons

The layering follows the physics because that is the axis along which this tool
changes. Every issue on this board from milestone 04 to milestone 09 is a change
to one layer: an atmosphere, a bridging relation, a heat load, a breakup rule, a
population lookup. A split that puts each of those in one crate makes a change
reviewable as one thing.

The one-way order is what makes a test possible. A propagation test that has to
construct a population grid to compile is a test nobody writes, and the layer
that reaches backwards is exactly the layer that gets tested last.

Putting the traits in the consuming crate is the part most
likely to be argued with, so its reason is here. `core` is meant to know about
quantities, frames and fragments and nothing else. A population grid trait in
`core` would put the shape of a risk calculation into the crate the frames live
in, and the next such trait would too, and `core` becomes the place everything is
declared and therefore the place everything depends on. The cost is that `data`
sits high in the order and depends on four crates, which looks upside down for
something called data and is the right way round for something that supplies
implementations.

Making `data` the only crate that touches the filesystem is what turns the
network claim of record 0006 and the data handling statement of issue #88 from
assertions into something a reader can check without reading the whole tree.

The text attribute default is set to `-text` because the two failure directions
are not symmetrical. A prose file that keeps CRLF is untidy. A fixture that
loses a CR is a test that silently stopped testing what it was written for, and
nothing about it looks wrong afterwards.

## Reasons against

Six crates before there is any code is a structure imposed on work that has not
been done, and the honest form of the objection is that some of these crates may
turn out to be one line of `pub use`. The counter is that the boundaries are
cheap to remove and expensive to add, but that is an argument for the direction
of the risk, not proof that six is the right number.

A crate boundary is also a compile-time cost and an API surface. A type that
wants to move from `model` to `core` is a coordinated change across a version
bump, and in a workspace with a single consumer that
ceremony buys nothing on the day it is paid.

The one-way order forces awkwardness somewhere, and here it is forced onto
`data`. A scenario file describes fragments, materials and an entry state, which
are types spread across four crates, so the scenario reader depends on all of
them. Somebody reading the manifest will reasonably ask why the file reader is
the most connected crate in the workspace.

The two rules that Cargo cannot hold are the weakest part of this record. Until
#35 exists they are prose, and prose that reads like enforcement is worse than
prose that admits it is prose. This record says which of the three rules the
compiler refuses, which is the dependency direction, and which two it does not.

On the text attributes, `* -text` means a new source file with an extension
nobody added to the list is treated as binary, and the first sign of that is a
diff that shows the whole file. That is a real annoyance and it is chosen over
the failure it prevents.

## What would change this

A layer that turns out to need something above it. The named candidate is the
breakup model: if a thermally triggered breakup ever has to consult something in
`risk`, the order is wrong and the crate is not.

A measurement that the workspace is slow to build. No number is quoted, because
there is nothing to measure yet, and this is the shape of thing that would change
the crate count.

The arrival of a second binary. The rule that only `cli` prints is written for
one binary, and a second one would make `cli` a library with two thin fronts or
would make the rule name a set.

A fixture format that is genuinely text and genuinely needs to be diffable. The
`-text` default would then be wrong for that one class and the list gains an
entry and the default stays.

## What depends on this

Issue #26, which this record answers.

Issue #35, which owes the checks that refuse the two rules Cargo cannot hold, and
without which this record's Done-when is not met.

Issue #14, whose compile-fail test needs the `core` crate and the quantity types
in it before it can be written.

Records 0004, 0005, 0006 and 0011, each of which names a type or an interface
that this record places in a crate. None of them is changed by this record and
each would have to be re-read if the layering moved.

Issue #29 and issue #30, since a workspace decides what the default test gate
runs over and which suites are separated from it.
