# The population grids the risk number would be computed against

The risk number is the ground footprint multiplied by who is underneath it. The
second factor is somebody else's data set, and picking one fixes a licence, a
resolution and a reference year at the same time. This document records the
candidates that could be checked from public sources.

Two things decided the shape of the document. Every number in the table below
either carries a source or was measured with a command that is quoted with it,
and the properties that matter most to a reentry footprint, which are the ones
about water, the poles and the antimeridian, are recorded separately because the
data set pages do not answer them. Where a file could be opened those properties
are read out of it instead, and where it could not the section says so.

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

    curl -sIL https://jeodpp.jrc.ec.europa.eu/ftp/jrc-opendata/GHSL/GHS_POP_GLOBE_R2023A/GHS_POP_E2020_GLOBE_R2023A_4326_30ss/V1-0/GHS_POP_E2020_GLOBE_R2023A_4326_30ss_V1_0.zip
    HTTP/1.1 200 OK
    Accept-Ranges: bytes
    Content-Length: 482351880
    Content-Type: application/zip

    curl -sIL https://data.worldpop.org/GIS/Population/Global_2000_2020/2020/0_Mosaicked/ppp_2020_1km_Aggregated.tif
    HTTP/1.1 200 OK
    Content-Length: 869715253
    Content-Type: image/tiff

So the global GHS-POP grid for one epoch is 322 MB compressed at 1 km in
Mollweide, 482 MB compressed at 30 arc-second in WGS84 and 5.1 GB compressed at
100 m, and the WorldPop global 1 km mosaic for 2020 is 870 MB as an uncompressed
GeoTIFF. The 100 m figure settles one thing on its own: a 100 m
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

Two of the four candidates could be read from their own files, and for those the
file is the source below rather than the page describing it. The other two are
not obtainable without an account and the last section says what that leaves.

### Reading the corner of the file that answers this

A GeoTIFF carries its no-data value, its cell size and the position of its first
cell in tags at the front of the file, so the questions above can be answered
without the raster behind them. The JRC file server honours a byte range and the
GeoTIFF is the first member of each archive, so the front of the archive is
enough:

    curl -s -r 0-599999 https://jeodpp.jrc.ec.europa.eu/ftp/jrc-opendata/GHSL/GHS_POP_GLOBE_R2023A/GHS_POP_E2020_GLOBE_R2023A_4326_30ss/V1-0/GHS_POP_E2020_GLOBE_R2023A_4326_30ss_V1_0.zip > ghs4326.bin
    curl -s -r 0-599999 https://jeodpp.jrc.ec.europa.eu/ftp/jrc-opendata/GHSL/GHS_POP_GLOBE_R2023A/GHS_POP_E2020_GLOBE_R2023A_54009_1000/V1-0/GHS_POP_E2020_GLOBE_R2023A_54009_1000_V1_0.zip > ghsmoll.bin

data.worldpop.org answers a range request with the whole 870 MB file, so the head
of the stream is taken and the connection dropped instead:

    curl -s https://data.worldpop.org/GIS/Population/Global_2000_2020/2020/0_Mosaicked/ppp_2020_1km_Aggregated.tif | head -c 120000 > wp.bin

One reader serves all three. It follows the first image directory of a plain
GeoTIFF or of the first deflated member of a zip, in classic or BigTIFF form, and
prints the tags this section rests on:

    python - ghs4326.bin <<'PY'
    import struct, sys, zlib
    b = open(sys.argv[1], 'rb').read()
    if b[:4] == b'PK\x03\x04':
        n1, e1 = struct.unpack('<HH', b[26:30])
        b = zlib.decompressobj(-15).decompress(b[30 + n1 + e1:], 400000)
    bo = '<' if b[:2] == b'II' else '>'
    big = struct.unpack(bo + 'H', b[2:4])[0] == 43
    ifd = struct.unpack(bo + ('Q' if big else 'I'), b[8:16] if big else b[4:8])[0]
    step, pl = (20, 8) if big else (12, 4)
    base = ifd + (8 if big else 2)
    n = struct.unpack(bo + ('Q' if big else 'H'), b[ifd:base])[0]
    size = {2: 1, 3: 2, 4: 4, 12: 8}
    name = {256: 'width', 257: 'height', 258: 'bits', 339: 'sampleformat',
            33550: 'pixelscale', 33922: 'tiepoint', 42112: 'metadata', 42113: 'nodata'}
    for k in range(n):
        e = b[base + k * step: base + (k + 1) * step]
        tag, typ = struct.unpack(bo + 'HH', e[:4])
        cnt = struct.unpack(bo + ('Q' if big else 'I'), e[4:4 + pl])[0]
        if tag not in name:
            continue
        nb, p = cnt * size.get(typ, 1), e[4 + pl:4 + 2 * pl]
        d = p[:nb] if nb <= pl else b[struct.unpack(bo + ('Q' if big else 'I'), p)[0]:][:nb]
        v = (d.rstrip(b'\0').decode() if typ == 2 else
             list(struct.unpack(bo + '%d%s' % (cnt, 'd' if typ == 12 else 'H'), d)))
        print(name[tag], '=', v)
    PY

