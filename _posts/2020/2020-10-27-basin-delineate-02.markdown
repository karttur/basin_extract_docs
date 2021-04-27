---
layout: post
title: "Basin delineation: 2 Python distilling"
categories: basindelineate
excerpt: "Python script for analysing and distilling basin candidates, and producing a set of GRASS scripts"
tags:
  - Python
  - GRASS script
  - basin
  - delineation
image: avg-trmm-3b43v7-precip_3B43_trmm_2001-2016_A
date: '2020-10-27 T06:17:25.000Z'
modified: '2020-10-27 T06:17:25.000Z'
comments: true
share: true

---
<script src="https://karttur.github.io/common/assets/js/karttur/togglediv.js"></script>

# Introduction

Stage 2 of the tutorial series on Basin delineation is the central module. At its core it only requires a Digital Elevation Model (DEM) and produces a vector datasets with polygons representing basins. The processing, however, benefits greatly from preprocessing of the DEM as described in the three preceding posts on patching up the DEM. The first stage DEM adjustments removes some coastal situations that causes errors in the basin delineation in this stage.

Further, the basins produced in this stage (stage 2) are only preliminary. They need to be cleaned up in order to correctly represent entire basins and only entire basins. And you need to do a manual inspections for some particular geographical situations geographical that can not be resolved automatically.

This stage requires the custom written Python package, _basin_extract_. From this stage the package generates three shell scripts that each represent a step and muct be run in sequence.

Bot the running of the scripts and the manual inspection and correction are covered in [stage 3](../basin-delineate-03).

## Prerequisites

You must have setup GRASS 7 and imported a DEM as described in [Installation & Setup](../../basindelineatesetup) of this blog. And then you must have completed the GRASS watershed processing as outlined in [part 1](../basin-delineate-01c).

### Python processing

If you followed the Installation & Setup and part 1 of this series, you should have candidate basin outlets available as vector point files in shape format. You can use <span class='app'>QGIS</span> (or another GIS software) to inspect the data source.

The aim of this stage of the basin delineation is threefold:

1. to identify outlet points to include for generating preliminary basins,
2. to retain only a single outlet point per separate mouth - including one point per mouth for rivers (basins) that have multiple mouths (like e.g. a river delta),
3. to generate the GRASS command line code for basin delineation.

