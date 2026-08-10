# 0016. The material record: fields, provenance per value, and what the loader refuses

## Date

2026-08-10

## Status

accepted

## Question

What does one material in the library have to carry, so that a wrong number or
a missing one is visible in the output rather than absorbed into it, and what
does the loader do with a material that does not carry it.

The question is asked before any of the models that read a material, because
the answer decides what those models are allowed to assume. A thermal model
written against a flat table of constants cannot later be handed a curve
without every consumer of the property changing with it.

What this record does not answer is which numbers belong in the library, or
whether this project ships one at all. That is issue #49 and entry 4 of #1, and
`docs/survey/material-data.md` is their input.

## Options considered

A flat table of constants, one row per material and one column per property.
Cheapest to write, cheapest to read, and it is what the published demise
studies work from. One of them says so of its own database in the words the
survey quotes:

    git grep -n 'assumed temperature independent' origin/main -- docs/survey/material-data.md
    origin/main:docs/survey/material-data.md:299:properties are assumed temperature independent" [4, section 2.5]. A composite

It costs three things at once. Specific heat capacity, thermal conductivity and
emissivity are not flat from ambient to a melting point for the alloys this
tool cares about, so a constant is a modelling choice made silently in a data
file. A value has nowhere to say where it came from. And a value has nowhere to
say what it is worth, which is the field record 0007 depends on.

Constants with one citation per material. Better, and it fails for the reason
the issue that plans this record names. A material assembled from four sources
carries one citation, so a disagreement with another code is traceable to a
material rather than to a number, and tracing it to a number is the whole
reason to record provenance at all.

A general schema with arbitrary named properties and free-form metadata. It
carries anything, including everything a later model might want, and it removes
the loader's ability to refuse. A loader that does not know which properties
are required cannot report that one is missing, so the refusal rule of record
0010 would have nothing to stand on and the absence would surface as a model
reading a property that is not there.

A reference into an external database resolved when the tool runs. It removes
the redistribution problem by never shipping a value. It is refused by record
0028, which keeps every network request off the calculation path, and it would
make a run unreproducible against a database that moves under it.

A property entry carrying value, unit, validity range, uncertainty and
provenance, with the value expressible as a constant or as a curve, taken
below.

## Decision

The material library is a file the tool reads, in TOML, with a schema version
as its first key. Record 0009 decided that format and that rule for the
scenario, for a parser that cannot be talked into constructing something and
for a version that is read rather than guessed. The material library is the
second untrusted input this tool has and it takes the same answer for the same
reasons, which are not restated here.

The library holds one record per material. A material is named, and a component
in a scenario references it by that name rather than copying its values, which
is what record 0005 requires of a fragment.

A property is an entry rather than a number. Every entry carries five things,
and a missing one is a missing property.

The value, which is either a constant or a curve over temperature. Both are
expressible and neither is preferred by the schema.

The unit the value is written in, stated in the file. Record 0004 keeps SI
inside the tool and puts conversion at the boundary, and a data file an
operator writes is that boundary.

The temperature range the value is valid over. This is the field that makes the
next paragraph possible and it is required even where the value is a constant.

The uncertainty, or the statement that the source gives none. Record 0007 sends
the per-property uncertainty here and says that until it exists the material
entry of its own uncertainty list has no distribution at all. A source that
states no uncertainty is recorded as stating none. It is never assigned one,
because a fabricated spread is the same defect as a fabricated value and it is
harder to see.

The provenance, which is the source, the page or table inside it, and which of
measurement, handbook value, fit or estimate the number is. Provenance is per
value and not per material, so a material assembled from four sources carries
four of them.

The required properties are density, specific heat capacity, thermal
conductivity, melting temperature, heat of fusion and emissivity. The list is
this tree's rather than the general literature's, and the survey derives it
from what the models read:

    git grep -n '^## What the demise model reads' origin/main -- docs/survey/material-data.md
    origin/main:docs/survey/material-data.md:17:## What the demise model reads

