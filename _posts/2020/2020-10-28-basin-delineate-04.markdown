---
layout: post
title: "Basin delineation: 4 GRASS basin delineation"
categories: basindelineate
excerpt: "Use GRASS7 for delineating river basins"
tags:
  - GRASS7
  - watershed
  - basin
  - delineation
  - hydrology
image: avg-trmm-3b43v7-precip_3B43_trmm_2001-2016_A
modified: '2020-10-28 T06:17:25.000Z'
modified: '2020-10-28 T06:17:25.000Z'
comments: true
share: true
figure5a: GRASS7_Amazonia-drainage-SFD
figure5b: GRASS7_Amazonia-drainage-MFD
figure6: GRASS7_Amazon-River-drainage-SFD-MFD

---
<script src="https://karttur.github.io/common/assets/js/karttur/togglediv.js"></script>

# Introduction

In this part (4) of the of the Basin delineation tutorial series you will run a GRASS command line script generated in [part 3](../basin-delineate-03). The script delineates a vector polygon for each distilled outlet point generated from [part 3](../basin-delineate-03). Then another command line script will call GDAL for assembling all polygons into a single data source. You need to manually inspect this file to determine if and what editing is required.

if you use the default 'MOUTH' process alternative, the script will first manipulate the input Digital Elevation Model (DEM) to better represented the flow path in river mouths and then use the manipulated DEM for identifying the extent of all larger river basins.

## Prerequisites

You must have setup GRASS 7 and imported a DEM as described in [Installation & Setup](../../basindelineatesetup). Then you must have completed the GRASS watershed processing as outlined in [part 2](../basin-delineate-02) and run the Python script _Basin_extractor_ as described in [part 3](../basin-delineate-03).

### MOUTH processing

if you set the parameterization of _basin_extract_ in [part 3](../basin-delineate-03) to _outlet='MOUTH'_, the generated GRASS command line script will be called <span class='file'>"region"\_grass_basin_mouth2.sh</span>, where _region_ is the study area. It will be saved in your project data directory where all other spatial datasets are exported. As defined in the xml file and following the general structure of KArttur's GeoImagine Framework.

The GRASS shell script file start with commands for creating a virtual DEM and then rerunning [r.watershed](). Optionally the user can export intermediate raster layers by removing the '#' sign. The generated commands for the Amazonia region used as example in this tutorial series are shown below. Your version will be similar, but with the regional identifier (_amazonia_ in the example) replaced by your study region identifier. Also other parts of the file names may differ.
```
# Creating virtual DEM for directing hydrological flow in river basin mouths

# Import outlet points vector
v.in.ogr -o input=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/basin-mouth2-outlet_drainage_amazonia_0_cgiar-250.shp output=basin_distil_mouth2_shorewall_pt --overwrite

# rasterize outlet points
v.to.rast input=basin_distil_mouth2_shorewall_pt output=basin_distil_mouth2_shorewall_pt type=point use=val value=1 --overwrite

# Add outlet points to river mouth DEM with all cells=1
r.mapcalc "basin_distil_mouth2_lowlevel_DEM = if(isnull(basin_distil_mouth2_shorewall_pt),lowlevel_outlet_costgrow+1,basin_distil_mouth2_shorewall_pt)" --overwrite

# cost grow analysis from new outlets over river mouth DEM
r.cost -n input=basin_distil_mouth2_lowlevel_DEM output=basin_distil_mouth2_mouth_dist start_points=basin_distil_mouth2_shorewall_pt max_cost=228 --overwrite

# export cost grow analysis (optional)
# r.out.gdal -f input=basin_distil_mouth2_mouth_dist format=GTiff type=Int16 output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/basin-distil-mouth2-mouth-dist_drainage_amazonia_0_cgiar-250.tif --overwrite

# Invert cost grow to create mouth flow route DEM directing flow towards outlet point
r.mapcalc "basin_distil_mouth2_routing_dem = int(basin_distil_mouth2_mouth_dist-230)" --overwrite

# export the mouth flow route DEM (optional)
# r.out.gdal -f input=basin_distil_mouth2_routing_dem format=GTiff type=Int16 output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/basin-distil-mouth2-routing-dem_drainage_amazonia_0_cgiar-250.tif --overwrite

# combine mouth flow route DEM with original DEM and add the shorewall
r.mapcalc "basin_distil_mouth2_basin_dem = if(isnull(basin_distil_mouth2_routing_dem),(if(isnull(DEM@PERMANENT),shorewall,DEM@PERMANENT)),basin_distil_mouth2_routing_dem)" --overwrite

# export the hydrological corrected DEM (optional)
# r.out.gdal -f input=basin_distil_mouth2_basin_dem format=GTiff type=Int16 output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/basin-distil-mouth2-basin-dem_drainage_amazonia_0_cgiar-250.tif --overwrite

# run r.watershed for the new outlet points and the virtual (hydrologically corrected) DEM
r.watershed -a convergence=5 elevation=basin_distil_mouth2_basin_dem accumulation=basin_distil_mouth2_wshed_acc drainage=basin_distil_mouth2_wshed_draindir_draindir threshold=2000

# convert updrain accumulation raster to byte format (optional)
# r.mapcalc "basin_distil_mouth2_wshed_acc_ln = 10*log(basin_distil_mouth2_wshed_acc)" --overwrite

# Set color ramp for upstream accumulation raster (optional)
# set color ramp for byte version of updrain accumulationr.colors map=basin_distil_mouth2_wshed_acc_ln color=ryb

# export the visualised upstream accumulation raster (optional)
# r.out.gdal -f input=basin_distil_mouth2_wshed_acc_ln format=GTiff type=Byte output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/basin-distil-mouth2-wshed-acc-ln_drainage_amazonia_0_cgiar-250.tif --overwrite

# add column "updrain" to the new outlet vector
v.db.addcolumn map=basin_distil_mouth2_shorewall_pt columns="updrain DOUBLE PRECISION"

# extract data from r.watersehd updrain to the column "updrain" in the outlet point map
v.what.rast map=basin_distil_mouth2_shorewall_ptc column=updrain raster=basin_distil_mouth2_wshed_acc
```

