# 0007. How uncertainty enters, and whether the footprint is sampled or derived

## Date

2026-08-08

## Status

accepted

## Question

A single trajectory produces an impact point, and nobody should believe an impact
point. What this tool produces is a distribution. How that distribution is built
decides the shape of every interface above the trajectory core, so it is decided
before any of them exist.

The second half of the question is the one that is usually skipped: what is
actually uncertain, what distribution does each uncertain thing have, and where
does that distribution come from.

## Options considered

Propagate uncertainty analytically through a linearised model to a dispersion
ellipse. Fast, closed form, and it gives an answer in the time it takes to run
one trajectory. It is wrong in exactly the places reentry is most non-linear.
Near the demise threshold a small change in density decides whether a fragment
exists at all, and a linearisation cannot represent a variable that changes the
number of objects in the answer. An ellipse around an impact point that half the
time has no impact point is not an error bar, it is a different quantity.

Sample the inputs and fly many trajectories. Handles arbitrary distributions and
handles the non-linearity, including the case above, because a sample where the
fragment demised simply has no impact point and the surviving fraction is part
of the answer. It costs computer time, and it makes the answer depend on a random
number stream that then has to be controlled.

Sample, with the analytic form available for a quick answer and labelled as such.
Taken below.

Worst case bounding rather than a distribution. Run the extremes and report the
envelope. Cheap, defensible, and it produces a number nobody can act on, because
the envelope of a reentry footprint under pessimistic assumptions is most of a
hemisphere.

A single trajectory with a stated engineering margin. This is listed so that it
is visibly rejected rather than silently absent. It is what a tool does when
nobody decides, and it produces the confident wrong number this project exists
to avoid.

## Decision

The footprint is sampled. The distribution is built by drawing the uncertain
inputs and flying many trajectories, and the analytic linearised form exists as
a fast approximate mode which is labelled as such in its own output and is never
the default.

The random stream is the one record 0008 fixes: derived per sample from the run
seed and the sample index, so the answer does not depend on thread count or on
the order samples finish in. This record does not restate that property, it
depends on it.

What is uncertain, and what distribution each one has. The list is the decision.
Every entry is marked either measured, meaning the distribution comes from
published data, or assumed, meaning somebody chose it. Today every entry is
assumed, and that is the finding rather than a placeholder.

The entry state. Position and velocity at the interface altitude, from the
operator's orbit and epoch. Assumed. The distribution is whatever the operator's
own orbit determination gives, and where they supply none, a default spread over
the along-track direction, which is the direction a decay epoch is least well
known in. No source. This is the one entry where the operator is likely to have a
real covariance, and the input format has to be able to accept one rather than
only a scalar spread.

The atmosphere density. Assumed. Density is the single most influential input to
a reentry trajectory and the models disagree with each other, which is a stated
uncertainty rather than noise to be averaged away. The distribution used here is
a multiplicative factor on the density profile, drawn per sample and held for the
whole trajectory rather than redrawn per step, because the model error is
correlated in altitude and a per-step draw would average it away and understate
the spread. The width of that factor is an assumption until issue #16 records
what the model comparison actually shows.

The drag coefficient. Assumed. Largest in the transitional regime, where the
value comes from a bridging relation between the free molecular and continuum
limits rather than from either limit, and the choice of bridging relation is
itself an uncertainty. Issue #55 chooses the relation against measurement and
issue #52 fixes where the regime boundaries sit; until they do, the distribution
here is a multiplicative factor with an assumed width.

The heat transfer coefficient. Assumed, and treated as correlated with the drag
coefficient rather than drawn independently, since both come from the same flow
regime treatment and drawing them independently would produce samples where the
aerodynamics and the heating disagree about what regime the fragment is in.

The material properties. Assumed. Melting point, heat of fusion, specific heat,
thermal conductivity and emissivity each carry an uncertainty, and several vary
with temperature over the range that matters. The material record of issue #48
is where a property carries its uncertainty alongside its value and its
provenance, and until that exists this entry has no distribution at all. A
material whose required property is missing is a refusal under issue #21 and
never a sampled guess.

The breakup altitude. Assumed, and this is the entry with the most published
basis of any of them. The reference convention is a single value used regardless
of the parent object, and the group that uses it most has published a scheme that
computes a breakup altitude per parent body from the altitude at which the
radiative equilibrium surface temperature reaches the parent material's melting
point, which makes low ballistic coefficient objects break up higher and dense
ones lower [1, section 2.3]. That scheme gives this project a basis for a
distribution rather than a point, and the distribution is still assumed until
somebody derives it. Issue #65 owns the convention.

The attitude. Assumed. Where the treatment is a tumbling average, the
uncertainty is the spread between the tumbling average and the fixed attitude
extremes, which is bounded by the shape rather than by a measurement. Issue #56
owns the treatment and record 0011 records that six degree of freedom dynamics
are outside the model boundary.