### What the two readable grids say

GHS-POP R2023A in WGS84 at 30 arc-second, from `ghs4326.bin`:

    width = [43202]
    height = [21384]
    bits = [64]
    sampleformat = [3]
    pixelscale = [0.00833333330032682, 0.008333333299795073, 0.0]
    tiepoint = [0.0, 0.0, 0.0, -180.00791593130032, 89.0995831776456, 0.0]
    metadata = <GDALMetadata>
      <Item name="STATISTICS_MAXIMUM" sample="0">277608.74043128</Item>
      <Item name="STATISTICS_MEAN" sample="0">8.4874269494672</Item>
      <Item name="STATISTICS_MINIMUM" sample="0">0</Item>
      <Item name="STATISTICS_STDDEV" sample="0">235.7608994699</Item>
      <Item name="STATISTICS_VALID_PERCENT" sample="0">100</Item>
    </GDALMetadata>

There is no no-data tag in that file. The reader prints one when it is present
and printed none, every cell is declared valid, and the smallest value is zero.
So in this variant water and land where nobody lives are the same value, and
nothing in the file distinguishes them.

GHS-POP R2023A in Mollweide at 1 km, from `ghsmoll.bin`, is the same product with
the opposite convention:

    width = [36082]
    height = [18000]
    bits = [64]
    sampleformat = [3]
    pixelscale = [1000.0, 1000.0, 0.0]
    tiepoint = [0.0, 0.0, 0.0, -18041000.0, 9000000.0, 0.0]
    metadata = <GDALMetadata>
      <Item name="STATISTICS_MAXIMUM" sample="0">338726.57110608</Item>
      <Item name="STATISTICS_MEAN" sample="0">56.785121486231</Item>
      <Item name="STATISTICS_MINIMUM" sample="0">0</Item>
      <Item name="STATISTICS_STDDEV" sample="0">679.48856478339</Item>
      <Item name="STATISTICS_VALID_PERCENT" sample="0">21.26</Item>
    </GDALMetadata>
    nodata = -200

That is the finding this section exists for, and it is sharper than the one that
was expected. The disagreement is not between two producers, it is between two
distributions of one product. A consumer that hardcodes minus two hundred reads
a real population as a sentinel in the WGS84 file, and one that hardcodes nothing
adds minus two hundred per water cell in the Mollweide file, which moves the sum
in the direction that hides people rather than inventing them.

The Mollweide sentinel covers water and not only the corners outside the
projection. The projection's own ellipse is

    python -c "import math; a=2*6378137*math.sqrt(2); b=6378137*math.sqrt(2); print(math.pi*a*b/(36082000*18000000))"
    0.7871082124602158

so 78.71 per cent of that grid's bounding box is inside the projection while
21.26 per cent of it carries a value. Dividing the second by the first leaves
27.0 per cent of the projection interior holding data, which is near the land
fraction of the Earth, so the sentinel covers the water as well as the corners.

WorldPop's global mosaic for 2020, from `wp.bin`:

    width = [43200]
    height = [18720]
    bits = [32]
    sampleformat = [3]
    pixelscale = [0.0083333333, 0.0083333333, 0.0]
    tiepoint = [0.0, 0.0, 0.0, -180.001249265, 83.99958319871001, 0.0]
    nodata = -3.4028234663852886e+38

The sentinel is the most negative finite value a 32-bit float holds, which is the
safest of the three conventions here because a consumer that forgets to mask
produces a result nobody could mistake for a population.

### The poles

Neither readable grid reaches them, and this was assumed the other way before the
files were opened. The corner and the cell count give the extent directly:

    python -c "print(89.0995831776456 - 21384*0.008333333299795073)"
    -89.10041610517223
    python -c "print(83.99958319871001 - 18720*0.0083333333)"
    -72.00041617728999

GHS-POP in WGS84 runs from 89.0996 N to 89.1004 S, so about nine tenths of a
degree at each end is outside the file. Its Mollweide distribution stops in the
same latitude: the grid's north edge is 9,000,000 m while the projection's own
north limit is 6378137 times the square root of two, which is 9,020,047.8 m, and
inverting Mollweide at that edge gives 89.09 degrees.

    python -c "import math; R=6378137.0; t=math.asin(9e6/(R*math.sqrt(2))); print(math.degrees(math.asin((2*t+math.sin(2*t))/math.pi)))"
    89.09138295441029

