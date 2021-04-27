---
layout: post
title: GRASS 7 setup
categories: basin_delineate_setup
excerpt: "Setup GRASS 7 for the first time"
tags:
  - GRASS7
  - start
  - Mac OSX
image: avg-trmm-3b43v7-precip_3B43_trmm_2001-2016_A
modified: '2020-10-03 T06:17:25.000Z'
modified: '2020-10-03 T06:17:25.000Z'
comments: true
share: true
figure1: GRASS7_1stStartup_welcome01
figure1b: GRASS7_Amazonia-Startup_welcome01
figure2: GRASS7_1stStartup_database
figure3: GRASS7_1stStartup_create-location
figure4: GRASS7_1stStartup_define-location
figure5A: GRASS7_1stStartup_import-data
figure5B: GRASS7_1stStartup_import-success
figure6: GRASS7_empty-GUI
figure7: GRASS7_map-display-amazonia-DEM
figure8: GRASS7_empty-terminal

---
<script src="https://karttur.github.io/common/assets/js/karttur/togglediv.js"></script>
# Introduction

This post covers how to start GRASS 7 for the first time. Including defining a GRASS database and a location with a projection. The manual is for GRASS 7 on Mac OSX and is part of a series on how to [delineate river basins](../../) using GRASS GIS and other tools.

The manual only covers the command line GRASS tools for processing. But all the processing can also be done using the GRASS Graphical User Interface (GUI).

## Prerequisites

