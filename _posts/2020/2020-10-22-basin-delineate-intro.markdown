---
layout: post
title: "Basin delineation: Introduction"
categories: basindelineate
excerpt: "Introduction to Digital Elevation Models for hydrological modeling"
tags:
  - GRASS7
  - GDAL
  - QGIS
  - DEM
  - basin
  - watershed
image: avg-trmm-3b43v7-precip_3B43_trmm_2001-2016_A
date: '2020-10-22 T06:17:25.000Z'
modified: '2020-10-22 T06:17:25.000Z'
comments: true
share: true

---
<script src="https://karttur.github.io/common/assets/js/karttur/togglediv.js"></script>

## Introduction

The basin delineation system outlined in this blog only requires a Digital Terrain Model (DTM) as input. The quality of the DTM is, however, critical. This series of instructions deals with correcting and applying DTMs for hydrological characterisations using mainly GRASS GIS. You can either use the processing steps outlined in the manuals and translate them for your DTM, or you can get the python package [basin_extract](https://github.com/karttur/basin_extract) from [GitHub.com](https://github.com) and automatically generate the processing steps as a series of shell script files.

## Prerequisites

You must have setup GRASS 7 and imported a DEM as described under [Installation & Setup](../../basindelineatesetup). The manual, as well as the python package [basin_extract](https://github.com/karttur/basin_extract), assumes that the input DEM is in the GRASS GIS mapset _PERMANENT_ and have the name _DEM_. The internal GRASS reference to the DTM (or more general: Digital Elevation Model - DEM) then becomes _DEM@PERMANENT_.

## DEM errors affecting basin delineation

Basin delineation is grounded in the fact that water tends to flow downhill (from a higher to a lower potential, but strictly that does not necessarily mean downhill). This routing of water across the hillside and within water courses is the key for connecting any geographic point in the landscape to an outlet point.

DEM issues that can cause problems for flow routing include:

- regions lacking elevation data,
- artificial pits,
- artificial barriers,
- flat regions,
- wide river mouths.

Regions lacking data are generally difficult to mend. There are, however, special cases where it is necessary to patch up the holes. In [part 1a](../basin-delineate-00a) you will mend small "no data" regions (typically up to a few dozen cells) by first assigning a low elevation (typically 0) and then, optionally, apply a pit filling (in [part 0b](../basin-delineate-00b)). It is crucial to remove land locked "no data" cells as these will otherwise gorge all the (virtual) water flow entering. Levelling the filled "no data" is not required but if you intend to use the DEM for hydrological modelling it is recommended.

Artificial (virtual) pits is the most common problem with applying DEM for hydrological modeling, and most flow routing algorithms, including GRASS [r.watershed](https://grass.osgeo.org/grass78/manuals/r.watershed.html) can handle pits on the flow (or fly). For hydrological modeling it is often, but not always, better to mend artificial depressions. These kind of pits are dealt with in [part 0b](../basin-delineate-00b) and superimposed over the DEM in [part 0c](../basin-delineate-00c).

Artificial barriers are more difficult to handle. One way to locate problematic peaks is to invert the DEM and look for pits (inverted peaks) adjacent to water channels. The processing steps for achieving this is included in [part 0b](../basin-delineate-00b) and [part 0c](../basin-delineate-00c) of the DEM patch up, and also coded into the python package [basin_extract](https://github.com/karttur/basin_extract).

In the parameterization you can decide whether or not to include pit filling and peak flattening, and which pits and peaks to level. Note that as of April 2021 I recomment that you prepare your DEM, including pit filling and peak flattening, using [Karttur´s GeoImagine Framework DEM processing](https://karttur.github.io/geoimagine02-docs/blog/dem/) (see next section).

Flat regions and wide river mouths cause similar flow routing problems. Different algorithms can be applied (i.e. Single flow Direction, or SFD, versus Multiple Flow Direction, MFD) for handling flat area. But a more ideal solution is to increase the vertical resolution and properly steer the routing using elevation data. This post includes a test of different flow directions algorithms. To steer the virtual water flow to a single outlet in wide outlets the package can also be used for creating a sloping surface for large river mouths.

## Accessing and correcting DEMs

In another [set of articles on DEM data and processing](https://karttur.github.io/geoimagine02-docs/blog/dem/), belonging to [Karttur's GeoImagine Framework](https://karttur.github.io/geoimagine02-docs/), I have listed some [global DEMs and how to access them](https://karttur.github.io/geoimagine02-docs/blog/blog-global-dems/). The Framework now also contains the hydrological corrections described below, and I have depreciated the development of those parts for the _basin_extract_ package.

## Parameterizing _basin_extract_

Some of the processing for delineating river basins from a DEM can be manually edited, but some parts require an automated approach. The python package [basin_extract](https://github.com/karttur/basin_extract) was created to automate most of the processing. The package is parameterized from _json_ parameter files. The _json_ file used for the example region (Amazonia in South America) is included and explained here as a reference.

```

```

### \<userproject\>

The \<userproj\> tag contains global attributes, including:
- _userid_ and _projectid_ are the ids of the user and proejct when the package is used as part Karttur's GeoImagine Framework and can be set to anything when the package is used as a stand alone solution,
- _tractid_ identifies the genographical study region, and is used as part of the name of all generated files,
- _alias_ is used for substituting _tractid_ for defining the path of the location where generated files are saved,
- _siteid_ and _plotid_ are more detailed location identifiers not used for DEM processing,
-  _system_ refers to the organisation and projection of the data.

For the regions _Amazonia_ in this example the _system_ is _MODIS_ (the MODIS sinusoidal tiling system and projection).

### \<filecheck\>, \<overwrite\> and \<delete\>

\<filecheck\>, \<overwrite\> and \<delete\> are boolean parameters that determine if to restrict the processing to checking and reporting existing files, whether or not to recreate (overwrite) existing files, or just delete all files.

### \<process\>

The \<process\> tag defines which process that is defined when the package is used as part of Karttur's GeoImagine Framework.

### \<parameters\>

The core parameters that determine the process settings:

- stage = '0' # the process stage [0, 1, 2, 3 or 4]
- grassDEM = 'DEM@PERMANENT' # internal name of DEM in GRASS to use as input, NOTE that this can vary with stage
- adjacentDist = '-1' # The distance to use for identfying shorelines, if negative it is automatically set to the length of the diagonal of a single cell, otherwise in the distance units of the projection (e.g. metres)
- outlet = 'mouth' # Routing routine [SFD, MFD, mouth]
- distill = 'mouth' # Clumping routine [SFD, MFD, mouth]
- clusterOutlet = 'maximum' # Method for identifying single outlet cell from cell cluster of outlets [maximum, central]
- invertedFill ='1' #
- basinCellThreshold='2000' # Threshold in nr of cells for identifying basins
- watershed='MFD5' # flow routine when iterating r.watershed [SFD or MFD]
- terminalClumpCellThreshold # Threshold area for separating landlocked "no data" from the terminal drainage into which the basins spill out in nr of cells
- fillDirCells # 0 for no pit filling, 1 for filling of single cell pits, any higher integer for filling pits with sizes up to that number of cells.
- fillSQL # SQL for specifying the filling of pits.
- invertedFillDirCells # 0 for no inverted pit filling (peak suppression), 1 for depressing of single cell peaks, any higher integer for depressing peaks with sizes up to that number of cells.
- invertedFillSQL # SQL for specifying the flattening of peaks
- invertedFillValue # value to assign for the peaks to flatten [nbdemq1, nbdemmin, demfill]
- tileCellSize # Tile size in nr of cells when running [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html) for pits and peaks
- tileCellOverlap  # Tile overlap in nr of cells with analysing for pits and peaks
- proj4CRS = '+proj=sinu +lon_0=0 +x_0=0 +y_0=0 +a=6371007.181 +b=6371007.181 +units=m +no_defs' # proj4CRS definition for output data if input layers lack proper projection information
- verbose = '2' # Level of text response [0, 1, 2]

### \<srcpath\> and \<dstpath\>

The source and (_src_) and destination (_dst_) paths defines the volume (disk) of the input DEM (_src_) and all the outputs (_dst_). Each tag only have a single attribute, _volume_. In the example the same volume is (external disk) is used.

### \<comp\>

The \<comp\> (composition) tag contains the references to actual datasets. The reference to the input Digital Elevation Model (_DEM_) must be stated. The _system_ tag defined the generic structure of any resulting, or destination, data source produced by the package. If _system_ is not given, the _DEM_ tag is used for also defining the destination naming. All destination data sources have hard coded names both for the internal GRASS processing and for exporting as destination data sources. You can, however, override the export name for any destination data source. You do that by creating a complete reference, with the tag set to the default prefix and then with attributes defining the path and name of that particular destination data source.

Table 1. List of all data sources with prepared default names for export from GRASS to generic formats (GeoTiff for raster data and ESRI shape files for vector data). Note that some data sources are exported with hardcoded names. Most data sources produced internally do not have any export name. To export there internal data sources you need to manually define the export in GRASS.

|**default name** | **content**|
|**Stage 0** | -|
|  terminal-clumps | Clumps of contiguous cells of the terminal drainage feature |
| terminal-clumps-small  | Clumps of contiguous cells of the terminal drainage feature representing land holes  |
| inland-comp-DEM  | Composite DEM with all "no data" holes replaced with 0  |
| hydro-fill-pt | Single cell pits identified for DEM filling (as vector) |
| inverted-fill-pt | Single cell peaks identified for DEM suppression (as vector) |
| hydro-fill-pt-dem | DEM with single cell filled pits  |
| hydro-fill-area | Multi cell pits identified for DEM filling (as vector) |
| hydro-fill-area-dem | DEM with multi cell sized pit cell filled pits   |
|  hydro-fill-dem | DEM with filled pits (single and multiple cells)  |
| inverted-fill-dem |  DEM with flattened peaks (single and multiple cells) |
| fill-dem-tile  | The tiles used for filing pits and flattening peaks  |
| **Stage 1**   |  - |
| upstream-ln-SFD | Color ramped natural log upstream accumulated area from SFD |
| upstream-ln-MFD | Color ramped natural log upstream accumulated area from MFD |
| lowlevel-outlet-clumps | DEM over the levels of the basins |
| shoreline-outlet-clumps | Single cell deep shoreline clumps |
| all-outlets-pt | Combined SFD + MFD outlet points expanded to fill mouth widths  |
| SFD-outlets-pt | Single Flow Direction outlet points |
| MFD-outlets-pt | Multiple Flow Direction outlet points  |
| thickwall | Initial ("thick") barriers that closes off all basin mouths |
| mouthshoreline | Intermediate barriers for the basin mouths |
| shorewall | Final barrier wall that closes off all basin mouths |
| shorewall-pt | Final barrier wall convereted to vector points |
| shorefill-DEM | DEM with barriers sealing of the full width of all outlet mouths |
| **Stage 2**  | - |
| basin-allmouth-outlets | Distilled outlet points from full width basin mouths |
| basin-allmouth-outlets-redundant | Redundant outlet points from full width basin mouths |
| basin-allmouth-areas | Distilled basin polygons from full width basin mouths |
| basin-allmouth-outlets-omitted | Omitted outlet points from full width basin mouths |
| basin-allmouth-areas-omitted | Omitted basins polygons from full width basin mouths |
| basin-allmouth-outlets-remaining | Remaning outlet points from full width basin mouths |
| basin-allmouth-outlets-duplicate | Duplicate outlet points from full width basin mouths |
| basin-allmouth-costgrow | Cost grow from outlet point of full width basin mouths |
| basin-allmouth-route-DEM | DEM with tilted outlet towards single cell outlet of full width basin mouths |
| basin-allmouth-hydro-DEM | DEM with each basin flowing out through a single cell |
| basin-allmouth-hydro-DEM-updrain-ln | DEM with color ramp for visualization (only) |
| basin-SFDpoint-outlets | Basin outlet points from SFD flow model |
| basin-SFDpoint-outlets-redundant | Redundant outlet points from SFD flow model |
| basin-SFDpoint-outlets-omitted | Omitted outlet points from SFD flow model |
|  basin-SFDpoint-areas-omitted | Omitted outlet areas from SFD flow model |
| basin-SFDpoint-outlets-remaining | Remaining outlet points from SFD flow model |
| basin-SFDpoint-outlets-duplicate | Duplicate outlet points from SFD flow model |
| basin-MFDpoint-outlets | Basin outlet points from MFD flow model |
| basin-MFDpoint-outlets-redundant | Redundant outlet points from MFD flow model |
| basin-MFDpoint-outlets-omitted | Omitted outlet points from MFD flow model |
|  basin-MFDpoint-areas-omitted | Omitted outlet areas from MFD flow model |
| basin-MFDpoint-outlets-remaining | Remaining outlet points from MFD flow model |
| basin-MFDpoint-outlets-duplicate | Duplicate outlet points from MFD flow model |

### Compositions, or file paths and names

All compositions contain the following components or parts:

- source
- product
- content
- layerid
- prefix
- suffix

The _source_ is the origin of the dataset (e.g. the satellite platform, the organization or individual behind the data), _product_ is usually a coded product identifier or the producer, _content_ is a thematic identifier, _layerid_ is a content identifier with _prefix_ usually identical to _layerid_ but can be set differently, and _suffix_ is a more loose part that can be freely set but usually represents a version identifier.

Compositions as such do not relate to any spatial extent or temporal validity.

#### File naming conventing

All data file names are composed of five (5) parts of which three relate to the composition (above), and the remaining two to location and timestamp. In the filename, the parts are separated by underscore, ("\_"), and the parts are not allowed to contain any underscore themselves:

1. band or content identifier (from composition)
2. product or producer (from composition)
3. location
4. timestamp
5. suffix (from composition)

All files in the thus have the following general format:

content_product_location_timestamp_suffix.extension

Within each part, a hyphen ("-") is used for separating different codes or labels. A hyphen in the timespan part, for example, denotes that the data in the file represent aggregated data for the period between two dates.

### Hierarchical folder structure

All the data files are organized in a hierarchical folder structure with the following levels:

- system
- source (from composition)
- division (tiles, region or mosaic)
- content (from composition)
- location
- timestamp

The two lowest hierarchical levels, _location_ and _timestamp_, are identical to the _location_ and _timestamp_ in the file name. The thematic _content_ can be anything like "basin", "elevation", "landform" or other thematic identifier and equals the _folder_ in the composition. _division_ can only take three different values "tiles", "region" or "mosaic". All basin delineation is done at the _region_ level.

The _source_ level identifies the source or origin of the data and is equal to the composition _source_, for instance "SRTM" for Shuttle Radar Topography Mission data. The top _system_ level is also a fixed attribute of each process, and relate to the different systems for representing data that is included in the Framework (e.g. _ancillary_, _modis_, _sentinel_ or any of the three EASE-grid v2 projections [_ease2n_, _ease2s_ and _ease2t_]), plus one system (_system_) for some default data.