Run the shell script from your GRASS terminal with the active _mapset_ being the same as when you processed the data in [part 2](../basin-delineate-02). Before you can run the commands you have to change the permission of the shell file (<span class='file'>.sh</span>) itself. To change the permissions to an executable script, type at the terminal:

<span class='terminal'>> chmod 777 "region"_grass_basin_mouth2.sh </span>.

Tu run the file you need to give either the relative or full path, where the relative path would be:

<span class='terminal'>> ./"region"_grass_basin_mouth2.sh </span>.

The full path depends on where you put the GRASS location.

You can run the script now, as it will take several hours or even days to run. But that is not because of the commands above, but the commands following that create polygons of full river basins for each outlet point.

### GRASS script for basin polygon delineation

For each identified outlet point from the Python script _basin_extract_ in [part 2](../basin-delineate-02) the delineation of the associated river basin contains the following steps:

- identify all upstream cells from the outlet coordinates
- convert the identified cell region to a polygon
- clean the polygon form errors
- build the polygon attribute table
- export the polygon vector to an ESRI shape file

To save disk space, the raster version of the basin, created in the very first step, is deleted once the raster is vectorized in the second step. The actual GRASS commands for each outlet looks like the example below:

```
r.water.outlet input=basin_distil_mouth2_wshed_draindir output=basin_mouth2_000001 coordinates=-7928554.691232,1295538.184247 --overwrite
r.to.vect input=basin_mouth2_000001 output=basin_mouth2_000001 type=area --overwrite
g.remove -f type=raster name=basin_mouth2_000001 --quiet
v.clean input=basin_mouth2_000001 output=basin_mouth2_000001c type=area tool=prune,rmdupl,rmbridge,rmline,rmdangle thresh=0,0,0,0,-1 --overwrite
v.db.addcolumn map=basin_mouth2_000001c columns="x_mouth DOUBLE PRECISION"
v.db.addcolumn map=basin_mouth2_000001c columns="y_mouth DOUBLE PRECISION"
v.db.update map=basin_mouth2_000001c column=x_mouth value=-7928554.691232
v.db.update map=basin_mouth2_000001c column=y_mouth  value=1295538.184247
v.db.addcolumn map=basin_mouth2_000001c columns="mouth_id INT"
v.db.addcolumn map=basin_mouth2_000001c columns="basin_id INT"
v.db.update map=basin_mouth2_000001c column=mouth_id  value=1
v.db.update map=basin_mouth2_000001c column=basin_id  value=1
v.to.db map=basin_mouth2_000001c type=centroid option=area columns=area_km2 units=kilometers
v.out.ogr input=basin_mouth2_000001c type=area format=ESRI_Shapefile output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazoniax/0/basin_mouth2_000001c.shp --overwrite
```

As there can be thousands of outlet points the processing of the entire shell script might take a long time. But progress is reported int he <span class='app'>Terminal</span> window, indicating the number of the outlet under process. And as the outlets are sequential numbers you can always find out how far the processing has gone.

### Assembling basin polygons to a single shape file

Looking at each polygon vector representing a river basin individually is not doable. To collect all the produced polygons into a single shape file, the Python package in [part 2](../basin-delineate-02) produced another shell script, <span class='file'>"region"\_ogr2ogr_basin_mouth2.sh</span>. This script requires that you have installed GDAL (see [Installation & Setup](../basindelineatesetup)). The command _ogr2ogr_ is the vector processing tool of GDAL. The script simply assembles all the polygons into single vector file. In the first line, the first polygon is simply copied to a new file, all remaining commands then _append_ data to the first file.

```
ogr2ogr -skipfailures -nlt MULTIPOLYGON /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/basin-mouth2-outlet-area_drainage_amazonia_0_cgiar-250.shp /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazoniax/0/basin_mouth2_000001c.shp
ogr2ogr  -append -skipfailures -nlt MULTIPOLYGON /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/basin-mouth2-outlet-area_drainage_amazonia_0_cgiar-250.shp /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazoniax/0/basin_mouth2_000002c.shp
ogr2ogr  -append -skipfailures -nlt MULTIPOLYGON /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/basin-mouth2-outlet-area_drainage_amazonia_0_cgiar-250.shp /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazoniax/0/basin_mouth2_000003c.shp
```

the script will only take a few minutes, and then you should have a new ESRI shape file with all the delineated basins.

### Inspect
