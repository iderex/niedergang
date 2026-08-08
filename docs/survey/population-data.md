# The population grids the risk number would be computed against

The risk number is the ground footprint multiplied by who is underneath it. The
second factor is somebody else's data set, and picking one fixes a licence, a
resolution and a reference year at the same time. This document records the
candidates that could be checked from public sources.

Two things decided the shape of the document. Every number in the table below
either carries a source or was measured with a command that is quoted with it,
and the properties that matter most to a reentry footprint, which are the ones
about water, the poles and the antimeridian, are recorded separately because
they are the ones the data set pages do not answer.

## The candidates

| Grid | Producer | Resolution | Reference years | CRS | Licence | Redistribution inside an artefact | Commercial use |
| --- | --- | --- | --- | --- | --- | --- | --- |
| GPW v4.11 | CIESIN, Columbia University, distributed by NASA SEDAC | 30 arc-second, with aggregates at 2.5, 15 and 30 arc-minute and 1 degree | 2000, 2005, 2010, 2015, 2020 | Geographic, WGS84 | "openly shared, without restriction, in accordance with the EOSDIS Data Use and Citation Guidance" [1] | Permitted, with the required citation [1] | Not restricted by the terms read [1] |
| GHS-POP R2023A | European Commission Joint Research Centre | 100 m and 1 km in Mollweide; 3 arc-second and 30 arc-second in WGS84 | 1975 to 2030 in five-year steps, the last two being projections | World Mollweide EPSG:54009, and WGS84 EPSG:4326 | European Commission reuse notice: "Reuse is authorised, provided the source is acknowledged" [2] | Permitted, with attribution. Access conditions recorded as no limitations [2] | Not restricted by the terms read [2] |
| LandScan Global | Oak Ridge National Laboratory | 30 arc-second | Annual, 2000 to 2022 in the release described [3] | WGS84, 21,600 rows by 43,200 columns spanning 90 N to 90 S and 180 W to 180 E [3] | Not established. The paper describing the data is CC BY 4.0; the paper's data availability section says the data sets are freely available through figshare and the LandScan Portal and does not state a licence for the data [3] | Not established | Not established |
| WorldPop | WorldPop, University of Southampton | 3 arc-second and 30 arc-second | Annual. 2000 to 2020 in the global mosaics measured below; the project also publishes 2015 to 2030 sets | Geographic, WGS84 | Creative Commons Attribution 4.0 [4] | Permitted under CC BY 4.0 | Permitted under CC BY 4.0 |

## What a download actually costs

The size of the download decides whether a grid can sit inside the artefact at
all, so it is measured rather than described. These were run against the
publishers' own file servers.

    curl -sIL https://jeodpp.jrc.ec.europa.eu/ftp/jrc-opendata/GHSL/GHS_POP_GLOBE_R2023A/GHS_POP_E2020_GLOBE_R2023A_54009_1000/V1-0/GHS_POP_E2020_GLOBE_R2023A_54009_1000_V1_0.zip
    HTTP/1.1 200 OK
    Content-Length: 322293568
    Content-Type: application/zip

    curl -sIL https://jeodpp.jrc.ec.europa.eu/ftp/jrc-opendata/GHSL/GHS_POP_GLOBE_R2023A/GHS_POP_E2020_GLOBE_R2023A_54009_100/V1-0/GHS_POP_E2020_GLOBE_R2023A_54009_100_V1_0.zip
    HTTP/1.1 200 OK
    Content-Length: 5097074334
    Content-Type: application/zip

    curl -sIL https://data.worldpop.org/GIS/Population/Global_2000_2020/2020/0_Mosaicked/ppp_2020_1km_Aggregated.tif
    HTTP/1.1 200 OK
    Content-Length: 869715253
    Content-Type: image/tiff

So the global GHS-POP grid for one epoch is 322 MB compressed at 1 km and 5.1 GB
compressed at 100 m, and the WorldPop global 1 km mosaic for 2020 is 870 MB as an
uncompressed GeoTIFF. The 100 m figure settles one thing on its own: a 100 m
global grid is not going inside a downloadable artefact, whatever its licence
says.

No equivalent measurement was made for GPW v4.11 or LandScan Global, because
both are served from behind a login or a portal session rather than from a plain
file URL.

## The reference year is a model, not a fact

