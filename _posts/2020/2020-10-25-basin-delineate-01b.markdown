---
layout: post
title: "Basin delineation: 1b patch up DEM pits"
categories: basindelineate
excerpt: "Patch up pits in DEM using GRASS"
tags:
  - GRASS7
  - GDAL
  - QGIS
  - DEM
  - fix
image: avg-trmm-3b43v7-precip_3B43_trmm_2001-2016_A
date: '2020-10-25 T06:17:25.000Z'
modified: '2020-10-25 T06:17:25.000Z'
comments: true
share: true
figure1: GRASS7_Amazonia-Startup_welcome01

---
<script src="https://karttur.github.io/common/assets/js/karttur/togglediv.js"></script>

## Introduction

The basin delineation system outlined in this blog only requires a Digital Elevation Model (DEM) as input. The quality of the DEM is, however, critical. [Part 1a](../basin-delineate-01a) deals with how to change land locked voids of "no data" (__not__ 0 elevation) in a DEM to have a valid elevation of 0. This part instead goes through the rather more complicated process of filling up pits and flatten peaks to correct the DEM hydrologically. If you want to use the python package [basin_extract](https://github.com/karttur/basin_extract) you must also have prepared an XML file as outlined in the [introductory post](../basin-delineate-00). If you did use the Python package [basin_extract](https://github.com/karttur/basin_extract) for replacing land locked "no data" in [part 1a](..7basin-delineate-01a) the GRASS script for filling up pits and flattening peaks has already been created.

## Prerequisites

These instructions assume that you have a DEM called _inland_comp_DEM_ in your local GRASS mapset; the DEM generated in [part 1a](../basin-delineate-01a). If you have another DEM, you just need to change the local GRASS reference to that DEM in the XML parameter file.

## GRASS preparations

There are several GRASS commands that can be used for mending and adjusting DEMs. The easiest is to use ordinary gap filling ([r.fill.stats](https://grass.osgeo.org/grass78/manuals/r.fill.stats.html) or [r.fillnulls](https://grass.osgeo.org/grass78/manuals/r.fillnulls.html)) but these methods do not consider the flow routing. Instead GRASS offers the [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html) algorithm. [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html) identifies pits along flow paths and then fills them. You can also use [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html) for mending small no data holes, by first filling all the holes with a low elevation (typically by setting the elevation equal to zero (0) and then run [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html)). If you followed [part 1a](../basin-delineate-01a) of this series this is actually what you are now doing.

The command [r.flowfill](https://grass.osgeo.org/grass78/manuals/addons/r.flowfill.html) is an alternative, available as an [addon](https://grass.osgeo.org/grass78/manuals/addons/). To install GRASS addons you must have prepared the GRASS c-compiler as described in the post [Install GDAL, QGIS and GRASS](https://karttur.github.io/setup-ide/setup-ide/install-gis/#grass).

If you want to try [r.flowfill](https://grass.osgeo.org/grass78/manuals/addons/r.flowfill.html) , the command for installing it is:

<span class='terminal'>> g.extension  extension=r.fill.gaps</span>

The rest of this manual however, uses [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html).

## DEM pits

Artificial (virtual) pits are a common problem in DEMs. Most flow routing algorithms, including GRASS [r.watershed](https://grass.osgeo.org/grass78/manuals/r.watershed.html) can, however, handle pits. To delineate river basins from a DEM, filling up of pits is thus strictly not required. But hydrological modelling of both surface and ground water flow and storage will improve if the DEM is correct.

If you want to fill up pits in your DEM in a hydrologically sound manner you should run [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html). For large grids that will, however, take a very (very) long time. Thus you need to tile the DEM, with some overlap, before running [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html) separately on each tile. To write such a command structure would also take a day. But with the help of the Python package [basin_extract](https://github.com/karttur/basin_extract) the GRASS commands will be setup in a second. Below both the principles for the pit filling, and how to use [basin_extract](https://github.com/karttur/basin_extract), are covered.

## GRASS processing

[r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html) is a demanding process. The python package [basin_extract](https://github.com/karttur/basin_extract) thus automatically tiles the DEM, with overlap to capture all pits. This speeds up the processing about 2 orders of magnitude (100-fold).

Apart from filling up pits, [basin_extract](https://github.com/karttur/basin_extract) can also be used to flatten peaks. The filling of pits is defined by the parameter _fillDirCells_ and the flattening of peaks by the parameter _invertedFillDirCells_. The two parameters work in a similar way; set to _0_ no processing is done, set to _1_ pits/peaks the size of a single cell are filled/flattened, any larger number will render pits/peaks up to that size (in cells) to be filled/flattened. The filling up of pits is always levelling the pits with the surrounding terrain. For the flattening of peaks, there are three alternatives, defined in the parameter _invertedFillValue_ (the alternatives refers to columns in the vector database where the respective values are stored):
1. flattening to the lowest DEM neighbour [_nbdemmin_]
2. flattening to next highest DEM neighbour [_filldem_]
3. flattening to the 1st quartile of the DEM neighbourhood [_nbdem1q_]

Both the filling of pits and the flattering of peaks can be restricted by applying an SQL (parameters = _fillSQL_ and _inverteFillSQL_). Filling up of single cell pits (parameter _fillDirCells_ = _1_) does not require any SQL. With the parameter _fillDirCells_ set to multiple cells, you can apply an SQL, or sequence of SQLs to create defined channels. But the general recommendation is to only fill up pits that are a single cell in size.
For peaks, it is suggested that you apply an SQL that restricts the flattening also of single cell peaks to those that are located adjacent to a stream channel - identified from an [r.watershed](https://grass.osgeo.org/grass78/manuals/r.watershed.html) upstream analysis.

### _basin_extract_ control over pit filling

[r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html) can be used for directly filling up pits (or flattening peaks from an inverted DEM). For several reasons this is not how the pit filling / peak flattening is implemented in the package _basin_extract_. Instead the pits (peaks) identified from [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html) are vectorized as points, and information on the location of each point and its neighbourhood is then added to the vector database. Then an SQL is applied to fill up the original DEM, giving more control over both which cells to be filled and to what elevation value. Further, the vectorization process also guarantees that all pits in the overlap between tiles are correctly superimposed on the DEM.

### Restrict DEM filling to coastal strip

If you want to restrict the pit filling to the coastal strip (or another geographic sub-region) you need to create a mask ([r.mask](https://grass.osgeo.org/grass78/manuals/r.mask.html)). A mask limits the region for raster operations and will reduce the processing time.

To set a mask restricting the processing to the coastal strip you have to create a new layer for the terminal drainage - this time excluding the cells identified in _drain_terminal_small_.

<span class='terminal'>> r.mapcalc \"drain_terminal_v2 = if(isnull(\'inland_comp_DEM\'), 1, null())\" --overwrite</span>

Then do an [r.buffer](https://grass.osgeo.org/grass78/manuals/r.buffer.html) starting from version 2 (_v2_) of the terminal drainage, set the maximum distance to the width of the coastal strip you want to analyse (e.g. 3 kilometers in the example).

<span class='terminal'>> r.buffer input=drain_terminal_v2 output=coastal_strip distances=3000 units=meters --overwrite</span>

Apply an [r.mapcalc](https://grass.osgeo.org/grass78/manuals/r.mapcalc.html) calculation to create a boolean layer of the coastal strip.

<span class='terminal'>> r.mapcalc \"coastal_mask = if((coastal_strip > 1), 1, null())\"</span>

To apply the mask by [r.mask](https://grass.osgeo.org/grass78/manuals/r.mask.html) you just set it and it will remain in place until explicitly removed.

<span class='terminal'>> r.mask raster=coastal_mask</span>

### Filling pits in the DEM

As noted above, [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html) is problematic to apply with large DEM files. Thus I only list the principal steps here. To actually run the pit filling/peak flattening, make use of the Python package [basin_extract](https://github.com/karttur/basin_extract) to create the required GRASS commands as explained further down.

The [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html) command can be set to either identify all pits regardless of size, or to identify pits that are just a single cell in size. Filling up single cell pits is more straight forward and the recommended practice.

The principal steps for filling up single cell pits (with the _-f_ flags set in [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html)) in a DEM in GRASS:

- r.fill.dir -f input-dem -> fill-dem-pt [the f flag forces filling of single cells only]
- r.mapcalc difference between fill-dem-pt and input-dem
- r.to.vect for cells (points) with difference
- v.db.addcolumn for storing filled DEM value
- v.what.rast extract fill DEM to vector points
- v.patch to collect all vector points to fill in a single data source
- v.to.rast rasterize the DEM values
- r.mapcalc superimpose the filled DEM values over the original DEM

The principal steps for filling up also pits larger than a single cell in a DEM in GRASS:

- r.fill.dir input-dem -> fill-dem-area [all pit cells filled regardless of area]
- r.mapcalc difference between fill-dem-area and input-dem
- r.to.vect for regions (areas) with difference
- v.db.addcolumn for storing filled DEM value
- v.what.rast extract fill DEM to polygon centroids
- v.patch to collect all vector areas to fill in a single data source
- v.to.rast rasterize the DEM values
- r.mapcalc superimpose the filled DEM values over the original DEM

Superimposing the vector points on top of the original DEM can be done using different approaches. Apart from the GRASS commands used above (v.patch, v.to.rast and r.mapcalc) you can also make use of GDAL or ogr2ogr, detailed in  [part 1c](../basin-delineate-01c). You can also add additional columns and data to the vector database, and utilise that data for defining an SQL for more controlled superimposition.

### Create tiled process loop

The example DEM used in this tutorial of central South America is approximately 22700 × 22500 cells. This is far too large to process in one go. The Python package developed for assisting the basin delineation, [basin_extract](https://github.com/karttur/basin_extract) can generate a tiled process loop for GRASS.

#### Setup xml file and run _basin_extract_

The xml structure to use for parameterising [basin_extract](https://github.com/karttur/basin_extract) is explained in the [introduction](../basin-delineat-00). If you followed the instructions in [part 1a on removing land locked voids](../basin-delineate-01a), you will already have a complete XML structure of this part as well. The parameters affecting the pit filling and peak flattening include:

- fillDirCells
- fillSQL
- invertedFillDirCells
- invertedFillValue
- inverteFillSQL

Running [basin_extract](https://github.com/karttur/basin_extract) with _stage = 0_ generates five shell script files. The tiled [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html) processing is in the script file _"region"_grass_dem-filldir-pt_stage0-step1.sh_. The first part of the shell script explains how to use it, followed by a processing sequence for each tile (the latter has been simplified in the example below):

```
# Basin delineation: Hydrological DEM fillup (r.fill.dir) with tiled DEM
# Created by the Python package basin_extract

# The tiling speeds up the process hundredfold compared to using the entire DEM.
# Just make sure you have the parameter "tilecelloverlap" set to capture the pit sizes you want to fill.

# To run this script you must have GRASS GIS open in a Terminal window session.
# Change the script it to be executable by the command:
# chmod 755 /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia_test_20201201/0/stage0/script/amazonia_grass_dem-filldir-pt_stage0-step1.sh
# Then execute the command form your GRASS terminal session:
# GRASS 7.x.3x ("region"):~ > /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia_test_20201201/0/stage0/script/amazonia_grass_dem-filldir-pt_stage0-step1.sh

# The script uses the filled DEM from stage0-step1 "inland_comp_DEM" as input.
# If you want to use another DEM as input you must manually edit this script file.

g.region -a n=-3681019.528636 s=-3914992.450317 e=-8846956.322919 w=-9080929.244600
r.fill.dir -f input=inland_comp_DEM output=hydro_cellfill_DEM_0_0 direction=hydro_cellfill_draindir_0_0 areas=hydro_cellfill_problems_0_0 --overwrite
r.mapcalc "DEM_cellfill_diff_0_0 = hydro_cellfill_DEM_0_0 - inland_comp_DEM" --overwrite
r.mapcalc "inland_fill_cell_0_0 = if(DEM_cellfill_diff_0_0 != 0, 1, null() )" --overwrite
r.to.vect input=inland_fill_cell_0_0 output=inland_fill_pt_0_0 type=point --overwrite
v.db.addcolumn map=inland_fill_pt_0_0 columns="filldem DOUBLE PRECISION"
v.what.rast map=inland_fill_pt_0_0 column=filldem raster=hydro_cellfill_DEM_0_0
v.out.ogr input=inland_fill_pt_0_0 type=point format=ESRI_Shapefile output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia_test_20201201/0/stage0/inland-fill-pt-0-0.shp --overwrite

g.region -a n=-3449363.170536 s=-3685652.655798 e=-8846956.322919 w=-9080929.244600
r.fill.dir
```

Below I have taken the essential, but simplified, sequence of commands for the first tile (_0_0_) and inserted explanations of the key processes steps:

```
# This example shows the script for single cell filling (with the -f flag set):

# region is automatically set in the tiling process
g.region -a n=-3681019.528636 s=-3914992.450317 e=-8846956.322919 w=-9080929.244600

# the -f flag restricts the filling to single cell pits
r.fill.dir -f input=inland_comp_DEM output=hydro_cellfill_DEM_0_0 direction=hydro_cellfill_draindir_0_0 areas=hydro_cellfill_problems_0_0 --overwrite

# get the difference between the filled and the original DEM
r.mapcalc "DEM_cellfill_diff_0_0 = hydro_cellfill_DEM_0_0 - inland_comp_DEM" --overwrite

# create boolean mask of changed cells
r.mapcalc "inland_fill_cell_0_0 = if(DEM_cellfill_diff_0_0 != 0, 1, null() )" --overwrite

# convert the cells with altered elevation to a point vector
r.to.vect input=inland_fill_cell_0_0 output=inland_fill_pt_0_0 type=point --overwrite

# add column to vector file with altered elevations
v.db.addcolumn map=inland_fill_pt_0_0 columns="filldem DOUBLE PRECISION"

# read the altered DEM values to the point vector
v.what.rast map=inland_fill_pt_0_0 column=filldem raster=hydro_cellfill_DEM_0_0

# export the point vector to an ESRI shape file
v.out.ogr input=inland_fill_pt_0_0 type=point format=ESRI_Shapefile output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia_test_20201201/0/stage0/inland-fill-pt-0-0.shp --overwrite
```

The flattening of peaks operate in a similar manner, but from an inverted DEM. 

#### Superimpose filled DEM over original DEM

All areas identified as pits or peaks are captured in the sequence of tiles. You now have several alternatives for superimposing these values over the original DEM, outlined in [part 1c](../basin-delineate-01c).
