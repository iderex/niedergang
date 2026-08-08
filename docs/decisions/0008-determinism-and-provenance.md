# 0008. Determinism and what a run records about itself

## Date

2026-08-08

## Status

accepted

## Question

A sampled answer that cannot be re-derived by somebody who was not there when it
ran is an anecdote. Two things have to be decided before any sampling code
exists: what property the tool guarantees about repeating a run, and what a run
has to write down about itself so that a second person can repeat it at all.

The two halves are one question because neither is worth much alone. A run that
is perfectly reproducible but does not record what it read cannot be repeated by
anybody who does not already have the same files. A run that records everything
but draws its samples from a stream that depends on how many threads were free
cannot be repeated by anybody, including the person who ran it.

What this record does not answer is whether two different machines agree. That
is a separate question with a different answer and it is issue #25.

## Options considered

On the random stream.

No determinism property at all, seeding from the operating system entropy
source. Cheapest, and it is what a program does if nobody decides otherwise. It
costs the ability to argue about a result: a disagreement between two runs
cannot be told from a disagreement between two versions of the code, and a
reported number can never be checked, only re-estimated.

One generator, seeded once, drawn from in the order the work happens. This is
deterministic while the work is single threaded and it is the option most likely
to be taken by accident, because it looks correct in every test written before
the sampling is parallelised. It costs the property outright the first time two
samples run at once, and it costs it silently: the run still completes, the
numbers are still plausible, and nothing in the output says the stream order
moved. Recovering determinism afterwards means holding a lock across every draw,
which serialises the part of the tool that most needs not to be.

A stream derived per sample from the seed and the sample index, so that sample
n draws the same numbers whatever else is running. Determinism no longer depends
on thread count, scheduling, work stealing or the order results are collected
in. It costs a stream construction per sample, and it forbids a convenience the
single generator allows: no part of the code may draw the next number across a
sample boundary, because there is no such thing as the next number.

Golden output files compared run to run, with no property about the stream. This
detects that something changed. It does not reproduce anything, it goes stale
against every deliberate change, and it answers the wrong question, which is not
whether the output moved but whether a stated input produces a stated output.

On what a run records.

Nothing, with provenance left to whatever the operator wrote down. This is the
current state of most reentry results in circulation and it is why arguing about
them is unproductive.

A human readable log. Easy to produce and easy to read, and it cannot be
compared by a machine. A log is a narrative of one run, so checking a second run
against it is a person reading two texts, which is exactly the check that does
not get done.

A machine readable manifest, written by every run without being asked, plus a
mode that takes a manifest and reproduces the run it describes. It costs a
schema that has to be versioned and a code path that has to stay working, and
the code path is the part that decays if nothing exercises it.

## Decision

The random stream is derived per sample from the run seed and the sample index.
One seed reproduces one run exactly, on one machine at one version, whatever the
thread count and whatever order the samples finish in.

Every run writes a manifest, without being asked and without a flag. The
manifest is a machine readable artefact rather than a log, and it carries:

- the tool version and the commit it was built from
- the seed
- the full input after defaults were applied, rather than as the operator wrote
  it
- the version and identity of every data set read, which is at least the
  atmosphere model, the space weather indices, the material library and the
  population grid
- the number of samples
- the wall clock time of the run and the machine it ran on

The input is recorded after defaults because the difference between what an
operator wrote and what the tool ran is where the disagreements will be. A
default that was substituted is visible as a default in the manifest, which is
the same rule issue #21 states from the refusal side.

The tool has a mode that takes a manifest and reproduces the run it describes.
That mode is the thing the determinism property is checked through, so it exists
from the first version that samples anything rather than being added once
somebody asks for it.

Two of the fields above cannot be equal between two runs of the same seed. The
wall clock differs by construction, and the machine differs as soon as anybody
else repeats the run. So the property is not that the manifest files are
identical. It is that the determining fields, which are every field except the
wall clock and the machine, reproduce the same outputs. A reproduction check
that compared whole manifests would fail on its first honest use, and a check
that ignored the difference would have to ignore it by name.