Three of them carry a field that a bare number cannot hold.

Melting temperature names which temperature it is. An alloy melts over a range
rather than at a point, so the entry says whether the value is a solidus, a
liquidus or a nominal figure, and a source that does not say is recorded as not
saying.

Emissivity names the surface condition the value was measured for. An oxidised
or roughened surface radiates differently from a polished coupon, and
emissivity is the only heat loss this model has.

A constant standing where the property varies over the range it is asserted
over carries a sentence saying so. The loader requires the sentence to be there
and cannot judge what it says; whether the simplification is defensible is what
review is for.

Each material also carries the terms it is in the file under. The survey found
no openly redistributable source carrying this property set for any alloy class
this tool needs, so the licence position is the constraint that decides whether
a row exists rather than a footnote on one. What those terms may be for a row
this project ships is issue #49. This record fixes only that the field is
required and that a row without it is not loadable.

What the schema does not require, so that a source is neither rejected for
lacking it nor accepted for carrying it. Mechanical strength, because record
0011 leaves structural failure out of the model. Heat of vaporisation, because
the demise criterion stops at melt and record 0011 leaves melt layer removal
out in the specific sense that melted material is assumed to leave the body
immediately.

The loader refuses in four cases, and each refusal names the input, the field,
where in the scenario or the library it came from, and what would fix it, which
is the shape record 0010 fixes.

A material a component names and the library does not hold. Refused before
flight, naming the component and the material string as it was written.

A required property absent from a material the run needs. Refused before
flight, naming the material, the property, the file it was read from and the
component that referenced it.

A property entry missing one of its five fields. Refused before flight, and the
material is not partially loaded. A value with no provenance is not a value
with unknown provenance, it is a number somebody typed.

A property evaluated outside the temperature range the entry declares. This one
is not knowable before flight, because the temperature a fragment reaches is an
outcome of the run. The fragment ends in the Refused terminal state that record
0005 names, the run flies every other sample to its end under record 0010, and
the artefacts carry the refused count. The value is not extrapolated. An
extrapolated specific heat capacity produces a demise altitude that reads
exactly like an earned one.

An operator may supply a library of their own instead of or alongside anything
this project ships. The manifest records which library a run read and at which
version, which is the property record 0005 asks for when it makes a fragment
hold a reference rather than a copy.

No path root is fixed here. The `Scope:` line of issue #48 names
`data/materials/` and record 0013 fixes six crate names and no directory, which
is an open question on issue #26 rather than something this record settles in
passing.

## Reasons

The demise answer is a property lookup with physics wrapped around it, and the
physics is checkable in a way the lookup is not. A reader can follow the heat
balance in a document and cannot tell, from any output this tool produces, that
the emissivity underneath it was a handbook value for a polished coupon rather
than a measurement of an oxidised surface. Putting provenance next to the value
is what makes that visible without reading the source.

Provenance per value follows from what disagreement between two codes looks
like. It is almost never a disagreement about a material. It is a disagreement
about one number inside one material, and a citation attached to the material
cannot resolve it.

The uncertainty field exists because record 0007 already committed to it. That
record lists the material properties among the uncertain inputs, marks them
assumed, and says the distribution comes from here. Leaving the field out would
leave a landed record pointing at a schema that cannot answer it.

Refusing rather than extrapolating follows the rule this project has already
taken twice. Record 0010 makes a missing input a refusal and never a default,
and a value evaluated outside its stated range is a missing input wearing the
last number that was in range.

Requiring the range even for a constant is what makes that refusal possible. A
constant with no range is a value that is silently valid everywhere, which is
the flat table this record rejected.

## Reasons against

