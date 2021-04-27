---
layout: post
title: "Basin delineation: Visualization of DEM patch up"
categories: basindelineate
excerpt: "Superimpose the pit filled cells over the DEM"
tags:
  - GRASS7
  - GDAL
  - QGIS
  - DEM
  - fix
image: avg-trmm-3b43v7-precip_3B43_trmm_2001-2016_A
date: '2020-10-26 T06:17:25.000Z'
modified: '2020-10-26 T06:17:25.000Z'
comments: true
share: true
figure1: GRASS7_Amazonia-Startup_welcome01

---
<script src="https://karttur.github.io/common/assets/js/karttur/togglediv.js"></script>

## Introduction

The previous posts on [removing land locked data voids](), [identifying pits and peaks](#) and [superimposing selected pits and peaks](#) can be done automatically without any manual adjustment. Visual inspection is, however, often necessary for improving the results (adjusting the parameters). this post will take you through how you can visualize the processing in both GRASS and in QGIS with exported data. 


basin delineation system outlined in this blog only requires a Digital Elevation Model (DEM) as input. The quality of the DEM is, however, critical. [Part 1a](../basin-delineate-01a) deals with how to change selected "no data" pits in a DEM to have a valid elevation of 0. [Part 1b](../basin-delineate-01b) deals with how to identify pits and peaks. This section deals with alternatives for how to superimpose the identified pits and peaks from [part 1b](../basin-delineate-01b) over the input DEM.

## Prerequisites

These instructions assume that you have a DEM called _inland_comp_DEM_  in your local GRASS mapset; the DEM generated in [part 1a](../basin-delineate-01a). If you have another DEM, you just need to change the local GRASS reference to that DEM in the shell script. The shell scripts and command for the DEM pit filling in this post are generated alongside the processing explained in [part1a](../basin-delineate-01a) and [part1b](../basin-delineate-01b). You should thus already have the two alternative scripts files for superimposing filled pits and/or flattened peaks.

## Superimpose filled DEM over original DEM

If you followed the instructions in [part1b]([part 1a](../basin-delineate-01a)) you should have a set of tiles with vector points representing filled pits and flattened peaks (dependent on your parameterization). The databases of each of these vector files should at least contain columns on the DEM values to superimpose over the input DEM in order to fill up pits and flatten peaks. By default, the inverted DEM (peaks) vector database also contains additional information, on the neighbouring elevation of the input DEM and on the upstream accumulated drainage size. This information can be used for controlling which peaks are actually adjusted, and to which elevation. You can create similar information to control also the filling of pits.

Superimposing the adjustments on the input DEM is done in two steps: mosaicking (patching), the tiles into a single vector and then superimposing. Dependent on your machine setup and whether or not you want to use any further information for defining an SQL for piloting the superimposition, two alternative scripts are prepared by _python_extract_:

- GRASS internal patchup and superimpositin
- GDAL and ogr2ogr mosaicing and superimposition

The first alternative, using GRASS, is slower, more complex, but can also be expanded to include additional information and data and thus used to control the DEM adjustment in greater detail and with better precision.

The GDAL alternative is faster and easier to implement, given that you have installed GDAl on your machine. Also GDAL and ogr2ogr can be used for adding additional information and data but requires a whole new set of commands that are not covered in these instructions.

### GRASS commands

The GRASS commands for patching the vector tiles and then superimposing the adjusted values are in the script file called "region"_grass_patch-tiles-dem-filldir-pt_stage0-step2.sh.

```
# Basin delineation: Patch up and superimpose filled cells from r.fill.dir using GRASS
# Created by the Python package basin_extract

# To run this script you must have GRASS GIS open in a Terminal window session.
# Change the script to be executable by the command:
# chmod 755 /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia_test_20201220/0/stage0/script/amazonia_grass_patch-tiles-dem-filldir-pt_stage0-step2.sh
# Then execute the command form your GRASS terminal session:
# GRASS 7.x.x ("region"):~ > /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia_test_20201220/0/stage0/script/amazonia_grass_patch-tiles-dem-filldir-pt_stage0-step2.sh

# Create a shell variable of all the input vectors
MAPS=$(g.list type=vector separator=comma pat="inland_fill_pt_*")

# Patch together all the input vectors
v.patch -e input=$MAPS output=inland_fill_pt

# Rasterize the patched vectors to the value of column filldem
v.to.rast input=inland_fill_pt  output=inland_fill_pt use=attr attribute_column=filldem --overwrite

# Superimpose the rasterized filldir DEM values
r.mapcalc "hydroptfill_DEM  = if( isnull(inland_fill_pt),inland_comp_DEM, inland_fill_pt)" --overwrite

# Export the patched vector as an ESRI shape file by removing the "#"
# v.out.ogr input=inland_fill_pt type=point format=ESRI_Shapefile output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia_test_20201220/0/hydro-fill-pt_drainage_amazonia_0_cgiar-250.shp

# Export the hydrologically corrected DEM as a geotiff
r.out.gdal -f input=hydroptfill_DEM format=GTiff type=Int16 output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia_test_20201220/0/hydro-fill-dem_drainage_amazonia_0_cgiar-250.tif --overwrite
```



The principle steps for superimposing the adjustements over the input DEM


All areas identified as pits are captured in the sequence of tiles. You now have several alternatives for superimposing these values over the original DEM. All alternatives can be applied using a tile by tile approach, or applied for the whole region after patching the tiles.

Filling of pits affects flow routing and hydrological modeling. When the flow routing enters a flat area, an algorithm traversing the flat area towards the lowest drainage(s) out of the flat must be identified. Dependent on the parameterization of the flow dividsion (single or multiples and the spread if multiple) the flat area will be wetted differently. This will later also affect the estimation of soil moisture, evapotranspiration and flooding tendencies of the cells in the flat area (in general all tend to increase). The most widely spread recommendation is to allow filling of single cell pits, but be more careful with filling larger and larger pits.  

On the other hand, DEMs generated by SAR data have a tendency to underestimate the elevation of water surface (due to radar signal double bouncing on water surface and shoreline structures). Thus no general recommendation can be given regarding how to handle larger depressions in DEMs.

You have different alternatives for superimposing
The alternatives for filling pits that I have tested include:

- superimpose tile by tile with GDAL
- patch up vectors in GDAL, superimpose
- patch up vectors in GRASS, rasterize, superimpose

GDAL is quicker and can be used for _burning_ vector data on a raster. The burning can be done on a tile by tile basis, or after patching up the tiles to a single vector data source. With GRASS you have to first patch the vector tiles, then rasterize them, and then superimpose. This takes longer and demands more processing capacity.

##### Patching tiles (optional)

If you use GDAL (GDAL_rasterize) for burning the filled DEM on the original DEM you do not need to patch the tiles. Just continue to the section on _Rasterize using GDAL_ below. The shell script for the burning is already produced by the Python package _basin_extract_.

You can use two different methods for patching together the tiles. The first alternative is to use the GRASS command [v.patch](https://grass.osgeo.org/grass78/manuals/v.patch.html). The commands for patching all the times are given at the end of the GRASS script file:

```
g.region raster=DEM@PERMANENT

MAPS=$(g.list type=vector separator=comma pat="inland_fill_pt_*")
v.patch -e input=$MAPS output=inland_fill_pt

MAPS=$(g.list type=vector separator=comma pat="inland_fill_area_*")
v.patch -e input=$MAPS output=inland_fill_area
```

The other alternative is to use the GDAL vector command [ogr2ogr](). The command line process for patching together all vectors are prepared in the script file _"region"_ogr_part0.sh_, produced and reported by the Python package _basin_extractor_. The first lines of the shell script looks like this:

```
ogr2ogr -skipfailures -nlt /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/flowdir-pt-dem_drainage_amazonia_0_cgiar-250.shp /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/stage0/inland-fill-pt-0-0.shp

ogr2ogr -skipfailures -nlt /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/flowdir-pt-dem_drainage_amazonia_0_cgiar-250.shp /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/stage0/inland-fill-area-0-0.shp

ogr2ogr  -append -skipfailures -nlt /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/flowdir-pt-dem_drainage_amazonia_0_cgiar-250.shp /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/stage0/inland-fill-pt-0-1.shp

ogr2ogr -append  -skipfailures -nlt /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/flowdir-pt-dem_drainage_amazonia_0_cgiar-250.shp /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/stage0/inland-fill-area-0-1.shp
```

The two first lines coopies the input vectors to a new file, all following lines appends the input vectors to the initial output file.

##### Patch up vectors in GRASS, rasterize, superimpose

It is more complicated to superimpose the filled elevation values using GRASS compare to GDAL. For GRASS you first have to patch up the vectors, then rasterize and finally superimpose. The required sequence of commands is included in the GRASS shell script file that was produced when running _basin_extract_ for stage0. The very last lines of the that shell script file should be something like:

```
```


<span class='terminal'></span>
v.to.rast input=inland_fill_area where="area_km2<2.0" output=inland_fill_accept use=attr attribute_column=filldem --overwrite</span>

<span class='terminal'>r.mapcalc inland_fill_dem = if( isnull(inland_fill_accept), inland_comp_DEM, inland_fill_accept)</span>

#### Rasterize using GDAL

With GDAL you can instead use [GDAL_rasterize](https://gdal.org/programs/gdal_rasterize.html) to burn the DEM values from the vectors with the filled DEM value directly over the original DEM. The burning can be done using either the vector tiles or a patched vector. By using vector tiles you bypass the need for patching tiles.

That requires fewer step and is already prepared for running in the two shell script files:

- amazonia_GDAL_pt_part0.sh
- amazonia_GDAL_area_part0.sh

Here are the first lines from the latter:
```
GDAL_rasterize -a filldem /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/stage0/inland-fill-area-0-0.shp /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/inland-comp-demm_drainage_amazonia_0_cgiar-250.shp
GDAL_rasterize -a filldem /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/stage0/inland-fill-area-0-1.shp /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/inland-comp-demm_drainage_amazonia_0_cgiar-250.shp
```

### Visualizing the DEM errors and fixes

If you want to try the hydrological fixing of DEM using [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html), you need to start by setting a subregion to process with [g.region](https://grass.osgeo.org/grass78/manuals/g.region.html). About 1 million cells is a suitable region.

<span class='terminal'>> g.region -a n="float" s="float" e="float" w="float"</span>

If you do not set a subregion, the process will take several hours, or even days. With a small subregion set , you can run [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html).

for single cell pits only:

<span class='terminal'>> r.fill.dir -f input=inland_comp_DEM output=hydro_cellfill_DEM_ direction=hydro_cellfill_draindir areas=hydro_cellfill_problems --overwrite</span>

or for all depressions:

<span class='terminal'>> r.fill.dir input=inland_comp_DEM output=hydro_areafill_DEM_ direction=hydro_areafill_draindir areas=hydro_areafill_problems --overwrite</span>

Calculate the difference between the pit filled and the original DEM using [r.mapcalc](https://grass.osgeo.org/grass78/manuals/r.mapcalc.html).

<span class='terminal'>> r.mapcalc "DEM_fill_diff = hydro_areafill_DEM - inland_comp_DEM" --overwrite</span>

Set a color ramp to the difference layer.

<span class='terminal'>> r.colors DEM_fill_diff color=differences</span>

Export the difference layer to a goetiff.

<span class='terminal'>> r.out.gdal format=GTiff input=DEM_fill_diff output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazoniax/0/DEM-fill-diff_amazonia_0_cgiar-250.tif --overwrite</span>

Calculate a boolean map of the difference area.

<span class='terminal'>> r.mapcalc "inland_fill_areas = if(DEM_fill_diff != 0, 1, null() )" --overwrite</span>

Convert the boolean map of identified pits to a vector.

<span class='terminal'>> r.to.vect input=inland_fill_areas output=inland_fill_areas type=area --overwrite</span>

Export the vector with identified pits to an ESRI shape file.

<span class='terminal'>> v.out.ogr input=inland_fill_areas type=area format=ESRI_Shapefile output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazoniax/0/inland-fill-areas_SRTM_hydro-amazonia_0_cgiar-250.shp --overwrite</span>

You can inspect the exported files in for instance <span class='app'>QGIS</span>

You can also open a monitor for GRASS directly from the command line and then add the layers from the command line.

<span class='terminal'>> d.mon wx0</span>

<span class='terminal'>> d.shade shade=inland_comp_DEM</span>

<span class='terminal'>> color=inland_comp_DEM</span>

<span class='terminal'>> d.vect inland_fill_areas type=boundary color=red</span>
