---
layout: post
title: "Basin delineation: 1 patch up DEM"
categories: basindelineate
excerpt: "Patch up the DEM using GRASS"
tags:
  - GRASS7
  - GDAL
  - QGIS
  - DEM
  - fix
image: avg-trmm-3b43v7-precip_3B43_trmm_2001-2016_A
modified: '2020-10-04 T06:17:25.000Z'
modified: '2020-10-04 T06:17:25.000Z'
comments: true
share: true
figure1: GRASS7_Amazonia-Startup_welcome01
figure2: GRASS7_1stStartup_create-location
figure3: GRASS7_1stStartup_define-location
figure4A: GRASS7_1stStartup_import-data
figure4B: GRASS7_1stStartup_import-success
figure5a: GRASS7_Amazonia-drainage-SFD
figure5b: GRASS7_Amazonia-drainage-MFD
figure6: GRASS7_Amazon-River-drainage-SFD-MFD

---
<script src="https://karttur.github.io/common/assets/js/karttur/togglediv.js"></script>

## Introduction

The basin delineation system outlined in this blog only requires a Digital Elevation Model (DEM) as input. The quality of the DEM is, however, critical. This first part of the series of instructions deals with hydrological correction of DEMs using GRASS GIS.

## Prerequisites

You must have setup GRASS 7 and imported a DEM as described under [Installation & Setup](../../basindelineatesetup) in this blog. The DEM must have null ("no data") for the water body into which the basins to delineate drain.

## DEM errors affecting basin delineation

Basin delineation is grounded in the fact that water tends to flow downhill (from a higher to a lower potential, but strictly that does not necessarily mean downhill). This routing of water across the hillside and within water courses is the key for connecting any geographic point in the landscape to an outlet point.

DEM issues that can cause problems for flow routing include:

- regions lacking elevation data,
- artificial pits,
- artificial barriers,
- flat regions,
- wide river mouths.

Regions lacking data are generally difficult to mend. There are, however, special cases where it is necessary. In this tutorial you will mend small no data regions (a few cells) by first assigning a low elevation (typically 0) and then apply a pit filling approach to fill the no data holes. This is crucial as the no data will otherwise gorge all the (virtual) water flow entering that cell.

