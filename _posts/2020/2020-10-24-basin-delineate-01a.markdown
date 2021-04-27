---
layout: post
title: "Basin delineation: 1a patch up DEM voids"
categories: basindelineate
excerpt: "Patch up DEM small 'no data' regions at the shoreline using GRASS"
tags:
  - GRASS7
  - GDAL
  - QGIS
  - DEM
  - no data
image: avg-trmm-3b43v7-precip_3B43_trmm_2001-2016_A
date: '2020-10-24 T06:17:25.000Z'
modified: '2020-10-24 T06:17:25.000Z'
comments: true
share: true

---
<script src="https://karttur.github.io/common/assets/js/karttur/togglediv.js"></script>

## Introduction

Land locked region of voids or "no data" in DEMs gorge up all entering flow vectors. This causes under estimation of downstream flow and errors in for example flooding and water balance models. The objective of the instructions in this post is to remove all land enclosed "no data" regions from the DEM to use for basin delineation. The post is part of the instruction on [Basin delineation from Digital Elevation Models](../../).

## Prerequisites

You must have setup GRASS 7 and imported a DEM as described under [Installation & Setup](../../basindelineatesetup). If you want to use the python package [basin_extract](https://github.com/karttur/basin_extract) you must also have prepared an XML file as outlined in a [previous post](../basin-delineate-00).

### DEM coastal "no data" errors

Narrow bays, estuaries, lagoons etc. can cause problems in near shore regions. Both because they can come out as pits, but a worse problem is if such features become defined as land locked "no data" cells. That will cause problems both for identifying the basin outlets and later for modeling water flow out of the basin. To get rid of small "no data" regions along the coast you need to identify them and then assign an elevation that allows water to pass.

## GRASS processing

The next sections below explain in detail how to use GRASS for identifying and filling land locked "no data" regions. If you only want to run the process, all the commands are summarized further down. The best alternative, however, is to use the Python package [basin_extract](https://github.com/karttur/basin_extract) to generate the script for you, also explained further down.

Once you have filled the land locked "no data" holes you can use [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html) to fill them up with more realistic elevation values, the topic of parts [1b](../basin-delineate-01b) and [1c](../basin-delineate-01c). For running the watershed analysis in [stage 2](../basin-delineate-02), it is, however, enough to assign a low elevation (i.e. 0) to the "no data" holes.

### Make the target directory

If you want to follow the standard of Karttur's GeoImagine Framework, your target folder for the exported layers should be (where _amazonia_ is the region, _GRASS2020_ the local disk and _SRTM_ the source of the DEM):

<span class='terminal'>> mkdir -p /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/stage0/script</span>

### Create a map of the terminal drainage

Create a map of the terminal drainage using the GRASS command [r.mapcalc](https://grass.osgeo.org/grass78/manuals/r.mapcalc.html). For this to work, your original DEM (as described in the post on [GRASS 7 setup](../../basin_delineate_setup/basin-delineate-grass7/) ) must have the terminal drainage defined as "null" in the original DEM. Your DEM must also be named _DEM_ in GRASS and reside the mapset _PERMANENT_. As described in the setup section post on [GRASS 7 setup](../../basindelineatesetup/basin-delineate-grass7), it is good practice to create a separate mapset for the basin delineation, for example:

<span class='terminal'>> g.mapset -c drainage</span>

Then use [r.mapcalc](https://grass.osgeo.org/grass78/manuals/r.mapcalc.html) to create a raster layer of your terminal drainage with all land pixels set to _null_:

<span class='terminal'>>r.mapcalc \"drain_terminal = if(isnull(\'DEM@PERMANENT\'), 1, null())\"</span>

Clump ([r.clump](https://grass.osgeo.org/grass78/manuals/r.clump.html)) all contiguously agglomerated cells together with a unique id to allow identifying large and small bodies representing the terminal drainage (voids or "no data" in the original DEM).

<span class='terminal'>> r.clump input=drain_terminal output=terminal_clumps --overwrite</span>

Convert the clumps to polygons with the command [r.to.vect](https://grass.osgeo.org/grass78/manuals/r.to.vect.html).

<span class='terminal'>> r.to.vect input=terminal_clumps output=terminal_clumps type=area --overwrite</span>

Add the area of each polygon to the the vector database with the command [v.to.db](https://grass.osgeo.org/grass78/manuals/v.to.db.html).

<span class='terminal'>> v.to.db map=terminal_clumps type=centroid option=area columns=area_km2 units=kilometers</span>

Optionally export the vector with the polygons representing "no data" in the original DEM with [v.out.ogr](https://grass.osgeo.org/grass78/manuals/v.out.ogr.html).

<span class='terminal'>> v.out.ogr input=terminal_clumps type=area format=ESRI_Shapefile output=/path/to/dst.shp --overwrite</span>

### Fill in DEM over land

With the command [v.to.rast](https://grass.osgeo.org/grass78/manuals/v.to.rast.html) it is possible to apply an SQL while converting vectors to a raster. Use that function to assign polygons below a threshold size to be the voids to fill. Set the pixel value to 0 - the preliminary elevation you will assign these cells. If you use the package [basin_extract](https://github.com/karttur/basin_extract), the parameter _terminalClumpCellThreshold_ defines the threshold. In the example below I have set the threshold to 2.0 square kilometers. Note that if you use the package [basin_extract](https://github.com/karttur/basin_extract), the threshold area is set as nr of cells (not in distance or area units) in the parameter _terminalClumpCellThreshold_. You should inspect the result and change the threshold if required. The inverted relationship, areas at least as large as the threshold, then represent the large and "true" terminal drainage for the basins of the study region.

<span class='terminal'>> v.to.rast input=terminal_clumps type=area where=\"area_km2< 2.0\" output=drain_terminal_small use=val value=0 --overwrite</span>

<span class='terminal'>> v.to.rast input=terminal_clumps type=area where=\"area_km2 >= 2.0\" output=drain_terminal_large use=val value=0 --overwrite</span>

if you can not identify a threshold that works you might have to consider manually editing the polygons. For example by deleting individual polygons in <span class='app'>QGIS</span>.

Superimpose the small "no data" clumps over the original DEM to create a DEM that is complete over land. If you followed the instructions above the filled voids will all have an elevation = 0.

<span class='terminal'>> r.mapcalc \"inland_comp_DEM = if(( isnull(DEM@PERMANENT)), drain_terminal_small,DEM@PERMANENT )\" --overwrite</span>

Export the completed DEM for land areas:

<span class='terminal'>> r.out.gdal format=GTiff  input=inland_comp_DEM output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazoniax/0/inland-comp-DEM_amazonia_0_cgiar-250.tif --overwrite</span>

### _basin_extract_ python package

The required sequence of commands to patch up the "no data" holes of the DEM can be generated with the python package [basin_extract](https://github.com/karttur/basin_extract). You have to change the [xml file in the Introduction](../basin-delineate_00) to reflect your region, DEM and local disk names. The relevant core parameters include:

- stage [0, 1, 2 or 3] (this part is for stage = '0')
- verbose [0, 1 or 2]
- proj4CRS ['proj4CRS definition for output data if input layers lack proper projection information']
- grassDEM = the original input DEM

#### Output from _basin_extract_

Running [basin_extract](https://github.com/karttur/basin_extract) with _stage = 0_ generates five shell script files. The processing commands for replacing isolated "no data" holes in the DEM is in the shell script file named _"region"_grass_fix-terminal-drainage-nodata_stage0-step0.sh_.

### GRASS shell script

Shell script for replacing land locked "no data" regions with 0.

```
# Basin delineation: Terminal drainage no-data pit removal
# Created by the Python package basin_extract

# To run this script you must have GRASS GIS open in a Terminal window session.
# Change the script it to be executable by the command:
# chmod 755 /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia_test_20201201/0/stage0/script/amazonia_grass_fix-terminal-drainage-nodata_stage0-step0.sh
# Then execute the command form your GRASS terminal session:
# GRASS 7.x.3x ("region"):~ > /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia_test_20201201/0/stage0/script/amazonia_grass_fix-terminal-drainage-nodata_stage0-step0.sh

# Create destination path
mkdir -p /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia_test_20201201/0/stage0/script

# Create a map of the terminal drainage
r.mapcalc "drain_terminal = if(isnull('DEM@PERMANENT'), 1, null())" --overwrite

# Clump agglomerated cells of the terminal drainage
r.clump input=drain_terminal output=terminal_clumps --overwrite

# Vectorize the clumps
r.to.vect input=terminal_clumps output=terminal_clumps type=area --overwrite

# Add the area of each clump to the clump polygons attribute table
v.to.db map=terminal_clumps type=centroid option=area columns=area_km2 units=kilometers

# Export all clumps as ESRI shape files by removing the "#"
# v.out.ogr input=terminal_clumps type=area format=ESRI_Shapefile output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia_test_20201201/0/terminal-clumps_drainage_amazonia_0_cgiar-250.shp --overwrite

# Create a polygon vector for all clumps below a threshhold size.
# These polygons will be changed to land areas with initial elevation = 0
v.to.rast input=terminal_clumps type=area where="area_km2< 2.000000" output=drain_terminal_small use=val value=0 --overwrite

# Export all small (land) clumps as ESRI shape files by removing the "#"
# v.out.ogr input=drain_terminal_small type=area format=ESRI_Shapefil output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia_test_20201201/0/terminal-clumps-small_drainage_amazonia_0_cgiar-250.shp --overwrite

# Create a polygon vector for all clumps above a threshhold size.
v.to.rast input=terminal_clumps type=area where="area_km2 >= 2.000000" output=drain_terminal_large use=val value=0 --overwrite

# Export all large (terminal drainage) clumps as ESRI shape files by removing the "#"
# v.out.ogr input=drain_terminal_large type=area format=ESRI_Shapefile output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia_test_20201201/0/terminal-clumps-large_drainage_amazonia_0_cgiar-250.shp --overwrite

# Mapcalc a new composit filled DEM over all terrestrial regions
r.mapcalc "inland_comp_DEM = if(( isnull(DEM@PERMANENT)), drain_terminal_small,DEM@PERMANENT )" --overwrite

# Export the terrestrial composite DEM as a geotiff
r.out.gdal format=GTiff  input=inland_comp_DEM output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia_test_20201201/0/inland-comp-DEM_drainage_amazonia_0_cgiar-250.tif
```

### Next step

The DEM created in this example, _inland_comp_DEM_ is sufficient for use with [r.watershed](https://grass.osgeo.org/grass78/manuals/r.watershed.html) in [part 2](../basin-delineate-02). Parts [1b](../basin-delineate-01b) and [1c](../basin-delineate-01c) deal with filling up pits as a way to hydrologically correct the DEM. But that is strictly not needed for running [r.watershed](https://grass.osgeo.org/grass78/manuals/r.watershed.html).
