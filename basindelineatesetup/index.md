---
layout: page
title: Basin delineate setup
excerpt: "An archive of blog posts sorted by date."
image: std-trmm-3b43v7-precip_3B43_trmm_2001-2016_A
search_omit: true
---

If you want to follow this tutorial series in all its details, you need to [Install GDAL, QGIS and GRASS](https://karttur.github.io/setup-ide/setup-ide/install-gis/#grass). You must also [install <span class='app'>Anaconda</span>](https://karttur.github.io/setup-ide/setup-ide/install-anaconda/) for creating a Python environment, and the [<span class='app'>Eclipse</span> IDE](https://karttur.github.io/setup-ide/setup-ide/install-eclipse/) (Integrated Development Environment) for actually handling the Python code.

To install and compile the required GRASS addons on Mac OSX, you must also have installed and linekd to the Xcode c-compiler _clang_ under the Mac OSX System Development Kit (SDK). How to do that is also covered in the post [Install GDAL, QGIS and GRASS](https://karttur.github.io/setup-ide/setup-ide/install-gis/#grass).

The posts below cover the setup of GRASS, Anaconda and Eclipse. You need to finish them if you want to delineate your own river basins.

<ul class="post-list">
{% for post in site.categories.basin_delineate_setup reversed %}
  <li><article><a href="{{ site.url }}{{ post.url }}">{{ post.title }} <span class="entry-date"><time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%B %d, %Y" }}</time></span>{% if post.excerpt %} <span class="excerpt">{{ post.excerpt | remove: '\[ ... \]' | remove: '\( ... \)' | markdownify | strip_html | strip_newlines | escape_once }}</span>{% endif %}</a></article></li>
{% endfor %}
</ul>