The schema is heavy against the data that exists. Six properties with five
fields each is thirty fields for one material, and the survey's result is that
no redistributable source fills them for any alloy class this tool needs:

    git grep -n 'The result of this survey is negative' origin/main -- docs/survey/material-data.md
    origin/main:docs/survey/material-data.md:7:The result of this survey is negative and it is worth stating before the detail.

So the first library that satisfies this schema may be very small, and a schema
nobody can fill invites a fork with a simpler one that quietly drops the fields
that made it expensive.

Refusing an evaluation outside the declared range will refuse runs that the
reference codes complete. An operator comparing this tool against ORSAT or DAS
will find that this one declines to answer where the others produce a number.
The argument that the others extrapolate silently is the right argument and it
does not help somebody who needs a figure this week, and the pressure to add a
flag that permits extrapolation will come from exactly that direction.

The required set is a metals set. A composite does not melt in the way a
melting temperature and a heat of fusion assume, and a composite row that
satisfies this schema carries a melting temperature with no physical referent.
That is a real defect rather than a hypothetical one; the survey records a
published study doing it. The schema will accept such a row and nothing here
catches it.

Provenance per value is a heavy cost on the one person the library depends on.
Entering a material from a handbook becomes six citations with page numbers
rather than one, and the person who would otherwise build the first library is
the person that cost falls on.

An uncertainty field that may say the source gives none will be read as an
optional field. It is not, and the distinction between a stated absence and an
omission is the kind that erodes.

## What would change this

A redistributable source carrying the temperature-resolved property set for an
alloy class. The survey names the single question that would most change the
position, and it was not read:

    git grep -n 'The terms of the NASA Software Usage Agreement for DAS' origin/main -- docs/survey/material-data.md
    origin/main:docs/survey/material-data.md:341:The terms of the NASA Software Usage Agreement for DAS. Whether the material

If that agreement, or the terms of MatWeb, TPSX or the ESA Space Materials
Database, turns out to permit redistribution, the cost of this schema is paid
once against real data rather than argued about against an empty directory.

A composite treatment. The survey records that ORSAT 7.1 added a pyrolysis
model for fibre reinforced plastics and that SCARAB 4 added a demise and
ablation model covering six material types. A pyrolysing material needs
properties this required set does not name, and giving it a melting temperature
to satisfy the loader is the failure named above. That is a supersession rather
than a field added to this record.

A demise criterion that keeps melted material on the body. Record 0011 assumes
it leaves immediately, and issue #62 is where the criterion is decided. If it
decides otherwise, heat of vaporisation stops being a property the model does
not read and the required set grows.

A thermal model that resolves temperature through the thickness. The conduction
mode behind the interface of issue #60 reads thermal conductivity as a curve
rather than as a value at one temperature, and if it turns out to need more
than that, the entry shape rather than the property list is what moves.

A measurement showing that the material property uncertainties dominate the
width of a footprint. Today the uncertainty field is a record-keeping field,
and issue
#71 is where the sampled inputs are chosen. If material properties enter the
sampler, an entry whose source states no uncertainty stops being a recorded
absence and becomes a decision somebody has to take.

## What depends on this

Issue #49, the library this project ships and the licence position of every row
in it, which fills the terms field this record requires.

Issue #50, the parent object and its component list, where a component names a
material and the loader resolves the name.

Issue #60 and issue #62, which read specific heat capacity, thermal
conductivity, melting temperature and heat of fusion, and issue #61, which
reads emissivity and its temperature dependence.

Record 0007, which sends the per-property uncertainty here and has no
distribution for the material entry of its list until this exists.

Record 0010, whose pre-flight refusal list names a material property that a
model reads, and whose in-flight case now has a second member in the
out-of-range evaluation above.

Record 0005, whose fragment holds a reference into this library rather than a
copy of it, and whose Refused terminal state the out-of-range case ends in.

Issue #19, the manifest, which records the library and the version a run read.

Issue #86, whose fuzz targets name the material file among the untrusted
inputs, and which now has a schema to malform rather than a shape to guess.
