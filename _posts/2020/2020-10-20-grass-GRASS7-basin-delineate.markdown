---
layout: post
title: REMOVE THIS POST
categories: grass_setup
excerpt: "River basin delineation in GRASS7"
tags:
  - GRASS7
  - basin
  - delineation
image: avg-trmm-3b43v7-precip_3B43_trmm_2001-2016_A
modified: '2020-10-20 T06:17:25.000Z'
modified: '2020-10-20 T06:17:25.000Z'
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
# Introduction

This post is the first in a series of how to use a combination of Python scripting and different GIS applications, mainly GRASS, for delineating basins from Digital Elevation Models (DEMs). The first part is a manual on how to get GRASS 7 setup and started with a DEM.

#### Watershed accumulation and basin area

The GRASS raster command [r.watershed](https://grass.osgeo.org/grass78/manuals/r.watershed.html) calculates a variety of hydrological parameters, including the upstream (accumulation) area of each cell. Note that the accumualtion area is calculacated as number of cells. [r.watershed](https://grass.osgeo.org/grass78/manuals/r.watershed.html) can be run assigning a Single Flow Direction (SFD) out of each cell, or a Multiple Flow Direction (MFD). MFD is in general preferred and also the default. MFD can be parameterised for determining the flow convergence/divergence, with the recommended default being an average. If you want to avoid negative numbers for upstream area, set the _-a_ flag. [r.watershed](https://grass.osgeo.org/grass78/manuals/r.watershed.html) can be set up to produce a range of hydrologically related layers. For delineating basins you must produce layers for _accumulation_ and _drainage_.

To apply an SFD flow routing, set the flag _-s_. In the example below I have set both the _-a_ and _-s_ flags, and I have requested the production of layers for _accumulation_ and _drainage_. I have set the cutoff threshold at 2000 cells:

<span class='terminal'>> r.watershed -as elevation=DEM@PERMANENT accumulation=SFD_upstream drainage=SFD_drainage threshold=2000 --overwrite</span>

```
SECTION 1a (of 5): Initiating Memory.
SECTION 1b (of 5): Determining Offmap Flow.
 100%
SECTION 2: A* Search.
 100%
SECTION 3: Accumulating Surface Flow with SFD.
 100%
SECTION 4: Watershed determination.
 100%
SECTION 5: Closing Maps.
Writing out only positive flow accumulation values.
Cells with a likely underestimate for flow accumulation can no longer be
identified.
```

Omitting the _-s_ flag causes [r.watershed](https://grass.osgeo.org/grass78/manuals/r.watershed.html) to run in default mode, applying an MFD flow routing.

<span class='terminal'>> r.watershed -a elevation=DEM@PERMANENT accumulation=MFD_upstream drainage=MFD_drainage threshold=2000 --overwrite</span>

```
SECTION 1a (of 5): Initiating Memory.
SECTION 1b (of 5): Determining Offmap Flow.
 100%
SECTION 2: A* Search.
 100%
SECTION 3a: Accumulating Surface Flow with MFD.
 100%
SECTION 3b: Adjusting drainage directions.
 100%
SECTION 4: Watershed determination.
 100%
SECTION 5: Closing Maps.
Writing out only positive flow accumulation values.
Cells with a likely underestimate for flow accumulation can no longer be
identified.
```

If you want to capture the entire mouths of wider rivers, you should set a smaller _convergence_ factor for MFD (default = 5)

<span class='terminal'>> r.watershed -a  elevation=DEM@PERMANENT accumulation=MFD_upstream drainage=MFD_drainage threshold=2000 --overwrite</span>


#### Beautify flat areas (wide channels)

<span class='terminal'>> r.watershed -ab convergence=1 elevation=DEM@PERMANENT accumulation=MFDab_upstream drainage=MFDab_drainage threshold=2000 --overwrite</span>

#### Maximum divergence

<span class='terminal'>> r.watershed -a convergence=1 elevation=DEM@PERMANENT accumulation=MFD01_upstream drainage=MFD01_drainage threshold=2000 --overwrite</span>

#### Beautify flat areas with maximum divergence

<span class='terminal'>> r.watershed -ab convergence=1 elevation=DEM@PERMANENT accumulation=MFDab01_upstream drainage=MFDab01_drainage threshold=2000 --overwrite</span>

#### Maximum divergence 4 directions

<span class='terminal'>> r.watershed -4a convergence=1 elevation=DEM@PERMANENT accumulation=MFD4a01_upstream drainage=MFD_drainage threshold=2000 --overwrite</span>

If you look at the accumulation layer, the major Rivers (e.g. the Amazon River) should have the largest accumulation values (see figure 5).

#### Export upstream layers

You will need the raster layers showing the upstream accumulated areas for inspecting the processing. The upstream layer, however, contains very high numbers and is difficult to visualise. Thus it is better to first get the natural logarithm of the dataset and then export it with byte format. Apply the logarithmic conversion, including a multiplication wth 10, with the GRASS command [r.mapcalc](https://grass.osgeo.org/grass78/manuals/r.mapcalc.html). Set a colour ramp that is intuitive for interpreting water flow with the command [r.colors](https://grass.osgeo.org/grass78/manuals/r.out.gdal.html). I choose red-yellow-blue (ryb) with a histogram equalisation (-e flag). Then export the raster layer with the command [r.out.gdal](https://grass.osgeo.org/grass78/manuals/r.out.gdal.html).

<span class='terminal'>> r.mapcalc "SFD_ln_upstream = 10*log(SFD_upstream)" --overwrite</span>

<span class='terminal'>> r.colors -e map=SFD_ln_upstream color=ryb</span>

<span class='terminal'>> r.colors map=SFD_ln_upstream color=ryb</span>

<span class='terminal'>> r.out.gdal -f input=SFD_ln_upstream format=GTiff type=Byte output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/upstream-ln-ryb-SFD_amazonia_0_cgiar-250.tif --overwrite</span>

<span class='terminal'>> r.mapcalc "MFD_ln_upstream = 10* log(MFD_upstream)" --overwrite</span>

<span class='terminal'>> r.colors map=MFD_ln_upstream color=ryb</span>

<span class='terminal'>> r.out.gdal -f input=MFD_ln_upstream format=GTiff type=Byte output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/upstream-ln-ryb-MFD_amazonia_0_cgiar-250.tif --overwrite</span>

#### Beautify flat areas (wide channels)

<span class='terminal'>> r.mapcalc "MFDab_ln_upstream = 10* log(MFDab_upstream)" --overwrite</span>

<span class='terminal'>> r.colors map=MFDab_ln_upstream color=ryb</span>

<span class='terminal'>> r.out.gdal -f input=MFDab_ln_upstream format=GTiff type=Byte output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/upstream-ln-ryb-MFDab_amazonia_0_cgiar-250.tif --overwrite</span>

#### Maximum divergence

<span class='terminal'>> r.mapcalc "MFD01_ln_upstream = 10* log(MFD01_upstream)" --overwrite</span>

<span class='terminal'>> r.colors map=MFD01_ln_upstream color=ryb</span>

<span class='terminal'>> r.out.gdal -f input=MFD01_ln_upstream format=GTiff type=Byte output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/upstream-ln-ryb-MFD01_amazonia_0_cgiar-250.tif --overwrite</span>

#### Beautify flat areas with maximum divergence

<span class='terminal'>> r.mapcalc "MFDab01_ln_upstream = 10*log(MFDab01_upstream)" --overwrite</span>

<span class='terminal'>> r.colors map=MFDab01_ln_upstream color=ryb</span>

<span class='terminal'>> r.out.gdal -f input=MFDab01_ln_upstream format=GTiff type=Byte output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/upstream-ln-ryb-MFDab01_amazonia_0_cgiar-250.tif --overwrite</span>

#####

<span class='terminal'>> r.mapcalc "MFD4a01_ln_upstream = 10*log(MFD4a01_upstream)" --overwrite</span>

<span class='terminal'>> r.colors map=MFD4a01_ln_upstream color=ryb</span>

<span class='terminal'>> r.out.gdal -f input=MFD4a01_ln_upstream format=GTiff type=Byte output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/upstream-ln-ryb-MFD4a01_amazonia_0_cgiar-250.tif --overwrite</span>

#### Identify terminal water body

A powerful raster commands in GRASS is the raster map calculator [r.mapcalc](https://grass.osgeo.org/grass78/manuals/r.mapcalc.html). You can use [r.mapcalc](https://grass.osgeo.org/grass78/manuals/r.mapcalc.html) to create a layer where the terminal water body, into which the basins you want to delineate, drain. This is equal to setting all land areas (cells in your DEM layer with valid data) to NULL, and your terminal water body (e.g. the areas assigned with NULL in the DEM) to a value of 1.

<span class='terminal'>> r.mapcalc "drain_terminal = if(isnull('DEM@PERMANENT'), 1, null())"</span>

#### Get shoreline

With the terminal water body identified, the shoreline with all the basin outlets will all be found in the neighbouring cells of the water body. In GRASS you can use the proximity analysis tool [r.buffer](https://grass.osgeo.org/grass78/manuals/r.buffer.html) to identify the coastline from the terminal water body. To capture all shoreline cells set the distance to the length of the side of one cell * sqrt of 2 + plus a little extra. The spatial resolution of the DEM used in this example is 231.6 m. I thus set the distance threshold for the proximity analysis to 330 m.

<span class='terminal'>> r.buffer input=drain_terminal output=shoreline distances=330 units=meters --overwrite</span>

#### Extract accumulated drainage for shoreline

With both the shoreline and the flow accumulation identified on a per pixel level, you can once again apply [r.mapcalc](https://grass.osgeo.org/grass78/manuals/r.mapcalc.html), but this time for mask out the layer with the accumulated upstream area to only include the coastline (i.e. the potential basin outlets).

<span class='terminal'>> r.mapcalc "coastline_SFD_flowacc = if((shoreline > 1), SFD_upstream, null())"</span>

<span class='terminal'>> r.mapcalc "coastline_MFD_flowacc = if((shoreline > 1), MFD_upstream, null())"</span>

#### Threshold minimum drainage area

The map of the upstream (accumulated) cells masked to show only the shoreline contains the upstream area of every cell along the coastline, regardless of the upstream area. But you only want the river basins, not the cells that empty directly into the terminal water body or via insignificant water courses. You need to [r.reclass](https://grass.osgeo.org/grass78/manuals/r.reclass.html) the raster layer showing the upstream area of the coastline to retain only the cells above a given threshold. Remember that the accumulated upstream drainage is given in cells. [r.reclass](https://grass.osgeo.org/grass78/manuals/r.reclass.html) can be parameterized in different ways, but to keep a memory of the processing it is best to do via a simple text file that is linked in the command. To retain all basins composed to 2000 cells or more, the file looks like this:

```
2000 thru 99999999 = 1
* = NULL
```

In the example I have set the threshold to 2000. As the map has a spatial resolution of 231.6 m, this equals approximately 100 (107 to be exact) square kilometres. Then run [r.reclass](https://grass.osgeo.org/grass78/manuals/r.reclass.html) with the text file used for parameterization.

<span class='terminal'>> r.reclass input=coastline_SFD_flowacc output=basin_SFD_outlets rules='/Volumes/GRASS2020/GRASSsupport/reclass/reclass_flow_acc_2000.txt' --overwrite</span>

<span class='terminal'>> r.reclass input=coastline_MFD_flowacc output=basin_MFD_outlets rules='/Volumes/GRASS2020/GRASSsupport/reclass/reclass_flow_acc_2000.txt' --overwrite</span>

#### Vectorize basin outlet candidates

You now have a raster layer with candidate basin outlets. To clean the candidates for redundancies and later actually use the accepted candidates for basin delineation, you need to have the outlet positions as point vectors. In GRASS you convert raster cells to vector points with the command [r.to.vect](https://grass.osgeo.org/grass78/manuals/r.to.vect.html).

<span class='terminal'>> r.to.vect input=basin_SFD_outlets output=basin_SFD_outlet_pt type=point</span>

<span class='terminal'>> r.to.vect input=basin_MFD_outlets output=basin_MFD_outlet_pt type=point</span>

<figure class="half">
	<a href="{{ site.commonurl }}/images/{{ site.data.images[page.figure5a].file }}"><img src="{{ site.commonurl }}/images/{{ site.data.images[page.figure5a].file }}" alt="image"></a>
  <a href="{{ site.commonurl }}/images/{{ site.data.images[page.figure5b].file }}"><img src="{{ site.commonurl }}/images/{{ site.data.images[page.figure5b].file }}" alt="image"></a>
	<figcaption>Upstream area and identified candidate basin outlets using Single Flow Direction (SFD) and Multi Flow Direction (MFS). The large river channel is the Amazon River. Note the many more identified basin outlet candidates when applying the MFD algorithm. The wider spread of the accumulated flow with MFD (right) causes the wide river mouths show up. The accumulated flow with SFD forces flow to a channel with a width of only 1 cell - and is not really seen in the map at the presented scale.
  The next figure shows the mouth of the Amazon River in detail.</figcaption>
</figure>

<figure>
<img src="{{ site.commonurl }}/images/{{ site.data.images[page.figure6].file }}">
<figcaption> {{ site.data.images[page.figure6].caption }} </figcaption>
</figure>

#### Add tables to vector databases

Use GRASS vector command [v.db.addcolumn](https://grass.osgeo.org/grass78/manuals/v.db.addcolumn.html) to add columns to the vector attribute tables. The columns will be used for storing the cell data on upstream area and drainage direction.

<span class='terminal'>> v.db.addcolumn map=basin_SFD_outlet_pt columns='upstream DOUBLE PRECISION'</span>

<span class='terminal'>> v.db.addcolumn map=basin_SFD_outlet_pt columns='drainage INT'</span>

<span class='terminal'>> v.db.addcolumn map=basin_SFD_outlet_pt columns='elevation INT'</span>

or with multiple columns added in one command:

<span class='terminal'>> v.db.addcolumn map=basin_MFD_outlet_pt columns='upstream DOUBLE PRECISION, drainage INT, elevation INT'</span>

#### Add basin data to vector databases

To transfer the cell data underlying each point to the vector attribute table, use the GRASS vector command [v.what.rast](https://grass.osgeo.org/grass78/manuals/v.what.rast.html).

<span class='terminal'>> v.what.rast map=basin_SFD_outlet_pt raster=SFD_upstream column=upstream</span>

<span class='terminal'>> v.what.rast map=basin_SFD_outlet_pt raster=SFD_drainage column=drainage</span>

<span class='terminal'>> v.what.rast map=basin_MFD_outlet_pt raster=MFD_upstream column=upstream</span>

<span class='terminal'>> v.what.rast map=basin_MFD_outlet_pt raster=MFD_drainage column=drainage</span>

#### Upload the x and y coordinates to the vector db

Some inherent vector properties can be added to the vector database without prior definition of the column. The command [v.to.db](https://grass.osgeo.org/grass78/manuals/v.to.db.html) creates the column(s) on the fly, for instance for adding point coordinates:

<span class='terminal'>> v.to.db map=basin_SFD_outlet_pt option=coor columns=xcoord,ycoord</span>

<span class='terminal'>> v.to.db map=basin_MFD_outlet_pt option=coor columns=xcoord,ycoord</span>

#### Export basin outlet candidates

Distillation of basin outlets from the candidates is done using a Python script and requires shape files as input. Export the outlet candidates with [v.out.ogr](https://grass.osgeo.org/grass78/manuals/v.out.ogr.html). Start by creating the target directory, and then export the GRASS vectors as a shape files:

<span class='terminal'>> mkdir -p /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0</span>

<span class='terminal'>> v.out.ogr type=point input=basin_SFD_outlet_pt format=ESRI_Shapefile output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/basin-SFD-outlet-pt_drainage_amazonia_0_cgiar-250.shp --overwrite</span>

<span class='terminal'>> v.out.ogr type=point input=basin_SFD_outlet_pt format=ESRI_Shapefile output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/basin-MFD-outlet-pt_drainage_amazonia_0_cgiar-250.shp --overwrite</span>

### GRASS shell script I

The complete GRASS processing chain after importing the DEM and creating a new mapset, can be run as a shell script.

```
# Watershed accumulation and basin area

r.watershed -as elevation=DEM@PERMANENT accumulation=SFD_upstream drainage=SFD_drainage threshold=2000 --overwrite

r.watershed -a elevation=DEM@PERMANENT accumulation=MFD_upstream drainage=MFD_drainage threshold=2000 --overwrite

# Identify terminal water body

r.mapcalc "drain_terminal = if(isnull('DEM@PERMANENT'), 1, null())"

# Get shoreline

r.buffer input=drain_terminal output=shoreline distances=330 units=meters --overwrite

# Extract accumulated drainage for shoreline

r.mapcalc "coastline_SFD_flowacc = if((shoreline > 1), SFD_upstream, null())"
r.mapcalc "coastline_MFD_flowacc = if((shoreline > 1), MFD_upstream, null())"

# Threshold minimum drainage area

# Create reclass parameterization file
### 2000 thru 99999999 = 1 ###
### * = NULL ###

r.reclass input=coastline_SFD_flowacc output=basin_SFD_outlets rules='/Volumes/GRASS2020/GRASSsupport/reclass/reclass_flow_acc_2000.txt' --overwrite

r.reclass input=coastline_MFD_flowacc output=basin_MFD_outlets rules='/Volumes/GRASS2020/GRASSsupport/reclass/reclass_flow_acc_2000.txt' --overwrite

# Vectorize basin outlet candidates

r.to.vect input=basin_SFD_outlets output=basin_SFD_outlet_pt type=point

r.to.vect input=basin_MFD_outlets output=basin_MFD_outlet_pt type=point

# Add tables to vector databases

v.db.addcolumn map=basin_SFD_outlet_pt columns='upstream DOUBLE PRECISION'

v.db.addcolumn map=basin_SFD_outlet_pt columns='drainage INT'

# or with multiple columns added in one command

v.db.addcolumn map=basin_MFD_outlet_pt columns='upstream DOUBLE PRECISION, drainage INT'

# Add basin data to vector databases

v.what.rast map=basin_SFD_outlet_pt raster=SFD_upstream column=upstream

v.what.rast map=basin_SFD_outlet_pt raster=SFD_drainage column=drainage

v.what.rast map=basin_MFD_outlet_pt raster=MFD_upstream column=upstream

v.what.rast map=basin_MFD_outlet_pt raster=MFD_drainage column=drainage

# Upload the x and y coordinates to the vector db

v.to.db map=basin_SFD_outlet_pt option=coor columns=xcoord,ycoord

v.to.db map=basin_MFD_outlet_pt option=coor columns=xcoord,ycoord

# Export basin outlet candidates

mkdir -p /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0

v.out.ogr type=point input=basin_SFD_outlet_pt format=ESRI_Shapefile output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/basin-SFD-outlet-pt_drainage_amazonia_0_cgiar-250.shp --overwrite

v.out.ogr type=point input=basin_MFD_outlet_pt format=ESRI_Shapefile output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/basin-MFD-outlet-pt_drainage_amazonia_0_cgiar-250.shp --overwrite

```

### Python processing

Once you have exported the candidate basin outlets, the next step is to select which outlet points to use for the initial basin identification and then construct the GRASS script file for generating the initial basins. For this I have created a Python script called [basin_extractor]() available as part of Karttur's GeoImagine Framework.

#### Alternative basin distillations I

You will most certainly not be able to pull out all basins without some manual inspections and iterative joining and/or additions of missing basins. Then you also need to think about how you want to represent river mouths - i.e. if you want to include the full widht of river mouths wider than the cell size of your DEM. There are 4 basic alternatives for how to distill your candidate basin outlet points to get a final map of basins:

1. Use all Multiple Flow Direction (MFD) identified outlets for generating initial basins,
2. Cluster adjacent MFD outlets based on the MFD candidate outlets and distill to a single outlet per cluster,
3. Use all Single Flow Direction (SFD) identified outlets for generating initial basins,
4. Cluster SFD outlets based on adjacent MFD candidate outlets and distill to a single SFD outlet per cluster.

The cluster algorithms can retain either the most centrally positioned candidate or the candidate with highest registered accumulation area. The best result is achieved by combining two or more of the alternative parameterisations, as explained further down.

#### Parameterisation

 The Python script _basin_extractor_ is parameterised from an XML file (the general structure of the xml file follows [Karttur´s GeoImagine Framework standard](https://karttur.github.io/geoimagine/concept/concept-concepts/#xml-coded-parameterization)):

```
<basindelineate>
	<userproj userid = 'karttur' projectid = 'karttur' tractid= 'amazonia' siteid = '*' plotid = '*' system = 'MODIS'></userproj>

	<!-- Define processing -->
	<process processid ='BasinDelineate'>
		<overwrite>Y</overwrite>
		<delete>N</delete>
		<parameters
		stage = '1'
		threshholddist = '660'
		outlet = 'SFD'
		distill = ''
		clusteroutlet = 'central'
		verbose = '1'
		proj4CRS = '+proj=sinu +lon_0=0 +x_0=0 +y_0=0 +a=6371007.181 +b=6371007.181 +units=m +no_defs'
		>

		</parameters>
		<srcpath volume = "GRASS2020"></srcpath>
		<dstpath volume = "GRASS2020"></dstpath>

		<srccomp>
			<basin-MFD-outlet-pt source = "SRTM" product = "drainage" folder = "basin" band = "basin-MFD-outlet-pt" prefix = "basin-MFD-outlet-pt" suffix = "cgiar-250">
			</basin-MFD-outlet-pt>
			<basin-SFD-outlet-pt source = "SRTM" product = "drainage" folder = "basin" band = "basin-SFD-outlet-pt" prefix = "basin-SFD-outlet-pt" suffix = "cgiar-250">
			</basin-SFD-outlet-pt>
			<!--
			To force add outlet points use the tag <basin-ADD-outlet-pt>
			<basin-ADD-outlet-pt source = "SRTM" product = "drainage" folder = "basin" band = "basin-SFD-outlet-pt" prefix = "basin-SFD-outlet-pt" suffix = "cgiar-250">
			</basin-DD-outlet-pt>
			-->
		</srccomp>

		<dstcomp>
			<basin-outlet>
			</basin-outlet>
			<!--
			To force a non-default destination path and name, you can enter all components explicitly
			<basin-outlet band = "basin-mfd+mfd-max" prefix = "basin-mfd+mfd-max">
			</basin-outlet>
			-->
		</dstcomp>
	</process>

</basindelineate>
```

The core parameters include:

- stage [1, 2 or 3]
- threshholddist [the maximum distance in map units used for clustering candidate outlet points]
- outlet ['SFD' or 'MFD']
- distill ['MFD' or 'None']
- clusteroutlet ['central' or 'maximum']
- verbose [0, 1 or 2]
- proj4CRS ['proj4CRS definition for output data if input layers lack projection information']

The script _basin_extractor_ will produce a point vector file in shape format containing the selected outlet candidates points as requested in the parameterization (alternatives 1 to 4 above). Alongside the point vector file, the script also produces a shell script (<span class='file'>.sh</span>) with the GRASS commands for delineating the basins from the selected candidate points. The name of the shell script file is defaulted to reflect which of the four main alternatives the parameterisation was set to.

1. 1
2. 2

#### Selected candidate vectors - example

The figure below illustrates a small portion of the Amazonia region, and the selected candidate points from the four alternative parameterizations above.

FIGURE

#### Alternative basin distillations II

Alternative distillation processes for identifying initial basin outlet obviously generate different sets of outlets. Picking one of them depends on the objective of your final basin map. It you are going to model water flow, then you shoud use the same flow routing (SFD vs MFD) for defining you basin as you will use for the hydrological modeling. If you want to portray river mouths as single points you should go for SFD. If you want a dataset where you can capture the full outflow from rivers with mouths wider than the cell size, you need to apply an MFD.

The quickest and simplest solution is to use an SFD solution without clustering (with parametrisation: _outlet_ = 'SFD', _distill_ = 'None').

To remove rivers branches that represent bifurcations, use SFD outlets distilled though MFD clustering (with parametrisation: _outlet_ = 'SFD', _distill_ = 'MFD').

If your objective is hydrological modeling, it is more likely you need an MFD approach. If you are not considering the outflow from the entire basin, an MFD clustered approach is the best (with parametrisation: _outlet_ = 'MFD', _distill_ = 'MFD'). If, on the other hand, you want to capture the outflow from the entire basin, you need to include all identified MFD outlets (with parametrisation: _outlet_ = 'MFD', _distill_ = 'None'). This alternative takes the longest to run and also requires the most post processing (in the GRASS part II step outlined in the next section that is).

## GRASS part II

The main output of stage 1 from the Python script _basin_extract_ is a GRASS command line script.

Regardless of how you parameterised the Python script _basin_extractor_ the generated GRASS shell script will have the same sequence of commands for each basin to delineate.

### GRASS shell script II

The ...

##### r.water.outlet

The basin as such is generated with the GRASS command [r.water.outlet](https://grass.osgeo.org/grass78/manuals/r.water.outlet.html) using the _drainage_ direction output from [r.watershed](https://grass.osgeo.org/grass78/manuals/r.watershed.html) and the _coordinates_ of the outlet point as input parameters:

<span class='terminal'>> r.water.outlet input=SFD_drainage output=SFD_1 coordinates=-7928554.691232,1295074.871531 --overwrite</span>

This generates a raster layer identifying the cells upstreams of the coordinates (the identified basin outlet point).

##### r.null

In order to convert the raster basin to a vector, set all cells falling outside the basin to NULL with [r.null](https://grass.osgeo.org/grass78/manuals/r.null.html). In GRASS version 7 this is not needed as the cells outside the identified basin are set to NULL by default.

<span class='terminal'>> r.null SFD_1 setnull=0</span>

#### r.to.vect

Convert the identified raster basin to a polygon vector with the GRASS command [r.to.vect](https://grass.osgeo.org/grass78/manuals/r.to.vect.html):

<span class='terminal'>> r.to.vect input=SFD_1 output=SFD_1 type=area --overwrite</span>

#### g.remove

The raster basin is no longer needed and take up a lot of space, remove it with the GRASS command [g.remove](https://grass.osgeo.org/grass78/manuals/g.remove.html).

<span class='terminal'>> g.remove -f type=raster name=SFD_1 --quiet</span>

#### v.clean

There are seldom any errors in the vector representation of the basin created with [r.to.vect](https://grass.osgeo.org/grass78/manuals/r.to.vect.html), but there might be. Hence I clean all the vectors using the [v.clean](https://grass.osgeo.org/grass78/manuals/r.to.vect.html) command. I apply the following cleaning options:

- prune:
- rmdupl: remove duplicate geometry features
- rmbridge: remove bridges connecting area and island or 2 islands
- rmline: remove all lines or boundaries of zero length, threshold is ignored
- rmdangle: remove dangles, threshold ignored if < 0

v.clean input=basin_sfd_000001 output=basin_sfd_000001c type=area tool=prune,rmdupl,rmbridge,rmline,rmdangle thresh=0,0,0,0,-1 --overwrite

#### Export to shape

Finally export the vector to a shape file as in the next step you need to apply another Python script for some more advanced vector editing.

[v.out.ogr](https://grass.osgeo.org/grass78/manuals/v.out.ogr.html).

<span class='terminal'>> v.out.ogr type=area input=SFD_1 format=ESRI_Shapefile output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/SFD_1 --overwrite</span>


```
# SFD basin 1
r.water.outlet input=SFD_drainage output=SFD_1 coordinates=-7928554.691232,1295074.871531 --overwrite

r.null SFD_1 setnull=0

r.to.vect input=SFD_1 output=SFD_1 type=area --overwrite

g.remove -f type=raster name=SFD_1 --quiet

v.clean input=SFD_1 output=SFD_1c type=area tool=break,rmdangle,rmbridge,rmdupl,rmdac,rmline,rmarea, thresh=0,-1,0,0,0,0,200000 --overwrite --quiet

v.out.ogr input=SFD_1cc type=area dsn=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/SFD_1

```

The commands in the shell script are explained below. You can run the shell script  after making it executable:

chmod 777 etc

The complete GRASS processing chain after importing the DEM and creating a new mapset, can be run as a shell script.

```
r.water.outlet input=SFD_drainage output=basin_sfd_000001 coordinates=-7928323.034873,1295306.527889 --overwrite
# in 7.8 NULL is set in r.water.outlet # r.null basin_sfd_000001 setnull=0
r.to.vect input=basin_sfd_000001 output=basin_sfd_000001 type=area --overwrite
g.remove -f type=raster name=basin_sfd_000001 --quiet
v.clean input=basin_sfd_000001 output=basin_sfd_000001c type=area tool=prune,rmdupl,rmbridge,rmline,rmdangle,rmline thresh=0,0,0,0,-1,0 --overwrite
v.db.addcolumn map=basin_sfd_000001c columns="xcoord DOUBLE PRECISION"
v.db.update map=basin_sfd_000001c layer=1 column=xcoord value=-7928323.034873
v.db.addcolumn map=basin_sfd_000001c columns="ycoord DOUBLE PRECISION"
v.db.update map=basin_sfd_000001c layer=1 column=ycoord value=1295306.527889
v.to.db map=basin_sfd_000001c  type=centroid option=area columns=area_km2 units=kilometers

v.out.ogr input=basin_sfd_000001c type=area format=ESRI_Shapefile output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/basin_sfd_000001c.shp --overwrite
```

### ogr2ogr script

Before deciding on how to proceed you must inspect the initial basins of your entire region. As there might be several thousand basins, you need to asessmble them in a single dataset, and then view that singel file. The script _basin_extract_ produced a shell script file that uses the GDAL command line vector processor [ogr2ogr](https://gdal.org/programs/ogr2ogr.html) ("region"_ogr2ogr_basin_"xfd"). The first line takes the first basin vector and copies it to a new data source. All the following command lines appends then basins to the new data source file, thus assembling all basin vector to a single data source file.

```
ogr2ogr  -skipfailures -nlt MULTIPOLYGON /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/basin-SFD-outlet_drainage_amazonia_0_cgiar-250.shp  /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/SFD_000001c.shp

ogr2ogr  -append -skipfailures -nlt MULTIPOLYGON /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/basin-sfd-outlet-area_drainage_amazonia_0_cgiar-250.shp /Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/basin_sfd_000002c.shp

ogr2ogr  -append -skipfailures -nlt MULTIPOLYGON /Volumes/...
```

### Identify complete river mouths

#### Cost growth (only for ocean data)

Principal steps:

1. Get final basin outlet point vector file
2. If your DEM is very large you can apply a buffer to only include shoreline
3. Apply a cost growth analysis from the outlets along the shoreline (this does not work if the water body us not the sea)
4. Cluster the contiguous raster cells
5. Identify the outlet point(s) of each cluster
6. If multiple outlet points are found, you have to decide if you want to join these into a single basin, and which point to keep (default = point with largest drainage area).

#####

##### Cost growth analysis

Cost growth analysis calcuales the accumualted travel cost over a friction surface from predefined starting points. You can use the GRASS cost growth tool [r.cost](https://grass.osgeo.org/grass78/manuals/v.out.ogr.html) for finding the complete width of the mouths of larger rivers. By using the outlet points as starting point and the DEM as a friction surface. All cells with a total accumulated friction cost that equals zero and that are also adjacent to the sea represent the mouth of the basin.

##### Cost grow for entire basin

If you use the entire DEM for the cost growth, the result will connect all mouth cells (at sea level), even if not directly connected along the coast. Mouths across a a river delta will be linked (as long as the DEM shows that the water levels equals 0). The [r.cost](https://grass.osgeo.org/grass78/manuals/v.out.ogr.html) analysis for an entire basin can take a long time.





If you want to export this layer, use [r.out.gdal](https://grass.osgeo.org/grass78/manuals/r.out.gdal.html).

r.out.gdal -f input=lowlevel_SFD_outlet_costgrow format=GTiff type=Byte output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/lowlevel-SFD-outlet-costgrow_amazonia_0_cgiar-250.tif --overwrite



If you want to export this layer, use [r.out.gdal](https://grass.osgeo.org/grass78/manuals/r.out.gdal.html).

r.out.gdal -f input=coastline_SFD_outlet_costgrow format=GTiff type=Byte output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/coastline-SFD-outlet-costgrow_amazonia_0_cgiar-250.tif --overwrite

##### ?????

r.mapcalc "SFD_separate_outlets = if((coastline_SFD_outlet_costgrow <= 0 && SFD_upstream > 1), lowlevel_SFD_outlet_clumps, null())"


#### Clump

From the cost growth analysis you will have identified the complete width of all river mouths in your region. Identified separated from other mouths. To cluster the cells constituting a basin mouth use the GRASS command [r.clump](https://grass.osgeo.org/grass78/manuals/r.clumpl.html). A mouth can either be a single entity, or an assembly of several separate mouth opening, for instance mouths in a river delta. You thus need to run two different clusterings, one for separate mouths (along the coastline) and one for mouths with upstream joints. In both cases you need to set the -d flag to clump diagonal cells.

###### Clump for united (river delta) mouths

r.clump -d input=lowlevel_SFD_outlet_costgrow output=lowlevel_SFD_outlet_clumps --overwrite

r.out.gdal -f input=lowlevel_SFD_outlet_clumps format=GTiff output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/lowlevel-SFD-outlet-clumps_amazonia_0_cgiar-250.tif --overwrite

###### Clump for separate (bifurcated) mouths

r.clump -d input=coastline_SFD_outlet_costgrow output=coastline_SFD_outlet_clumps --overwrite

r.out.gdal -f input=coastline_SFD_outlet_clumps format=GTiff output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/coastline-SFD-outlet-clumps_amazonia_0_cgiar-250.tif --overwrite

##### Identify outflow point

For computational reasons and for avoiding flow double counting, you need to identify a single outflow point for each separate mouth. BALLE





##### Combine to get mouth

All mouth cells can now be extracted by the clump number, only along the coastline where the water level is at sea surface and connected to an outlet candidate while also eliminating all shoreline cells that does not have any upstream area.

###### Add columns

<span class='terminal'>> v.db.addcolumn map=basin_SFD_outlet_pt columns='sepclump INT'</span>

###### load separate mouth clump id

<span class='terminal'>> v.what.rast map=basin_SFD_outlet_pt raster=coastline_SFD_outlet_clumps column=sepclump</span>


######

r.mapcalc "SFD_separate_outlets = if((coastline_SFD_outlet_costgrow <= 0 && SFD_upstream > 1), lowlevel_SFD_outlet_clumps, null())"

r.out.gdal input=SFD_all_outlets format=GTiff output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/SFD-basin-outlets_amazonia_0_cgiar-250.tif --overwrite


r.mapcalc "SFD_all_outlets = if((coastline_SFD_outlet_costgrow <= 0 && SFD_upstream > 1), lowlevel_SFD_outlet_clumps, null())"

r.out.gdal input=SFD_all_outlets format=GTiff output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/SFD-basin-outlets_amazonia_0_cgiar-250.tif --overwrite

##### Create a virtual wall

<span class='terminal'>> r.buffer input=coastline_SFD_outlet_costgrow  output=shorebuffer distances=330 units=meters --overwrite</span>

r.mapcalc "shorewall = if((drain_terminal == 1 && shorebuffer > 1), 1, null())" --overwrite

r.out.gdal input=shorewall format=GTiff output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/shorewall_amazonia_0_cgiar-250.tif --overwrite

Convert the wall to vector with [r.to.vect](https://grass.osgeo.org/grass78/manuals/r.to.vect.html).

<span class='terminal'>> r.to.vect input=shorewall output=shorewall_line type=line</span>

Export

<span class='terminal'>> v.out.ogr type=line input=shorewall_line format=ESRI_Shapefile output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/shorewall_amazonia_0_cgiar-250.shp --overwrite</span>

# Get the center point

The GRASS command for extracting line centerpoints is [v.centerpoint](https://grass.osgeo.org/grass78/manuals/addons/v.centerpoint.html). This command, however, is not installed by defaulet but is an [addon](https://grass.osgeo.org/grass78/manuals/addons/). Thus you must first install the module with [g.extension](https://grass.osgeo.org/grass78/manuals/g.extension.html), before you can run it.

On Mac OSX the compilation of addons require that you have xcode.app installed (it is not enough to just have the command line tool).

g.extension -l


NOTE THAT THE ADDONS CNTAIN LOTS OF HYDROLOGICAL STUFF


Set the -s flag to install system wide:

g.extension -s extension=v.centerpoint operation=add

v.centerpoint input=shorewall_line output=shorewall_center_pt type=line lcenter=mid

v.center input=shorewall_line output=shorewall_center_pt type=line lcenter=mid

###### MAKE A HOLE IN THE WALL


####

r.mapcalc "shorewallDEM = if((drain_terminal == 1 && shorewall > 1), 1, DEM@PERMANENT)" --overwrite

<span class='terminal'>> r.mapcalc "shorewallDEM = if(isnull('DEM@PERMANENT'), shorewall, DEM@PERMANENT)" --overwrite</span>

r.out.gdal input=shorewallDEM format=GTiff output=/Volumes/GRASS2020/GRASSproject/SRTM/region/basin/amazonia/0/shorewallDEM_amazonia_0_cgiar-250.tif --overwrite
