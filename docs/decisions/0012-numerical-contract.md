# 0012. The numerical contract, and what reproducible means across machines

## Date

2026-08-08

## Status

accepted

## Question

Two people will run the same scenario on different machines and compare the
answers. What agreement are they entitled to expect, stated as a property a test
can check rather than as a hope, and what does the tool do about the parts of the
arithmetic that no platform guarantees.

Record 0008 fixed reproducibility on one machine at one version. Record 0002
fixed the arithmetic the language performs and named what it does not fix. This
record answers the part both of them deferred, which is what happens when the
machine changes.

Deciding this after the first disagreement means deciding it under pressure, and
the answer would then be whatever makes the disagreement go away.

## Options considered

Bit-identical output on any platform, for the same input, seed and version. The
strongest position and the one that ends every argument about agreement before it
starts. It is achievable, and record 0002 already holds most of what it needs:
no relaxed floating point, no reassociation, a pinned toolchain and a pinned
dependency set. What is left is the elementary functions, which come from the
platform library and are not specified to the last bit, so holding this position
means shipping an implementation of them rather than calling the platform's. It
costs the work of validating that implementation, it costs speed on every
trigonometric and exponential call in the integrator and the atmosphere lookup,
and it costs it before anybody has measured whether the difference matters.

Agreement within a stated tolerance, per output quantity, with no bit-level
promise anywhere. Easiest to hold and the honest form of what most numerical
codes actually deliver. It costs the ability to tell a real disagreement from
noise: once the contract is a tolerance, a change that moves the answer inside
that tolerance is invisible, and a regression suite over a chaotic trajectory
cannot distinguish a refactor that was meant to change nothing from one that
changed something small. The larger cost is where the tolerance comes from. A
tolerance set to the observed spread is not a contract, it is a description of
the current implementation with a promise attached.

Golden outputs compared across a fixed set of platforms in the gate. Detects a
platform disagreement without stating a contract. It answers the wrong question,
in the same way record 0008 says of golden outputs generally, and it goes stale
against every deliberate change.

Bit-identical within a platform, toolchain version and commit as the enforced
property, with a cross-platform tolerance published per quantity and measured
rather than guessed. Taken below.

On the elementary functions specifically.

Call the platform library and publish a tolerance. Cheapest, fastest, and it
makes the platform an input to the answer without saying so in the type of any
function.

Ship a portable implementation of every elementary function the core uses, so
that the same code runs everywhere. This is what buys cross-platform bits. It
costs an accuracy validation of that implementation against a reference, which is
a real piece of work, and it costs throughput in the innermost loop of the
integrator.

Call the platform library through one seam, so that the choice above can be made
later as a change to one module rather than to every call site, and measure
before making it. Taken below.

## Decision

The contract has two halves and only the first is a promise about bits.

The enforced property. For one commit, one toolchain version and one target,
the same scenario and the same seed produce byte-identical artefacts, whatever
the thread count and whatever order the samples finish in. The manifest is
compared on its determining fields, which record 0008 defines as every field
except the wall clock and the machine, and the other artefacts are compared as
bytes.

That is stated so a test can execute it. Run the case twice with different
thread counts and compare; run it a third time through the reproduce-from-
manifest path of record 0008 and compare again. A failure is a defect in this
tool and never a tolerance question.

Byte comparison only means arithmetic if the text a number is written as is
determined by the number. Every artefact that carries a floating point value
writes it in a form that reads back as the same value and is the shortest such
form, so that two runs producing the same bits produce the same characters and
two runs producing different bits cannot produce the same characters. A rounded
display form anywhere in the artefacts would make this property a test of the
formatter.

The cross-platform half. No bit-level agreement is promised across targets. What
is promised is a tolerance per output quantity, published, and derived from a
measurement across the targets the project builds for rather than from the
observed spread of the current implementation.