A grid stamped with a year is an estimate for that year, and three of the four
candidates say so in different ways.

GPW v4.11 allocates census and register counts to cells with "a proportional
allocation gridding algorithm, utilizing approximately 13.5 million national and
sub-national administrative Units", and the estimates are "consistent with
national censuses and population registers" [1]. Its epochs are five years apart
and stop at 2020.

GHS-POP disaggregates GPWv4.11 estimates onto cells using built-up surface from
the Global Human Settlement Layer [2]. Its 2025 and 2030 epochs are projections,
which means using them is using somebody's demographic forecast, and that has to
be visible in the output rather than buried in a file name.

LandScan Global is annual, and it carries the sharpest caveat of the four. Its
authors write that "Users should avoid conducting change analysis at the cell
level" because the method and its input data changed over the series, and that
"Exact accuracy is difficult to report owing to the absence of ground truth
datasets" [3].

None of the candidates offers a year it does not have. There is no interpolation
mode, no forecast beyond the last epoch, and no statement of what error a
substitution costs. So a reentry in a year the grid does not carry is the
operator using a grid from another year, and the tool's job is to say which year
it used rather than to hide the substitution. That is the same rule the
determinism record already fixes for defaults, and it is the reason the grid
identity belongs in the manifest.

## Ambient against residential, which is not a resolution question

LandScan Global is not measuring the same quantity as the other three. It is an
ambient distribution, described by its authors as "an unwarned and average
distribution of population throughout the day", which "goes beyond traditional
residential (nighttime) distributions" by including workplaces, schools,
commercial areas and recreation spaces [3]. The others are residential.

This matters more than a factor of two in resolution. A casualty expectation
computed against an ambient grid and one computed against a residential grid are
different numbers answering different questions, and a footprint crossing a city
centre at noon and one crossing it at three in the morning are the same footprint
to every one of these data sets. Whichever is chosen, the tool has to name the
basis in its output, because a reader who does not know which one was used cannot
compare two results at all.

## Water, the poles and the antimeridian

A reentry footprint is a long thin object that crosses all three, so this section
is the one that decides whether a grid is usable rather than merely licensed.

Over water, only one candidate's handling could be established, and the way it
is established is the finding. GPW v4.11 does not encode water in the population
grid at all. It ships a separate Water Mask product whose single band classifies
each cell as "0: Total water pixels that are completely water and/or permanent
ice", "1: Partial water pixels that also contain land", "2: Total land pixels"
or "3: Ocean pixels" [5]. Applying it is the consumer's job, which means a
population sum over a coastal or oceanic footprint that skips the mask is wrong
in a direction nobody notices.

For GHS-POP, LandScan Global and WorldPop, no statement of the NoData value or
of how water and unpopulated cells are encoded was found in the pages read,
including each product's own catalogue entry. That absence is itself the thing
to act on rather than a gap to fill by assumption: a grid whose NoData is a
negative sentinel and a grid whose NoData is zero give different answers to the
same footprint, and the difference is invisible in the output. Reading the value
out of each file is the next step and it is not done here.

At the poles, the two CRS families behave differently and both behaviours are
properties of the projection rather than of the data. In the WGS84 geographic
grids used by GPW v4.11, LandScan Global, WorldPop and the WGS84 variants of
GHS-POP, the grid is a plate carree covering 90 N to 90 S, so cells exist all the
way to the pole and their ground area shrinks toward zero. LandScan Global states
its extent as exactly that, 21,600 rows by 43,200 columns from 90 N to 90 S [3].
A population per unit area computed from a count per cell is therefore wrong near
the poles unless the cell's true ground area is used. In the Mollweide grids of
GHS-POP the cells are equal area by construction and the pole is a point, which
removes that error and introduces a different one, because the footprint has to
be reprojected before it can be laid over the grid.

At the antimeridian, the WGS84 grids have their edge exactly there: the array
runs from 180 W to 180 E and a footprint crossing that line is two pieces in
array coordinates. Nothing in the sources read describes any of these grids
wrapping, so the wrap is the tool's problem. The Mollweide grid has the same
break in a different place, at the projection's outer boundary, which also falls
on the antimeridian.

None of this is measured. It follows from the stated CRS and grid extent of each
product, and the difference between that and an inspection of the files is
recorded in the last section.