The rule that makes the list above honest rather than decorative: every run
states, in its own output, which of its distributions are assumptions. Not in the
documentation, not in a footnote, in the artefacts a run writes. An uncertainty
budget assembled from guesses is worth more than a single number, and it is worth
that only if it says what it is made of. The risk summary artefact of record
0009 carries this and the validation standing of issue #81 is where its wording
is fixed.

The sample count is an input rather than a constant, it is recorded in the
manifest, and a run reports a convergence measure on the quantity that matters
rather than only the count. A footprint from too few samples is wrong in a way
that looks exactly like a footprint from enough of them.

## Reasons

The non-linearity decides it, and the decisive case is not the shape of the
footprint but the count of objects in it. Near the demise threshold the answer to
whether a fragment exists is a step function of density, and a linearised
propagation cannot cross a step. Any method that assumes the output is a smooth
function of the inputs is answering a different question in the regime this tool
is most often asked about.

Sampling also matches how the uncertainty is actually known. Almost none of the
entries above has a Gaussian with a published width. What several of them have is
a plausible range and a reason, and a sampler can take a range where a covariance
propagation needs a covariance.

Marking each distribution as measured or assumed, and printing that in the
output, is the part of this record that will matter most in an argument. A
distribution presented without provenance is read as a measurement, and the
spread on this tool's first results will be almost entirely chosen rather than
measured. Saying so in the artefact is the difference between an honest
uncertainty budget and a number with decorative error bars.

Keeping the analytic form available and labelled is a concession to the case
where somebody wants an answer in a second. It is kept because they will find a
way to get one anyway, and a labelled fast mode is better than a sampled run with
too few samples.

## Reasons against

The cost is real and it falls where the tool is most used. A footprint is
thousands of trajectories and an operator iterating on a design pays that every
time. The analytic mode does not fully answer this, because the case where it is
least accurate is the case somebody iterating on a design is most likely to be
in.

Every distribution being an assumption today is a serious objection to the whole
shape. A sampled answer whose inputs are all guessed has a spread that is a
property of the guesses rather than of reality, and it can be tighter or wider
than the truth by an unknown factor. Printing that it is assumed does not make
the number better, it only makes it honest, and somebody will reasonably say that
an honest wrong spread is still a wrong spread.

Correlating the heat transfer coefficient with the drag coefficient is a
modelling choice made here without a measurement behind it, and it narrows the
spread relative to drawing them independently. That is a decision that makes the
answer look better and it is recorded here rather than buried in the
implementation.

Holding the density factor fixed over a trajectory rather than redrawing it is
the same kind of choice in the other direction: it widens the spread. Both are
defensible and neither is measured.

There is also a fair argument that the sample count belongs in this record as a
number rather than as an input. Leaving it to the operator means most runs use
whatever the example used.

## What would change this

A published error distribution for any entry in the list. The atmosphere is the
most valuable one and the most likely to be obtainable, because the model
comparison issue #16 asks for produces exactly that: two models over the same
case give a difference, and a difference is the start of a distribution.

A measurement showing the analytic mode agrees with the sampled one across the
regime where the demise threshold matters. That would weaken the reason for the
default and is checkable as soon as both exist. It is not expected.

A material library that carries uncertainties, from issue #48, which would move
the material entry from having no distribution to having one and is the single
change that would most improve the honesty of the whole budget.

A convergence measurement showing the sample count needed for a stable expected
casualty number is far larger than a footprint run can afford. Nothing is
measured. If it were true, the answer would be a variance reduction technique
with its own record rather than a return to the analytic form.

## What depends on this

Record 0008, which this record depends on for the per-sample stream and which
this record does not restate.

Record 0009, whose risk summary artefact carries the statement of which
distributions are assumptions.

Issue #71, the dispersion implementation, which is where this list becomes code
and where the sample count and its convergence measure live.

Issue #16, the atmosphere, which owns the density uncertainty and is the source
this record is waiting on.

Issue #48, the material record, which owns the material property uncertainties.

Issue #55 and issue #52, the bridging relation and the regime boundaries, which
own the aerodynamic coefficient uncertainty.

Issue #65, the breakup altitude, which owns the entry with the most published
basis.

Issue #56, attitude, and record 0011, which records what is outside the model
boundary and therefore not sampled at all.

Issue #81, the validation standing, whose wording has to agree with the
statement this record requires a run to print.

## Sources

1. Ostrom, C. et al. "Operational and Technical Updates to the Object Reentry
   Survival Analysis Tool." First International Orbital Debris Conference, 2019.
   NASA NTRS 20190033904. Read and recorded in
   `docs/survey/existing-codes.md`.