For these tasks I have created a Python script called [basin_extractor](#) available through GitHub.

#### Default distillation method

Having experimented with different alternatives for creating a set of preliminary basins, the most resilient method is to create a virtual DEM directing flow out of the basin mouths towards a single outlet point (raster cell). Upstream basin delineation is rather problem free, given that the DEM is correct. As pointed out in [part 1](../basin-delineate-01a) it is the flow in wide channels and at outlets that cause problems. In particular if the basins are to be used for estimating water balances of entire basins (i.e. at the basin outflows).

The default method thus entails the following:

- Candidate outlets defined from MFD and SFD combined,
- virtual coastline walls enclosing the basin mouths,
- a single cell punched hole in each wall for directing basin outflow,
- changing outlet point from the "real" position to the punched hole,
- virtually tilting basin mouths towards the punched hole.

This ensures that all basins above the user defined threshold size are identified. And that flow out of each river mouth of all identified basins is through a single cell (per mouth if the outlet is a delta with multiple mouths) and can be exactly accounted for.

There are, however, other alternatives relying on using either the SFD or the MFD generated outlets. For some cases it might be necessary to separate or complementing basins, and then you might need to use the alternatives outlined below.

#### Alternative basin distillations

There are 5 basic alternatives for how to distil candidate basin outlet points:

1. Use the combined SFD-MFD ('MOUTH') default system with a virtual coastline wall,
2. Use all SFD identified outlets for generating preliminary basins,
3. Cluster SFD outlets based on adjacent MFD candidate outlets and distill to a single SFD outlet per cluster,
4. Use all MFD identified outlets for generating preliminary basins,
5. Cluster adjacent MFD outlets based on the MFD candidate outlets and distill to a single outlet per cluster.

You might have to combine two or more of the above methods in order to get a proper delineation of all basins in your study region. If that is required can only be decided after a visual inspection. My suggestion os that you start with 1. the combined SFD-MFD method.

#### Parameterisation

The Python script _basin_extractor_ is parameterised from an XML file that follows the general structure of [Karttur´s GeoImagine Framework](https://karttur.github.io/geoimagine/concept/concept-concepts/#xml-coded-parameterization):

```
<basindelineate>
	<userproj userid = 'karttur' projectid = 'karttur' tractid= 'amazonia' siteid = '*' plotid = '*' system = 'MODIS'></userproj>

	<!-- Define processing -->
	<process processid ='BasinDelineate'>
		<overwrite>Y</overwrite> # Y(es) or N(o)
		<delete>N</delete> # Y(es) or N(o)
		<parameters
		stage = '1' # 1, 2 or 3
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

The core parameters include:

- stage [1, 2 or 3] (this part is for stage = '1')
- adjacentdist [the maximum distance in map units used for clustering candidate outlet points] (only relevant if parameter _distill='MFD'_)
- outlet ['SFD', 'MFD' or 'MOUTH']
- distill ['MFD' or 'None']
- clusteroutlet ['central' or 'maximum']
- basinthreshold [r.watershed parameter _threshold_, default= 2000]
- watershed [flow direction setting for iterated run of r.watershed, only relevant if parameter _distill='MOUTH'_]
- verbose [0, 1 or 2]
- proj4CRS ['proj4CRS definition for output data if input layers lack proper projection information']

##### Combined SFD-MFD processing (recommended)

To do a combined SFD-MFD processing, use the following settings:

- outlet='MOUTH'
- distill is ignored
- adjacentdist is ignored
- watershed _must be set_

The combined processing is the most complex, but also generates the most consistent result. The processing includes the following steps:

- identify one candidate outlet point per river mouth (either central or with maximum upstream as determined by the _clusteroutlet_ parameter)
- move candidate outlet point to the nearest cell in the shorewall
- Create GRASS shell script for repeating the r.watershed analysis with the new outlet point and a hydrologically corrected DEM draining towards this point.

The ensuing processing (in step 2 of this stage) thus includes creating a new, hydrologically adjusted virtual DEM and then redoing the [r.watershed]() analysis.

If you want to run the combined alternative for other data source (ie. for and SFD analysis), you do that by defining the source for the standard combined input (""). ???

##### Non-clustered SFD processing

- outlet='SFD'
- distill='None'
- adjacentdist is ignored
- watershed is ignored

The following _srccomp_ tags must be set:

- basin-SFD-outlet-pt

This is the simplest delineation method. It just takes all SFD outlets and delineates basins from these.

##### Clustered SFD processing

- outlet='SFD'
- distill='MFD'
- adjacentdist _must be set_
- watershed is ignored

The following _srccomp_ tags must be set:

- basin-SFD-outlet-pt
- basin-MFD-outlet-pt


With this method, all MFD outlets are clustered and any if two or more SFD outlets are found within the same cluster, only one is retained. Which one to retain is determined by the parameter _clusteroutlet_. The parameter _adjacentdist_ determines the maximum distance between points for belonging to the same cluster.

##### Non-clustered MFD processing

- outlet='MFD'
- distill='None'
- adjacentdist is ignored
- watershed is ignored

The following _srccomp_ tags must be set:

- basin-MFD-outlet-pt

With these parameter setting, _basin_extract_ delineates basins from all MFD candidates.

##### Clustered MFD processing

- outlet='MFD'
- distill='MFD'
- adjacentdist _must be set_
- watershed is ignored

The following _srccomp_ tags must be set:

- basin-MFD-outlet-pt

With both _outlet_ and _distill_ parameters set to _MFD_, the MFD outlet candidates are first clustered. Within each cluster only one MFD outlet is retained. Which one to retain is determined by the parameter _clusteroutlet_. The parameter _adjacentdist_ determines the maximum distance between points for belonging to the same cluster.

##### Editing the data source

The processing is determined by the default naming convention of the source data sets. But you can manually edit any of the source data prior to processing. If you only want to process a few of the candidates, simply delete all other candidates. You can for instance use <span class='app'>QGIS</span> to open the point vector file (in ESRI shape format), change to edit mode and delete the candidates you are not interested in. Then run _basin_extract_ as usual.

You can also change the name of, say an SFD generated point vector file and change to the default MOUTH name, and then run the MOUTH analysis instead of the SFD analysis.

### Output

The script _basin_extractor_ will produce a point vector file in shape format containing the distilled outlet candidates points as requested in the parameterization. Alongside the point vector file, the script also produces a shell script (<span class='file'>.sh</span>) with the GRASS commands for delineating the basins from the selected candidate points. The name of the shell script file is defaulted to reflect which of the four main alternatives the parameterisation was set to.

#### Python script output

The main output of stage 1 of the Python script _basin_extract_ is a GRASS command line script. [Part 4](../basin-delineate-04) of this tutorial series will take you through he processing in that script.