You must have GRASS 7 installed as described in the post on [Install GDAL, QGIS and GRASS](https://karttur.github.io/setup-ide/setup-ide/install-gis/#grass) (MacOS) or [SPIDE for Ubuntu 20](https://karttur.github.io/setup-ide/blog/ubuntu20-setup-spide/) (Linux).

 You also need a projected raster file (e.g. a GeoTIFF) for defining both the projection and the location.

## Start GRASS

Before you actually start GRASS, you need to start up a <span class='app'>Terminal</span> session. Otherwise GRASS might not start properly. Once you have the <span class='app'>Terminal</span> up and running, locate your installation of GRASS7 and start it. Alternatively you can also start GRASS7 directly from the command line, just type in the full path to the <span class='file'>grass.sh</span> executable script (MacOS):

<span class='terminal'>/Applications/GRASS-7.X.X.app/Contents/MacOS/Grass.sh
</span>

The startup might take a while, and GRASS should then show the window _GRASS GIS 7.x.x Startup_.

<figure>
<img src="{{ site.commonurl }}/images/{{ site.data.images[page.figure1].file }}">
<figcaption> {{ site.data.images[page.figure1].caption }} </figcaption>
</figure>

If the location you want to work with is already defined in your GRASS database, it is listed as a GRASS location (the location _Amazonia_ in the example below). You can then just skip to the last section in this post _Create a new mapset_.

<figure>
<img src="{{ site.commonurl }}/images/{{ site.data.images[page.figure1b].file }}">
<figcaption> {{ site.data.images[page.figure1b].caption }} </figcaption>
</figure>

### Define new GRASS Location

The first time you start GRASS7 you have to set the database directory, and a first Location. As noted on the startup page, in red even, _GRASS needs a directory (GRASS database) in which to store its data ..._. The default database directory is the users home directory. The basin delineation will, however, require a rather large storage space and I recommend using an external disk or server for the data storage.

Use the <span class='button'>Browse</span> button to select the location for your GRASS database. Then use the <span class='button'>New</span> button between the smaller windows for _2 Select GRASS Location_ and _3. Select GRASS Mapset_. GRASS then presents you with a new window, _Define new GRASS Location_.

<figure>
<img src="{{ site.commonurl }}/images/{{ site.data.images[page.figure2].file }}">
<figcaption> {{ site.data.images[page.figure2].caption }} </figcaption>
</figure>

In the example figure above, I created a local disk <span class='file'>GRASS2020</span> and then the directory <span class='file'>GRASSDATA</span> for my GRASS database. I then also defined a _Location_ called _Amazonia_. All GRASS locations will get a default _MapSet_ called _PERMANENT_; let that be your initial mapset for the _Location_ you define.

Do not tick the boxes for _Set default region extent and resolution_ or _Create user mapset_, we will fix that in the next step. Click <span class='button'>Next ></span> to get to the next step.

#### Choose method for creating a new location

A new window, but still belonging to the _Define new GRASS Location_ set, opens. The best option here is to have a projected raster file that you can point at for defining the location. Assuming that you have one, click the radiobutton for that option.

<figure>
<img src="{{ site.commonurl }}/images/{{ site.data.images[page.figure3].file }}">
<figcaption> {{ site.data.images[page.figure3].caption }} </figcaption>
</figure>

Click <span class='button'>Next ></span> and browse to the raster file to use for creating the new location. If you did not select the option to use an existing file for creating the new location, you need to manually add the projection and location.

When you have added all the information required, GRASS will summarise the definition of the new location.

<figure>
<img src="{{ site.commonurl }}/images/{{ site.data.images[page.figure4].file }}">
<figcaption> {{ site.data.images[page.figure4].caption }} </figcaption>
</figure>

Click <span class='button'>Finish</span> and you are done with the definition of database, location and projection of your first GRASS7 project.

### Import raster defining the location

If you choose to define your location using a raster file, like illustrated above, GRASS7 will automatically ask if you want to import this data. In most cases the answer is most likely <span class='button'>yes</span>. Whereupon the file should be successfully imported into the MapSet _PERMANENT_. Click <span class='button'>OK</span> and GRASS7 should start its Graphical User interface.

<figure class='half'>
<img src="{{ site.commonurl }}/images/{{ site.data.images[page.figure5A].file }}">
<img src="{{ site.commonurl }}/images/{{ site.data.images[page.figure5B].file }}">
<figcaption> {{ site.data.images[page.figure5A].caption }} </figcaption>
</figure>

### GRASS Graphical User Interface (GUI)

GRASS7 opens an empty GUI with two windows, a _Map Display_ and a _Layer Manager_.

<figure>
<img src="{{ site.commonurl }}/images/{{ site.data.images[page.figure6].file }}">
<figcaption> {{ site.data.images[page.figure6].caption }} </figcaption>
</figure>

In the _Layer Manager_ find the button for _Add multiple raster or vector map layers_ and click it. Add the raster file you used for defining the location and projection.

<figure>
<img src="{{ site.commonurl }}/images/{{ site.data.images[page.figure7].file }}">
<figcaption> {{ site.data.images[page.figure7].caption }} </figcaption>
</figure>

### GRASS commandline

You can either user GRASS via the GUI, or using the GRASS commandline. If you started GRASS with the <span class='app'>Terminal</span> up and running, you should now have a terminal windown where GRASS is running.

<figure>
<img src="{{ site.commonurl }}/images/{{ site.data.images[page.figure8].file }}">
<figcaption> {{ site.data.images[page.figure8].caption }} </figcaption>
</figure>

#### Commandline tools

Try the commandline tool by checking out the projection:

<span class='terminal'>GRASS 7.2.2 (Amazonia):~ > g.proj -p</span>

The projection information will be returned at the prompt:

```
-PROJ_INFO-------------------------------------------------
name       : Sinusoidal (Sanson-Flamsteed)
a          : 6371007.181
es         : 0
proj       : sinu
lon_0      : 0
x_0        : 0
y_0        : 0
no_defs    : defined
-PROJ_UNITS------------------------------------------------
unit       : meter
units      : meters
meters     : 1
```

Then also check the region:

<span class='terminal'>GRASS 7.2.2 (Amazonia):~ > g.region -p</span>

Note how the returned information specifies the boundaries of the region as well as the spatial resolution and the number of rows and columns (cols) that span the region.

```
projection: 99 (Sinusoidal (Sanson-Flamsteed))
zone:       0
datum:      ** unknown (default: WGS84) **
ellipsoid:  a=6371007.181 es=0
north:      1297275.606933
south:      -3914992.450317
west:       -9080929.2446
east:       -3822329.91573
nsres:      231.6563581
ewres:      231.6563581
rows:       22500
cols:       22700
cells:      510750000
```

#### Rename layer

In the following parts of this series on Basin delineation, the name of the DEM layer to use for delineating basins will be assumed to be _DEM_. If you imported the DEM as part of the definition of your location, it will be given the same name as the original map. If you forgot the name, use the GRASS general command [g.list](https://grass.osgeo.org/grass78/manuals/g.list.html) to list the layers in the current mapset:

 <span class='terminal'>> g.list type=raster</span>

 If your DEM is not listed, make sure you are in the correct mapset with the command [g.mapset](https://grass.osgeo.org/grass78/manuals/g.mapset.html):

<span class='terminal'>> g.mapset -p</span>

<span class='terminal'>> g.mapset mapset=_name_</span>

Rename any layer in your current mapset with the GRASS general command [g.rename](https://grass.osgeo.org/grass78/manuals/g.rename.html):

<span class='terminal'>> g.rename raster=DEM_SRTM_amazonia_0_cgiar-250,DEM</span>

#### Import DEM

To manually import the DEM, and at the same time set a custom name you can use the GRASS command [r.in.gdal](https://grass.osgeo.org/grass78/manuals/r.in.gdal.html). Add the _-e_ flag to set the region to the extent of the imported DEM if not already set to fit this dataset.

<span class='terminal'>> r.in.gdal -e input=/Volumes/GRASS2020/MODIS/SRTM/region/DEM/amazonia/DEM_SRTM_amazonia_0_cgiar-250.tif output=DEM --overwrite
</span>


### Create a new mapset

The DEM raster file that you imported, either when you defined the projection and location, or as a separate raster layer, should have ended up in the mapset _PERMANENT_. For the delineation work, deriving a range of intermediate layers, and finally getting a map of all the basin outlets, it is better to change to a user defined mapset. Use the GRASS command [g.mapset](https://grass.osgeo.org/grass78/manuals/g.mapset.html).

<span class='terminal'>> g.mapset -c drainage</span>

### Exit GRASS

To end the session you need to to exit from the GRASS commandline:

<span class='terminal'>GRASS 7.2.2 (Amazonia):~ > exit</span>