Where a run is not reproducible under some condition, the condition is named,
here and in the output. The words best effort are not used about this property.
The conditions known today are these. A different tool version or a different
commit is a different run and no agreement is promised. A different platform is
issue #25 and this record promises nothing about it. A data set that the
manifest identifies but that the repeating machine does not have is a refusal
rather than a substitution, per issue #21.

## Reasons

The per sample stream is the only option whose property survives the change that
will actually happen. Sampling is the part of this tool that parallelises most
obviously and most profitably, and the single generator option is a property that
holds until the day somebody makes the tool faster and then stops holding without
saying so. A guarantee that a later performance change can silently remove is not
a guarantee.

The manifest is what makes a number arguable. The output of this tool is a risk
figure somebody may act on, and the useful form of a disagreement is not that two
people got different numbers but that two people can find which of their inputs
differed. Recording the input after defaults is the specific thing that makes
that possible, because the most common cause of a disagreement is a default
neither person knew they had accepted.

Writing it without being asked matters more than what it contains. A provenance
artefact behind a flag is absent from every run made in a hurry, which is every
run that later turns out to matter.

The reproduce mode is what stops the manifest from becoming decorative. A record
of what a run read is a claim about what a run read, and the only thing that
tests the claim is a second run driven from it.

## Reasons against

The per sample stream costs real speed on cheap samples. Constructing a stream
per sample is wasted work when the sample itself is short, and the option this
record rejects is faster in exactly the regime where a user runs a very large
number of very cheap samples to settle a tail.

It also constrains the code in a way that will be inconvenient later. Any model
that wants a random draw has to be reached from inside a sample with that
sample's stream in hand, which pushes the stream through interfaces that would
otherwise not carry it. The first model that wants a draw somewhere awkward will
make this argument, and the honest answer is that the constraint is the property.

The manifest field list is a list, and lists in documents go stale against the
thing that writes them. The list above is the decision and the implementation is
what a reader will actually get, so the two can drift. The repair is that the
manifest schema, once it exists, is the authority and this record cites it, which
cannot be done today because it does not exist.

Recording the machine is a small disclosure that somebody may not want in an
artefact they send onward. It is here because a cross machine disagreement cannot
be investigated without it, and issue #24 is where what may be recorded about a
host is decided. If that decision narrows this field, this record is superseded
rather than quietly trimmed.

Against the whole shape: this is a substantial obligation placed on a tool that
does not exist yet, and some of it will look like ceremony until the first
disagreement. The counter is that the first disagreement is the worst moment to
be deciding what should have been recorded.

## What would change this

A measurement showing the per sample stream construction is a material fraction
of the cost of a run. That is checkable as soon as sampling exists, by running a
representative case both ways and comparing, and it is not assumed here because
nothing has been measured. If it turned out to be large, the answer is a cheaper
stream construction rather than a shared generator, and this record would be
superseded only if no cheap construction preserved the property.

A decision under issue #25 that bit identical output cannot be held even within a
machine and a version, which would make the property here weaker than it is
stated and this record wrong rather than incomplete.

A narrowing under issue #24 of what a run may record about its host.

A manifest schema that has to be read by something outside this project, which
would make the field list here a translation of somebody else's format rather
than this project's own decision.

## What depends on this

Issue #25, which decides what agreement means across machines and versions and
which starts from the property stated here rather than restating it.

Issue #71, the sampling implementation, which is where the per sample stream is
built and where the thread count independence is tested.

Issue #21, the refusal rule, which shares the requirement that a default is
visible as a default rather than as an input.

Issue #20, the scenario and the artefacts a run leaves behind, which is where the
manifest sits among the other outputs and where its format is fixed.

Issue #81, the validation standing a run prints about itself, which is a second
thing every run states without being asked and which will share whatever
mechanism writes this one.

Issue #45, the propagation regression set, which contains the check that the same
case run twice gives the same bytes.