That measurement does not exist. Nothing is implemented, so no tolerance is
published today, and no number appears in this record. A tolerance written now
would be a guess wearing the clothes of a contract, which is worse than the
absence, and the absence is stated in the tool's own output rather than only
here: until the measurement exists, a run says that cross-platform agreement is
unmeasured. This paragraph is a disclosure and it does not become a promise by
being restated later.

The settings that hold the enforced property. Record 0002 fixes the first two
and this record adds the rest.

No relaxed floating point anywhere, in this tree or in any dependency that
performs arithmetic on the tool's behalf. The language is understood to perform
no reassociation or contraction on ordinary operators, and that is a claim
rather than something this tree checks, because there is no code here for a
check to read. So the rule is written as a rule and not as an inherited
guarantee: no build setting, intrinsic or dependency that permits either is
adopted, which includes a dependency built through a C compiler with fast math
enabled.

The target is named rather than detected. A build tuned to the machine that ran
it makes the build host an input to the answer, so the target specification is
fixed in the tree and a native tuning setting is not used for a released
artefact.

Every reduction has a fixed summation order, written into the code rather than
inherited from whatever order the results arrived in. Summing a per-sample
quantity in sample index order is deterministic; summing it in completion order
is not, and both compile.

No library whose result depends on vector width, thread count or a runtime
dispatch on processor features is used in the numerical path. This is the same
rule as the reduction rule seen from the dependency side.

No arithmetic reads a value out of a container whose iteration order is not
fixed. A sum over a hash map is a sum in an order the standard library is free to
change.

The pins record 0002 already requires, which are the exact toolchain in
`rust-toolchain.toml` and the exact dependency set in `Cargo.lock`, are part of
this contract rather than adjacent to it. A different toolchain version is a
different platform for the purpose of the property above.

The functions whose platform variation was considered. The distinction is not
between simple and complicated functions, it is between the operations
IEEE-754 requires to be correctly rounded and the ones it does not.

Correctly rounded by the standard, and therefore the same on every conforming
platform: addition, subtraction, multiplication, division, square root, the
remainder, the fused multiply-add, and conversions between the binary formats and
between binary and decimal. These carry no cross-platform risk and the tool uses
them freely.

The division above is taken from the standard's own separation between the
operations it requires and the ones it recommends. No clause is cited, because
the text was not opened for this record, and a reader who needs the citation
rather than the division should treat this paragraph as the claim it is.

Not guaranteed by the standard, and therefore the source of every cross-platform
difference this record is about: the sine, cosine and tangent and their inverses,
the two-argument arctangent, the hyperbolic functions and their inverses, the
exponential in base e and base two, the logarithm in base e, two and ten, the
power function with a floating point exponent, the power function with an integer
exponent, the cube root, and the hypotenuse. Record 0002 states that these come
from the platform's own library.

Two of them deserve naming separately because they look safe and are not. The
integer power is not a single operation and is free to be expanded into a
sequence of multiplications whose association a compiler chooses, so it is not
interchangeable with a hand-written product. The fused multiply-add is correctly
rounded where it is genuinely fused, and it is a different answer from a multiply
followed by an add, so using it is a numerical decision and never an
optimisation applied silently.

Whether the tool ships its own. Not now, and the code is written so that it can.
Every call to a function in the second list goes through one module, and no other
module calls one directly. That module is the seam: adopting a portable
implementation later is a change inside it, and issue #35 is where the rule that
nothing bypasses it is refused by a machine rather than remembered.

The condition that changes this is a measurement rather than an opinion. Once
the propagation regression set of issue #45 runs on more than one target, the
spread on each published quantity is measurable, and if it is larger than what a
user of the impact point needs, the answer is the portable implementation and
this decision moves. Deciding it before the measurement would be choosing between
a cost and a risk when neither has a number.

## Reasons

The enforced half is where the value is. Almost every disagreement that will
actually happen is between two runs of the same build: a refactor, a
parallelisation, a rerun of a case somebody is arguing about. A bit-level
property catches all of those exactly, with a test that compares bytes and needs
no judgement, and it is the property that makes a regression suite mean
something.

