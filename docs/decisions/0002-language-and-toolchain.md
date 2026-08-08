# 0002. The implementation language and toolchain

## Date

2026-08-08

## Status

accepted

## Question

What is this tool written in, and how is the toolchain that builds it fixed, so
that the three promises this project rests on can be kept.

The promises are the question, not the language. A run has to give the same
number on a different machine years later. A claim the tool makes has to be
provable by a test that a machine refuses to pass when the claim is false. An
operator with no administrator rights on their own workstation has to be able to
run the result.

What this record does not answer is what agreement across machines means. That
is issue #25, and the choice made here constrains it rather than settling it.

## Options considered

C++. The language most of the reference literature is written in, the deepest
numerical ecosystem of any candidate, and the one where an aerothermodynamicist
is most likely to be able to read the source. It costs the two properties this
project needs most. Reproducibility is a per-build-system discipline rather than
a language property: the standard says nothing about which libm the transcendental
functions come from, optimisation flags routinely relax floating point without
saying so, and a dependency set is whatever the machine happened to have unless a
great deal of work is done to make it otherwise. And the test story is a choice
between frameworks rather than a runner that is simply there, which matters
because every guard in this tree has to be provable by running something.

Fortran. Still the language of the reentry codes this project is measured
against, and genuinely good at the arithmetic. It costs a modern dependency and
build story, a test runner, and any realistic path to a single artefact with a
command line an operator drives. The tool it would produce would be a numerical
core with the surrounding apparatus written in something else anyway, which is
two languages rather than one.

Python. The largest pool of contributors in the intended field, the best
interactive story, and by far the fastest route to a first result. It costs the
artefact. The user this project names, somebody inside an organisation where
installing a language runtime is a procurement question, cannot be handed a
Python program without also handing them an environment problem, and the
freezers that turn one into a binary reintroduce the reproducibility question
they were meant to remove. Monte Carlo over thousands of trajectories in pure
Python is also the wrong shape, so the real proposal is Python around a compiled
core, which is again two languages.

Julia. Built for exactly this arithmetic, with the strongest differential
equation ecosystem of any candidate and a units and dimensions story that is
close to what issue #14 asks for. It costs the artefact in the same way Python
does, it costs a runtime install, and its compilation model makes a
single-command start slower than a compiled binary. The community is also small
enough that a dependency going unmaintained is a live risk over the lifetime a
tool like this needs.

Go. A single static binary, a test runner in the toolchain, a build that is
fast, and the smallest cognitive load of any candidate. It costs generics over
numeric types that are pleasant to use, which is what the unit and frame rule of
issue #14 has to be built from, and its numerical ecosystem is the thinnest
here. Go is a good answer to the packaging question and a weak one to the
arithmetic question.

Rust, taken below. A single static binary with no runtime, a test runner and a
benchmark story in the toolchain, a lock file that is committed by default for a
binary, a toolchain version that is pinned by a file in the tree, and a type
system strong enough that units and frames can be made a compile error rather
than a convention. It costs a thinner numerical library set than C++ or Julia
and a smaller pool of contributors from the intended field than any other
candidate here.

## Decision

The tool is written in Rust.

The toolchain version is pinned in the tree, not in a document and not in a
contributor's environment. Two files carry it. A `rust-toolchain.toml` at the
workspace root names an exact version rather than a channel, so that `cargo
build` in a fresh clone selects that version and no other. A committed
`Cargo.lock` fixes every dependency to an exact version and hash.

The minimum supported version is 1.97.0, which is the version this decision was
taken against:

    rustc --version
    rustc 1.97.0 (2d8144b78 2026-07-07)

That number is the floor rather than a target. Raising it is a change to the
pin, made deliberately, with a reason, and it is what issue #31 exists to catch
when it happens by accident.

On floating point, the position is narrower than "reproducible" and it is stated
now because it is the part that will be argued about later. No fast math
relaxation is used anywhere. Every arithmetic operation the tool performs on
`f64` is an IEEE-754 operation, and any construct that permits reassociation or
contraction is out of scope for this tree. Within one platform and one build of
one commit, the same input produces the same bits.

Across platforms it does not promise bits, and the reason is named rather than
left as a hedge. The elementary functions, meaning the sine, the exponential and
the logarithm that every atmosphere model and every frame transform calls, come
from the platform's own library and are not specified to the last bit by any
standard the platform follows. Two correctly working machines can differ in the
final bit of a logarithm, and a trajectory integration over a few hundred
seconds turns that into a visible difference in an impact point. Nothing about
the language choice removes this; C++, Fortran and Julia inherit exactly the same
problem. What the language choice buys is that everything else, the arithmetic,
the dependency set, the iteration order and the random stream, is pinned, so the
elementary functions are the only remaining suspect. Issue #25 decides what to do
about that one, and the options it inherits are a stated tolerance or a vendored
correctly-rounded implementation, not a promise.