WorldPop is the one that matters for this tool. Its global mosaic covers 83.9996 N
to 72.0004 S and no further. A reentry from a high-inclination orbit crosses both
of those latitudes on every revolution, and a footprint that lands beyond them is
not a footprint over empty ground, it is a footprint over cells that are not in
the file. The two failures look identical in a sum and are not the same thing,
which is the case the tool has to refuse rather than report as a small number.

Nothing about the two unreadable grids is claimed here. LandScan Global states an
extent of 90 N to 90 S [3], and that statement is not checked against the file.

### The antimeridian

The two readable grids sit differently against the line and neither sits on it.

GHS-POP in WGS84 is 43,202 cells wide where 360 degrees is 43,200, and it starts
west of the line:

    python -c "print(-180.00791593130032 + 43202*0.00833333330032682, 43202*0.00833333330032682 - 360)"
    180.008749309419 0.01666524071930553

So it runs from 180.0079 W to 180.0087 E and overlaps itself by almost exactly
two cells. Ground inside that overlap appears twice in the array, and a footprint
crossing the line counts it twice unless the consumer knows.

WorldPop leaves a gap in the same place instead:

    python -c "print(-180.001249265 + 43200*0.0083333333, 0.001249265*111319.49)"
    179.99874929500004 139.06754267485002

Its array runs from 180.0012 W to 179.9987 E, so the strip between 179.9987 E and
the line, about 139 m wide at the equator, is in no cell at all, and the strip of
equal width at the western edge carries a longitude below minus 180 that a naive
lookup either clamps or rejects.

Both of these are small in area and neither is small in consequence, because the
error they produce is silent. A footprint crossing the antimeridian is exactly
the case a reentry produces on most orbits.

The Mollweide grid has its break at the projection's outer boundary, which also
falls on the antimeridian, and its measured extent is padded about 900 m beyond
that boundary on each side rather than clipped to it.

    python -c "import math; print(18041000 - 2*6378137*math.sqrt(2))"
    904.3038527071476

### What GPW v4.11 says about water

GPW v4.11 does not encode water in the population grid at all. It ships a
separate Water Mask product whose single band classifies each cell as "0: Total
water pixels that are completely water and/or permanent ice", "1: Partial water
pixels that also contain land", "2: Total land pixels" or "3: Ocean pixels" [5].
Applying it is the consumer's job, which means a population sum over a coastal or
oceanic footprint that skips the mask is wrong in a direction nobody notices.
That is read from a catalogue description of the product and not from the file,
for the reason in the last section.

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

Coverage is a third filter and it separates those two. The WorldPop global mosaic
holds nothing north of 84 N or south of 72 S, and GHS-POP holds nothing beyond
about 89.1 degrees in either direction, both measured above. Whichever is chosen,
a footprint reaching past the edge of the grid is a case the tool has to name
rather than a case where the population happens to be zero.

## What this survey did not establish

The water, pole and antimeridian behaviour of GPW v4.11 and of LandScan Global as
their files hold it. Neither file can be fetched without an account, which is
measured rather than described:

    curl -sI "https://landscan.ornl.gov/system/files/LandScan%20Global%202022.zip" | grep -E "^HTTP|^Server"
    HTTP/1.1 403 Forbidden
    Server: AmazonS3

    curl -sS -L https://sedac.ciesin.columbia.edu/downloads/data/gpw-v4/gpw-v4-population-count-rev11/gpw-v4-population-count-rev11_2020_30_sec_tif.zip
    curl: (28) Failed to connect to sedac.ciesin.columbia.edu:443 after 21051 ms: Could not connect to server

For those two the record above is what their pages say, and the pages do not
state a no-data value. The GPWv4 water mask classes are cited from a catalogue
description of that product rather than from reading the file, and LandScan
Global's stated 90 N to 90 S extent is likewise its own claim about itself. The
two grids that could be opened both turned out to differ from what their
descriptions implied, so those two statements are the weakest lines in this
document and should be treated that way until somebody with an account checks
them.

The licence of the LandScan Global data as distinct from the licence of the
paper describing it. This is the exact conflation the issue warns against and the
survey stops at recording it rather than guessing.

Download sizes for GPW v4.11 and LandScan Global, for the same reason as above.

Whether the no-data conventions measured here hold for the other epochs of the
same products. One epoch of each was read, 2020 in both cases, and a producer
that changed the convention between releases would not be visible to that.

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