## Whether any of them is data about identifiable people

None of the sources read describes an input that is data about an identifiable
person. Every candidate is built the same way: counts published by censuses,
registers or administrative units, disaggregated onto cells using geospatial
ancillary layers. GPW v4.11 names census and register counts over roughly 13.5
million administrative units [1]. GHS-POP names GPWv4.11 counts and built-up
surface [2]. LandScan Global names census counts disaggregated with "building
footprints, settlement maps, points of interest, nighttime lights, slope, land
cover" [3]. WorldPop names a random forest dasymetric redistribution of census
counts [4].

This is a negative finding and it is written as one. It is what the sources say
about their own inputs, not an assessment of the underlying censuses, and it is
the sentence the data protection statement of issue #88 can point at instead of
asserting the same thing without a source.

One thing that follows anyway, and does not depend on the answer above: the
output of this tool is a footprint over inhabited ground, and that describes
where people are even though it is built from data about places. Issue #24 is
where that is handled and this document does not settle it.

## What is usable inside a shipped artefact

On licence alone, GHS-POP and WorldPop can be redistributed inside an artefact
with attribution, and GPW v4.11 can be redistributed with its required citation.
LandScan Global cannot be relied on for this, because no licence for the data
was found. The Creative Commons notice on the paper describing it covers the
paper.

On size, only the 1 km products are candidates for shipping, and even those are
322 MB to 870 MB for one epoch. That is a real cost to put in a downloadable
tool and it is the number entry 5 of issue #1 should be decided against.

The combination that survives both filters is a 1 km grid from GHS-POP or
WorldPop, shipped for one named epoch, with the operator able to supply another.
This document does not make that decision; it records what the options cost.

## What this survey did not establish

The over-water behaviour of GHS-POP, LandScan Global and WorldPop, including the
NoData value each uses. The GPWv4 water mask classes are cited from a catalogue
description of that product rather than from reading the file.

The licence of the LandScan Global data as distinct from the licence of the
paper describing it. This is the exact conflation the issue warns against and the
survey stops at recording it rather than guessing.

Download sizes for GPW v4.11 and LandScan Global. Both are served through a
session rather than a plain file URL, so no measurement was made.

The pole and antimeridian behaviour above is derived from each product's stated
coordinate reference system and grid extent, not from opening the files. A
reprojection error or an off-by-one at the array edge would not be visible to
this method, and finding one requires reading a grid, which needs tooling this
repository does not have yet. That is the one clause of this issue's Done-when
that is answered from documentation rather than from the data, and it is not
claimed as more than that.

Candidate grids not covered: GRUMP, HYDE, and the constrained WorldPop products
were not examined.

## References

1. Center for International Earth Science Information Network, Columbia
   University. "Gridded Population of the World, Version 4 (GPWv4): Population
   Count, Revision 11." NASA Socioeconomic Data and Applications Center, 2018.
   doi:10.7927/H4JW8BX5. Catalogue entry read at
   <https://www.earthdata.nasa.gov/data/catalog/sedac-ciesin-sedac-gpwv4-popcount-r11-4.11>
2. European Commission Joint Research Centre. "GHS-POP R2023A - GHS population
   grid multitemporal (1975-2030)." JRC Data Catalogue.
   <https://data.jrc.ec.europa.eu/dataset/2ff68a52-5b5b-4a22-8f40-c41da8332cfe>
3. Lebakula, V., Sims, K., Reith, A., Rose, A., McKee, J., Coleman, P., Kaufman,
   J., Urban, M., Jochem, C., Whitlock, C., Ogden, M., Pyle, J., Roddy, D.,
   Epting, J., Bright, E. "LandScan Global 30 Arcsecond Annual Global Gridded
   Population Datasets from 2000 to 2022." Scientific Data 12, 495 (2025).
   doi:10.1038/s41597-025-04817-z.
   <https://pmc.ncbi.nlm.nih.gov/articles/PMC11933676/>
4. WorldPop, University of Southampton. Data hub and licence statement.
   <https://hub.worldpop.org/project/list>
5. "CIESIN GPWv411 GPW Water Mask." Catalogue entry for the GPWv4.11 Water Mask
   product, read for the band classes.
   <https://developers.google.com/earth-engine/datasets/catalog/CIESIN_GPWv411_GPW_Water_Mask>