Artificial (virtual) pits are the most common problem, and most flow routing algorithms, including GRASS [r.watershed](https://grass.osgeo.org/grass78/manuals/r.watershed.html) can handle pits. For hydrological modeling it is, however, better to mend the artificial depressions. Also that will be done in the this manual.

Artificial barriers are more difficult to handle. And is not approached in this manual. You could, however, invert the DEM and solve problems with barriers as if they were pits.

Flat regions and wide river mouths cause similar flow routing problems. Different algorithms can be applied (i.e. Single flow Direction, or SFD, versus Multiple Flow Direction, MFD) for handling flat area. But a more ideal solution is to increase the vertical resolution and properly steer the routing using elevation data. This will be approached in [part 3](../basin-delineate-03) of this series.

### Coastal DEM errors

Narrow bays, estuaries, lagoons etc. can cause problems in near shore regions. Both because they can come out as pits, but a worse problem is if such features become defined as land locked no data cells. That will cause problems both for identifying the basin outlets and later for modeling water flow out of the basin. To get rid of small "no data" regions along the coast you need to identify them and then assign an elevation that allows water to pass.

## GRASS preparations

There are several GRASS commands that can be used for mending and adjusting DEMs. The easiest is to use ordinary gap filling ([r.fill.stats](https://grass.osgeo.org/grass78/manuals/r.fill.stats.html) or [r.fillnulls](https://grass.osgeo.org/grass78/manuals/r.fillnulls.html)) but these methods do not consider the flow routing. Instead GRASS offers the [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html) algorithm. [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html) identifies pits along flow paths and then fills them. You can also use [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html) for mending small no data holes, by first filling all the holes with a low elevation (typically DEM = 0).

The command [r.flowfill](https://grass.osgeo.org/grass78/manuals/addons/r.flowfill.html) is an alternative, available as an [addon](https://grass.osgeo.org/grass78/manuals/addons/). To install GRASS addons you must have prepared the GRASS c-compiler as described in the post [Install GDAL, QGIS and GRASS](https://karttur.github.io/setup-ide/setup-ide/install-gis/#grass).

If you want to try [r.flowfill](https://grass.osgeo.org/grass78/manuals/addons/r.flowfill.html) , the command for installing it is:

<span class='terminal'>> g.extension  extension=r.fill.gaps</span>

The rest of this manual however, uses [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html).

## GRASS processing

The following sections detail how to use GRASS to patch up a DEM to be hydrologically sound. The next section deals with identifying no data cells to include in the DEM. The text explains the process. If you only want to run the process, all the commands are supplied towards the end of the post.

Filling up of pits requires that you run [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html). For large grids that will take a very long time. For a DEM of 10 M pixels it can take 24 hours or longer. Thus you need to tile the DEM, with some overlap, before running [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html) separately on each tile. To write such a command structure would also take a day. But with the help of the Python package _basin_extract_ the GRASS commands will be setup in a second. Below both the principles for the pit filling, and how to use _basin_extract_, are covered.

### Create a map of the terminal drainage

Create a map of the terminal drainage using the GRASS command [r.mapcalc](https://grass.osgeo.org/grass78/manuals/r.mapcalc.html). For this to work, your DEM must have the terminal drainage defined as "null" in the original DEM.

<span class='terminal'>>r.mapcalc "drain_terminal = if(isnull('DEM@PERMANENT'), 1, null())"</span>

Clump ([r.clump](https://grass.osgeo.org/grass78/manuals/r.clump.html)) all contiguous cells together with a unique id to allow identifying large and small bodies representing the terminal drainage (null in the original DEM).

<span class='terminal'>> r.clump input=drain_terminal output=terminal_clumps --overwrite</span>

Convert the clumps to polygons with the command [r.to.vect](https://grass.osgeo.org/grass78/manuals/r.to.vect.html).

<span class='terminal'>> r.to.vect input=terminal_clumps output=terminal_clumps type=area --overwrite</span>

Add the area of each polygon to the the vector database with the command [v.to.db](https://grass.osgeo.org/grass78/manuals/v.to.db.html).

<span class='terminal'>> v.to.db map=terminal_clumps type=centroid option=area columns=area_km2 units=kilometers</span>

Optionally export the vector with the polygons representing "no data" in the original DEM with [v.out.ogr](https://grass.osgeo.org/grass78/manuals/v.out.ogr.html).

<span class='terminal'>> v.out.ogr input=terminal_clumps type=area format=ESRI_Shapefile output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazoniax/0/terminal-clumps_SRTM_hydro-amazonia_0_cgiar-250.shp --overwrite</span>

### Fill in DEM over land

With the command [v.to.rast](https://grass.osgeo.org/grass78/manuals/v.to.rast.html) it is possible to apply an SQL while converting vectors to a raster. Use that function to assign polygons below an area threshold to be the "no data" to fill, while polygons above the threshold should then represent the true terminal drainage. Set the pixel value to 0 - the preliminary elevation you will assign these cells. In the example I have set the threshold at 2 square kilometers. You should inspect the result and change the threshold if required.

<span class='terminal'>> v.to.rast input=terminal_clumps type=area where="area_km2< 2.0" output=drain_terminal_small use=val value=0 --overwrite</span>

<span class='terminal'>> v.to.rast input=terminal_clumps type=area where="area_km2 >= 2.0" output=drain_terminal_large use=val value=0 --overwrite</span>

if you can not identify a threshold that works you might have to consider manually editing the polygons. For example by deleting individual polygons in <span class='app'>QGIS</span>.

Fill in the small "no data" clumps with the original DEM to create a DEM that is complete over land.

<span class='terminal'>> r.mapcalc "inland_comp_DEM = if(( isnull(DEM@PERMANENT)), drain_terminal_small,DEM@PERMANENT )" --overwrite</span>

Optionally export the completed DEM for land areas:

<span class='terminal'>> r.out.gdal format=GTiff  input=inland_comp_DEM output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazoniax/0/inland-comp-DEM_amazonia_0_cgiar-250.tif --overwrite</span>

The DEM that you just created, inland_comp_DEM is sufficient for use with [r.watershed](https://grass.osgeo.org/grass78/manuals/r.watershed.html) in [part 2](../basin-delineate-02). The remaining part of this post deals with filling up pits. But that is strictly not needed for running [r.watershed](https://grass.osgeo.org/grass78/manuals/r.watershed.html) in [part 2](../basin-delineate-02) and might even have detrimental consequences.

### GRASS shell script

The shell script below contains all the GRASS commands explained above. You can run the commands below by copying them to a file with the extension <span class='file'>.sh</span>. Remember that you have to make the script file executable before you can run it.

```
# Create a map of the terminal drainage

r.mapcalc "drain_terminal = if(isnull('DEM@PERMANENT'), 1, null())"

r.clump input=drain_terminal output=terminal_clumps --overwrite

r.to.vect input=terminal_clumps output=terminal_clumps type=area --overwrite

v.to.db map=terminal_clumps type=centroid option=area columns=area_km2 units=kilometers

# v.out.ogr input=terminal_clumps type=area format=ESRI_Shapefile output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazoniax/0/terminal-clumps_SRTM_hydro-amazonia_0_cgiar-250.shp --overwrite

# Fill in DEM over land

v.to.rast input=terminal_clumps type=area where="area_km2< 2.0" output=drain_terminal_small use=val value=0 --overwrite

v.to.rast input=terminal_clumps type=area where="area_km2 >= 2.0" output=drain_terminal_large use=val value=0 --overwrite

r.mapcalc "inland_comp_DEM = if(( isnull(DEM@PERMANENT)), drain_terminal_small,DEM@PERMANENT )" --overwrite

# r.out.gdal format=GTiff  input=inland_comp_DEM output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazoniax/0/inland-comp-DEM_amazonia_0_cgiar-250.tif --overwrite
```

### Restrict DEM filling to coastal strip

If you want to restrict the pit filling to the coastal strip you need to create a mask ([r.mask](https://grass.osgeo.org/grass78/manuals/r.mask.html)). A mask limits the region for raster operations and will reduce the processing time.

To set a mask restricting the processing to the coastal strip you have to create a new layer for the terminal drainage - this time excluding the cells identified in _drain_terminal_small_.

r.mapcalc "drain_terminal_v2 = if(isnull('inland_comp_DEM'), 1, null())"

<span class='terminal'>> Then do an [r.buffer] starting from version 2 (_v2_) of the terminal drainage, set the maximum distance to the width of the coastal strip you want to analyse.</span>

<span class='terminal'>> r.buffer input=drain_terminal_v2 output=coastal_strip distances=3000 units=meters --overwrite</span>

Apply an [r.mapcalc]() calculation to create a boolean layer of the coastal strip.

<span class='terminal'>> r.mapcalc "coastal_mask = if((coastal_strip > 1), 1, null())"</span>

To apply the mask by [r.mask](https://grass.osgeo.org/grass78/manuals/r.mask.html) you just set it and it will remain in place until explicitly removed.

<span class='terminal'>> r.mask raster=coastal_mask</span>

### Filling pits in the DEM

With all land areas having an assigned elevation, the next task is to fill up pits. As for the filling in of no data above, you again need to set a threshold for the maximum pit size to fill. As a starting point you could set the same threshold as above (2 square kilometers). As noted above, you can not run [r.fill.dir](https://grass.osgeo.org/grass78/manuals/r.fill.dir.html) on large files. I only list the principal steps, to actually run the pit filling, make use of the Python package _basin_extract_ to create the required GRASS commands as eplained further down.

The principal steps for filling up pits in a DEM in GRASS:

- r.fill.dir -f originaldem -> filldem [the f flag forces filling of single cells only]
- r.mapcalc difference between filled and orignal dem
- r.to.vect for cells (points) with difference
- v.db.addcolumn for storing filled DEM value
- v.what.rast extract fill DEM to vector points
- r.fill.dir -f originaldem -> filldem2 [all pit cells filled regardless of area]
- r.mapcalc difference between filled and orignal dem
- r.to.vect for regions (areas) with difference
- v.db.addcolumn for storing filled DEM value
- v.what.rast extract fill DEM to polygon centroids
- v.to.rast rasterize the DEM values
- r.mapcalc superimpose the filled DEM values over the original DEM

There are some alternative commands to use once you have captured all the pits. More about that later.

The process steps above can not be run for very large grids - it takes too long time. But pits are usually small. Larger pits (hundreds of square kilometers or larger) usually represent endorheic basins and should not be filled. Thus you must, and can, partition large DEMs into smaller tiles and then run each tile separately. With an overlap that allows you to capture pits up to the size you deem relevant.

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

### Create tiled process loop

The example DEM used in this tutorial of central South America is approximately 22700 × 22500 cells. This is far too large to process in one go. The Python package developed for assisting the basin delineation, _basin_extract_ can generate a tiled process loop for GRASS.

#### Parameterisation

 The Python script _basin_extractor_ is parameterised from an XML file that follows the general structure of [Karttur´s GeoImagine Framework](https://karttur.github.io/geoimagine/concept/concept-concepts/#xml-coded-parameterization). The structure below shows the setup for generating the GRASS shell script for a tiled processing of pit filling of the DEM over central South America ("Amazonia").

```
<basindelineate>
	<userproj userid = 'karttur' projectid = 'karttur' tractid= 'amazonia' siteid = '*' plotid = '*' system = 'MODIS'></userproj>

	<!-- Define processing -->
	<process processid ='BasinDelineate'>
		<overwrite>Y</overwrite> # Y(es) or N(o)
		<delete>N</delete> # Y(es) or N(o)
		<parameters
		stage = '0' # 0, 1, 2 or 3
		adjacentdist = '330' # any number in map distance units
		outlet = 'SFD' # MOUTH, SFD or MFD
		distill = '' # MFD or None (nothing)
		clusteroutlet = 'central' # central or max
		basinthreshold='2000' # any number is cells
		watershed='MFD5' # SFD, MFD or MFD? (? in range 1..10)
		verbose = '1' # 0, 1 or 2
		proj4CRS = '+proj=sinu +lon_0=0 +x_0=0 +y_0=0 +a=6371007.181 +b=6371007.181 +units=m +no_defs'
		>

		</parameters>
		<srcpath volume = "GRASS2020"></srcpath>
		<dstpath volume = "GRASS2020"></dstpath>

		<srccomp>
			<!-- the name <tag> and prefix must be the same and can not be altered. The script looks for which source data is included and compares that with the parameters, and then sets the processing method if these corresponds. Otherwise the script ends with an error message.
			-->
			<basin-mouth-outlet-pt source = "SRTM" product = "drainage" folder = "basin" band = "basin-mouth-outlet-pt" prefix = "basin-mouth-outlet-pt" suffix = "cgiar-250">
			</basin-mouth-outlet-pt>

			<basin-MFD-outlet-pt source = "SRTM" product = "drainage" folder = "basin" band = "basin-MFD-outlet-pt" prefix = "basin-MFD-outlet-pt" suffix = "cgiar-250">
			</basin-MFD-outlet-pt>

			<basin-SFD-outlet-pt source = "SRTM" product = "drainage" folder = "basin" band = "basin-SFD-outlet-pt" prefix = "basin-SFD-outlet-pt" suffix = "cgiar-250">
			</basin-SFD-outlet-pt>

			<shorewall-pt source = "SRTM" product = "drainage" folder = "basin" band = "shorewall-pt" prefix = "shorewall-pt" suffix = "cgiar-250">
			</shorewall-pt>
		</srccomp>

		<dstcomp>
			<basin-outlet>
			</basin-outlet>
			<!--
			To force a non-default destination path and name, you can enter all components explicitly, e.g.
			<basin-outlet band = "basin-mfd+mfd-max" prefix = "basin-mfd+mfd-max">
			</basin-outlet>
			-->
		</dstcomp>
	</process>

</basindelineate>
```

The core parameters of relevance for this stage (0) include:

- stage [0, 1, 2 or 3] (this part is for stage = '0')
- adjacentdist [not relevant for stage 0]
- outlet [not relevant for stage 0]
- distill [not relevant for stage 0]
- clusteroutlet [not relevant for stage 0]
- basinthreshold [not relevant for stage 0]
- watershed [not relevant for stage 0]
- verbose [0, 1 or 2]
- proj4CRS ['proj4CRS definition for output data if input layers lack proper projection information']

#### Setup xml file and run _basin_extract_

Use the xml template file above to setup the processing for your own xml. Set the parameter _stage_ to _0_ and run _basin_extract_ pointing to the xml.

Running _basin_extract_ with _stage = 0_ generates two shell script files, one for GRASS and one for OGR2OGR. You do not need to use the OGR2OGR file, it is just supplied in case you prefer that.

For each tile that is processed the GRASS shell script includes the following set of commands (excluding comments):

```
# region is automatically set from the tiling process
g.region -a n=-3660170.456407 s=-3914992.450317 e=-8826107.250690 w=-9080929.244600

# the -f flag restricts the filling to single, isolated cells
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
v.out.ogr input=inland_fill_pt_0_0 type=point format=ESRI_Shapefile output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/stage0/inland-fill-pt-0-0.shp --overwrite

# redo the filling but without flag to fill all depressed areas
r.fill.dir input=inland_comp_DEM output=hydro_areafill_DEM_0_0 direction=hydro_areafill_draindir_0_0 areas=hydro_areafill_problems_0_0 --overwrite

# get the difference between the filled and the original DEM
r.mapcalc "DEM_areafill_diff_0_0 = hydro_areafill_DEM_0_0 - inland_comp_DEM" --overwrite

# create boolean mask of changed cells
r.mapcalc "inland_fill_area_0_0 = if(DEM_areafill_diff_0_0 != 0, 1, null() )" --overwrite

# convert the regions with altered elevation to an area vector
r.to.vect input=inland_fill_area_0_0 output=inland_fill_area_0_0 type=area --overwrite

# add the area of each polygon to the vector database
v.to.db map=inland_fill_area_0_0 type=centroid option=area columns=area_km2 units=kilometers

# add column to vector file with altered elevations
v.db.addcolumn map=inland_fill_area_0_0 columns="filldem DOUBLE PRECISION"

# read the altered DEM values to the area vector (the centroid is read)
v.what.rast type=centroid map=inland_fill_area_0_0 column=filldem raster=hydro_areafill_DEM_0_0

# export the area vector to an ESRI shape file
v.out.ogr input=inland_fill_area_0_0 type=area format=ESRI_Shapefile output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/stage0/inland-fill-area-0-0.shp --overwrite
```

#### Superimpose filled DEM over original DEM

All areas identified as pits are captured in the sequence of tiles. All singel cell pits are in a series of point vectors and all larger areas (including the singel cells) are in series of area vectors. You now have several alternatives for superimposing these values over the original DEM. All alternatives can be applied using a tile by tile approach, or applied for the whole region after patching the tiles.

Filling of single cell pits is non-ambiguous, it will not affect the flow routing or hydrological modeling. Whether to fill pits larger than a singel cell is another matter. Filling will lead to flat areas, with their own problems of flow routing. Flat areas will further in general be both more wet and more prone to flooding in the hydrological modeling. Ont he other hand, DEMs generated by SAR data have a tendency to underestimate the elevation of water surface (due to radar signal double bouncing on water surface and shoreline structures). Thus no general recommendation can be given regarding how to handle larger depressions in DEMs.

##### Patching tiles (optional)

If you use GDAL (GDAL_rasterize) for burning the filled DEM on the orignal DEM you do not need to patch the tiles. Just continue to the section on _Rasterize using GDAL_ below. The shell script for the burning is already produced by the Python package _basin_extract_.

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

##### Rasterize using GRASS

It is more complicated to superimpose the filled elevation values using GRASS compare to GDAL. For GRASS you first have to rasterize the vectors and then use [r.mapcalc]() for superimposing. With GRASS you also have to use the patched (complete) vector files. To superimpose the point vector.

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