The cross-platform half is the one that cannot be had cheaply, and the reason to
publish a measured tolerance rather than promise bits is that the measurement is
the useful artefact either way. A number saying how far two targets drift on an
impact point tells an operator something about the trajectory, and it is also
exactly the number needed to decide whether the portable implementation is worth
its cost. Promising bits first would spend that cost before knowing.

Putting the elementary functions behind one seam is the part that keeps the
option open. The alternative is a codebase where the decision would touch every
call site in the integrator, the atmosphere and the frame transforms, at which
point it stops being a decision and becomes a rewrite nobody schedules.

Refusing to publish a tolerance today follows from what a tolerance is for. A
published tolerance is a promise a reader may rely on, and one derived from
nothing is a promise about nothing. Saying it is unmeasured costs the project
some credibility with a reader who wanted a number, and it costs less than a
number that turns out to have been invented.

## Reasons against

The strongest argument against is that bit-identical within a platform is a
property that constrains the implementation for a benefit most users never see.
An operator comparing an answer to a colleague's is usually on a different
machine, which is the half this record does not promise, so the enforced property
is largely a property for the people maintaining the tool rather than for the
people using it.

The fixed summation order is the concrete cost. It forbids the reduction shape a
parallel runtime gives for free and it will be felt on large sampling runs. A
tree reduction with a fixed shape recovers most of it and is more code than a
plain parallel sum, and somebody will propose the plain one.

Not shipping the elementary functions leaves the platform in the answer, and the
seam is a discipline rather than a solution. Between today and the measurement,
two machines can disagree by an unstated amount, and a user who reads the
enforced property quickly may take it for more than it says. The disclosure in
the output is what stands between those two readings, and a disclosure is weaker
than a bound.

There is also a reasonable case that the whole contract is premature. Nothing is
implemented, no case has been run, and this record constrains code that does not
exist on the strength of an argument about what will be argued about later. The
counter is that the constraints are cheap to adopt now and expensive to retrofit,
particularly the summation order and the seam, and that the first disagreement is
the worst moment to be choosing between them.

Against the byte comparison specifically: it couples the numerical contract to
the artefact formats of record 0009, so a change to how a value is written is a
change that reddens a numerical test. That coupling is deliberate and it will
still be annoying.

## What would change this

A measurement of the cross-platform spread on the regression set of issue #45,
which is what turns the unpublished tolerance into a published one and which is
also the condition on shipping the elementary functions. The measurement is
possible as soon as that set runs on more than one target, and it is not assumed
here.

A measurement showing the fixed summation order costs a material fraction of a
large sampling run, which is checkable once issue #71 exists. A large cost would
be answered with a deterministic tree reduction before it were answered by
weakening the property.

A dependency the tool genuinely needs whose result depends on processor features
or thread count, where writing a replacement is a larger project than the tool.
That would force a choice between the property and the dependency, and the choice
belongs in a record rather than in a build file.

A decision under issue #26 or issue #31 that widens the supported targets to
include one whose libraries differ more than the current set, which changes the
size of the unmeasured gap rather than the shape of this record.

## What depends on this

Issue #45, the propagation regression set, which contains both the same-input
comparison that checks the enforced property and the cross-target measurement
that the tolerance waits on.

Issue #35, the greppable invariants gate, which is where the rule that nothing
calls an elementary function outside the seam is refused rather than remembered,
alongside the constants rule it already carries.

Issue #29 and issue #33, the test gate and the coverage report, which run the
comparison and which see the seam as one module rather than as a scattering of
call sites.

Issue #40, the integrator, and issue #42, the atmosphere behind the interface,
which are the two places the elementary functions are called most and where the
seam is felt.

Issue #71, the sampling implementation, which owns the fixed reduction order,
and record 0008, whose determinism property this record extends rather than
restates.

Issue #81, the validation standing a run prints, which carries the sentence
saying cross-platform agreement is unmeasured for as long as it is.

Record 0002, whose floating point position this record inherits, and record 0009,
whose artefact formats the byte comparison depends on.
