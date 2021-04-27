---
layout: post
title: Eclipse for Basin extraction
categories: basin_delineate_setup
excerpt: "Setup Eclipse for the Python packages Basin_extraction"
tags:
  - Eclipse
  - Basin extract
  - setup
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2020-10-15 11:27'
modified: '2020-10-15 T18:17:25.000Z'
comments: true
share: true
fig1: eclipse_ide_launcher
fig2: eclipse_welcome_page
fig3: eclipse_menu_open-other-perspective
fig4: eclipse_window_select_perspective
fig5: eclipse_select_interpreter_ortho-py38
---

## Introduction

_basin_extract_ is a Python package that belongs to the semi automated system for delineating river basins created by Thomas Gumbricht. This post is a manual on how to setup a Python environment for the _basin_extract_ package in the <span class='app'>Eclipse</span> Integrated Development Environment (IDE). The manual focuses on MacOS, if want to us Linux, the installation of <span class='app'>Eclipse</span> is covered in the post [SPIDE for Ubuntu 20](https://karttur.github.io/setup-ide/blog/ubuntu20-setup-spide/).

### Prerequisites

You must have installed [Anaconda](https://karttur.github.io/setup-ide/setup-ide/install-anaconda/) for handling virtual Python environments and [Eclipse IDE for PyDev](https://karttur.github.io/setup-ide/setup-ide/install-eclipse/).

### Setup virtual Python environment

The _basin_extract_ package requires the standard (default) Python packages that come with Eclipse, plus some additional packages:

- numpy
- GDAL
- shapely
- scipy

Trying to setup a working Python environment for _basin_extract_ in October 2020 I found that the listed packages are incompatible in Python 3.8. The virtual environment herein is thus setup using the older Python 3.6 interpreter.

The basics of [Conda virtual environments](https://karttur.github.io/setup-ide/setup-ide/conda-environ/) is covered in another post. More extensive information is available on the [official conda document site](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html). Here I will only step over the barebones of setting up a virtual environment for Ortho.

When you open a <span class='app'>Terminal</span> window on a machine with Conda installed, the first entry on the cursor line should be the active conda environment. <span class='terminal'>(base)</span> is the default (overall) environment. If the cursor line of your Terminal window does not say <span class='terminal'>(base)</span> you have activated another Conda environment. To return to base enter the command:

<span class='terminal'>$ conda activate</span>

#### Update conda and anaconda

Before setting up the virtual environment, update Conda and Anaconda at the terminal:

<span class='terminal'>(base) ...$ conda update conda</span>

<span class='terminal'>(base) ...$ conda update anaconda</span>


#### Create a Python 3.6 virtual environment through yaml

YAML (a recursive acronym for "YAML Ain't Markup Language") is usually a kind of configuration file. <span class='terminalapp'>conda</span> virtual environments can be created (or _activated_) using a yaml file that defines all the packages to install. The YAML file for the _basin_delineate_ package is available in the [GitHub]() repo under [docs]() as [basin_extract_py36.yml](). You can download the YAML file and then edit the last line that defines the location of your conda virtual environments. Put the YNAL file in your home diretory and open a <span class='app'>Terminal</span> also pointing to your home directory and activate a new environment with the command:

<span class='terminal'>$ conda activate -f basin_extract_py36.yml</span>

#### Create a Python 3.6 virtual environment from scratch

The basic command for creating a virtual Python environment with a pre-defined version of Python is (do not run it yet, read a bit further first):

<span class='terminal'>(base) ...$ conda create -n basin_extract_py36 python=3.6</span>

If your Conda environment is defined to include a set of additional packages (if you have no idea about what I am writing, never mind and just continue), then these will be installed in your virtual environment on top of the default Python installation. To avoid that instead type:

<span class='terminal'>(base) ...$ conda create \-\-no-default-packages -n basin_extract_py36 python=3.6</span>

This will install a barebone Python 3.6 virtual environment even if you have defined additional packages to be installed by default. Now you can go ahead and create the barebone virtual environment with one of the command above.

#### Install additional packages

Your newly created environment does not include the additional packages needed (the ones listed above)for the Python package _basin-extract_. When adding these packages you have to make sure that they are installed within your new environment. That can be achieved by either destining the installation from <span class='terminal'>(base)</span> or first activate your new environment. Here I will only show how to change the active environment and then install. Thus activate your virtual environment:

<span class='terminal'>(base) ...$ conda activate basin_extract_py36</span>

The cursor line will then change to reflect that you are now in another environment.

<span class='terminal'>(basin_extract_py36) ...$</span>

With your new Python 3.6 environment active you can install the required packages:

<span class='terminal'>(basin_extract_py36) ...$ conda install -c conda-forge gdal</span>

<span class='terminal'>(basin_extract_py36) ...$ conda install -c anaconda numpy</span>

<span class='terminal'>(basin_extract_py36) ...$ conda install -c anaconda scipy</span>

<span class='terminal'>(basin_extract_py36) ...$ conda install -c conda-forge shapely</span>

Setting this up in October 2020, all installations went through without problems. However, setting up a virtual environment with more recent versions of Python can lead to conflicts between packages. For that reason it is better to install all required packages with the Python initial installation, as explained in the next section.

#### Install packages with the python version

Setting up a virtual environment with all the packages listed has the advantage that Conda checks the consistencies between all packages before installing. To try that out for the _basin_extract_ environment, return to base:

<span class='terminal'>(basin_extract_py36) ...$ conda activate</span>

And then extend the Conda _create_ command with a list of all the packages to install:

<span class='terminal'>(base) ...$ conda create \-\-no-default-packages -n basin_extract_py36 python=3.6 numpy scipy gdal shapely</span>

### Set your virtual environment as your Python interpreter in Eclipse

Start <span class='app'>Eclipse</span>. In the _Eclipse IDE Launcher_ window, specify a new location and click <span class='button'>Launch</span>.

<figure>
<img src="{{ site.commonurl }}/images/{{ site.data.images[page.fig1].file }}">
<figcaption> {{ site.data.images[page.fig1].caption }} </figcaption>
</figure>

In the _Welcome_ page that opens, just go to Workbench (in the upper right corner).

<figure>
<img src="{{ site.commonurl }}/images/{{ site.data.images[page.fig2].file }}">
<figcaption> {{ site.data.images[page.fig2].caption }} </figcaption>
</figure>

The default IDE that opens is for Java projects, but you want it to be a _PyDev_ project page. To change to a PyDev perspective, go via the menu:

<span class='menu'>Window -> Perspective -> Open Perspective -> Other... </span>

<figure>
<img src="{{ site.commonurl }}/images/{{ site.data.images[page.fig3].file }}">
<figcaption> {{ site.data.images[page.fig3].caption }} </figcaption>
</figure>

In the window that opens, select _PyDev_. If _PyDev_ is not available as an alternative, you need to go back to post on [Eclipse IDE for PyDev](https://karttur.github.io/setup-ide/setup-ide/install-eclipse/) and use the Eclipse Marketplace to install _PyDev_ for Eclipse.

<figure>
<img src="{{ site.commonurl }}/images/{{ site.data.images[page.fig4].file }}">
<figcaption> {{ site.data.images[page.fig4].caption }} </figcaption>
</figure>

To set up your virtual Python environment as the Python interpreter, go via the main menu:

<span class='menu'>Eclipse -> Preferences</span>

And then in the _Preference_ window that opens, go via the left side navigation:

<span class='menu'>PyDev -> Interpreters -> Python Interpreter</span>,

and the right side of the window expands. Click the <span class='button'>Browse for python/pypy exe</span> and navigate to the directory <span class='file'>envs</span> under your <span class='file'>Anaconda</span> installation. There you will find the virtual environment that you just created, <span class='file'>basin_extract_py36</span>. The Python executable file is then under the path <span class='file'>bin/python</span>. The full path on my machine is shown in the figure below. Click <span class='button'>OK</span> to select the python interpreter.

<figure>
<img src="{{ site.commonurl }}/images/{{ site.data.images[page.fig5].file }}">
<figcaption> {{ site.data.images[page.fig5].caption }} </figcaption>
</figure>

In the [next post](../basin-delineate-pydev-setup) you will setup the projects, packages and modules that constitute the _basin_extract_ Python package.