On parallelism, the threading model is chosen to fit the determinism property
already fixed by record 0008 rather than the other way round. Samples are
independent and each draws from a stream derived from the seed and the sample
index, so work may be stolen, reordered and distributed across any number of
threads without moving the answer. Thread count is a performance knob and never
an input.

On the field, the commitment is a documented data interface and a command line
contract, so that somebody working in Fortran, C++ or Python can drive the tool
and read its output without writing Rust. That is a deliverable of issue #20 and
issue #92, not a promise of language bindings, which this record does not make.

## Reasons

The artefact decides it. The intended audience includes people who cannot
install a language runtime without asking somebody, and every candidate except
Rust and Go either hands them a runtime problem or hands them a two-language
build. A tool that a safety engineer cannot run on the workstation they have is
not independent of the agencies whatever its licence says.

The second reason is that the pin is a file rather than a practice. The property
this project needs is that a fresh clone builds the same thing, and in Rust that
is `rust-toolchain.toml` plus `Cargo.lock` sitting in the tree where a reviewer
can see them and a diff shows them moving. In C++ the same property is a build
system, a container, or a lockfile from a package manager that has to be chosen
and then maintained, and the version that actually built a release is
reconstructed rather than read.

The third is that the guard rule and the type system meet. Every rule in this
project is supposed to be refused by a machine rather than described in a
document, and the largest single class of defect a physics tool has, which is
mixing a geodetic altitude with a geopotential one or a body frame vector with
an inertial one, can be made a compile error in Rust at a cost the language is
built to carry. Issue #14 is where that is designed. In Go it would be a
convention plus a linter, and in Python and Fortran it would be a runtime check
at best.

Against the field argument specifically: the pool of people who can extend a
reentry code is small in every language, and the ones who could improve the
heating and ablation models are mostly employed by the agencies this project is
independent of, which is a constraint on contribution that no language choice
removes. Making the data interface and the command line the extension surface is
therefore a better answer than choosing a language on the strength of who might
one day contribute to it.

## Reasons against

The numerical ecosystem is the real cost and it is not small. Adaptive
Runge-Kutta integration with dense output, geodesy, and the interpolation an
atmosphere table needs are all available in C++ and are excellent in Julia. In
Rust some of that will be written here and, more expensively, validated here.
Every routine written rather than taken is a routine this project owns for its
lifetime, and the honest version of the estimate is that the integrator and the
frame transforms will be ours.

The contributor pool is the second cost and it argues against this record more
than anything else does. The people best placed to correct the physics in this
tool write Fortran and Python. Handing them a Rust codebase is handing them a
reason not to. The data interface mitigates this for using the tool and does
nothing for correcting it, and that distinction should be read as a real
limitation rather than a solved problem.

Compile times and the learning curve are the third, and they are the ones that
will be felt daily rather than argued about. A borrow checker fighting a change
to how a fragment holds its material reference is time not spent on the model.

There is also a reasonable case that C++ was rejected too fast. A C++ project
with a pinned container, a package manager with a lock file and a strict warning
set can hold every property claimed above. That it is achievable is not in
dispute. What is claimed is that it has to be built and then defended, where in
Rust it is the default, and a property that has to be defended is one that
erodes when somebody is in a hurry.

## What would change this

A measurement showing the numerical throughput of the Rust integrator is a
material fraction worse than a C++ or Fortran equivalent on a representative
footprint run. Nothing has been measured. The measurement is possible as soon as
issue #40 exists, by running the same case through both, and if the gap were
large the answer would be a faster integrator before it were a different
language.

A dependency that this tool genuinely needs and that exists in no usable form in
Rust, where writing it here would be a larger project than the tool. An
atmosphere model with a licence that only ships as a Fortran library is the
concrete shape this would take, and it is not hypothetical: it is issue #16's
problem, and the answer there may be a forced non-Rust means held to the
smallest surface rather than a change to this record.

A decision under issue #25 that bit identical output within one platform cannot
be held even with everything above pinned, which would make the reproducibility
argument for this choice weaker than it is stated here.

An operator constraint from outside that forbids an unsigned or unfamiliar
binary and requires a package in a specific ecosystem instead. That is a
distribution constraint rather than a language one, and it would be answered
first by issue #96 rather than by rewriting the tool.

## What depends on this

Every issue in milestone 03. The workspace layout of issue #26, the build gate of
issue #27, the format and lint gates of issue #28, the test gate of issue #29,
the separated suites of issue #30, the minimum toolchain build of issue #31, and
the advisory and licence policy gate of issue #32 all take their shape from this
record, and the toolchain pin above is the thing issue #31 checks.

Issue #14, the unit and frame rule, which is written as a type-level rule here
and would have to become a lint or a runtime check under any other candidate.

Issue #25, the numerical contract across machines, which inherits the floating
point position stated above and decides what to do about the elementary
functions.

Issue #71, the sampling implementation, which inherits the statement that thread
count is not an input.

Issue #20 and issue #92, the artefacts and the command surface, which carry the
commitment made here that the extension surface for the field is data and a
command line rather than the language.
