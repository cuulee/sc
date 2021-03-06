
<!-- README.md is generated from README.Rmd. Please edit that file -->
[![Travis-CI Build Status](https://travis-ci.org/mdsumner/sc.svg?branch=master)](https://travis-ci.org/mdsumner/sc) [![AppVeyor Build Status](https://ci.appveyor.com/api/projects/status/github/mdsumner/sc?branch=master&svg=true)](https://ci.appveyor.com/project/mdsumner/sc) [![Coverage Status](https://img.shields.io/codecov/c/github/mdsumner/sc/master.svg)](https://codecov.io/github/mdsumner/sc?branch=master)

sc
==

The goal of sc is to provide a general common form for complex multi-dimensional data.

The need for this for spatial data is illustrated here: <http://rpubs.com/cyclemumner/sc-rationale>

We aim to create a classification of spatial and other hierarchical data in R with tools for more general representations of spatial primitives and the intermediate forms required for translation and analytical tasks. The key is to provide a relational model of indexed primitives and component elements, as a bridge to the traditionally *structural*, or *array/matrix* indexing and storage used in computer graphics and gaming.

A **path** can be treated as a first-class type and and stored as such within a relational model, along with the other entities **objects** ("features") and **vertices**. with this approach we gain two advantages, we can *normalize* the relations (detect and remove redundancy) and also store any additional data about the entitities in the model.

(There is an interplay between "able to store extra information" and "able to normalize", since extra data may introduce further redundancy, but we defer this issue for now since full normalization is not our primary task).

This package provides two main schemes, the `PATH` and the `PRIMITIVE` models. The two models may be mutually exclusive, or they can co-exist. The PATH model can always be derived from the PRIMITIVE model and vice versa but the PRIMITIVE model has extra capacities that PATH cannot provide. (The current design is a distillation and improvement on previous implementations, see `mdsumner/gris` and `r-gris/rangl` for these earlier attempts).

Developer notes.
================

There are key worker functions `sc_coord`, `sc_node`, `sc_object`, `sc_path`.

-   sc\_coord returns the table of coordinates completely flattened, no normalization
-   sc\_object returns the highest level feature metadata
-   sc\_path returns the table of individual paths, with a coordinate count

This generic set of workers is chosen because we often want the complete set of vertices in their pure form. Returning them with no grouping or identifiers and without any de-duplication means we have a representation of the pure geometry. Since the table has no other columns, generic code can be sure that all columns contain a coordinate. That means we don't need specialist code for 'XYZ', 'XYZM', 'XYT', 'TYX' and so on.

The table of individual paths records which object it belongs to, how many coordinates there are and an ID for the path. This is not intended to be 'relational', it's an intermediate form link by pure indexing. Inserting more levels between the paths and the highest objects is possible, but unclear exactly how to do this yet.

The `unjoin` concept is key for mapping the key between unique vertices and a path's instance of those as coordinates, in the right order. We can use the unjoin engine to add structure to other more generic data streams, like GPS, animal tracking, and general sensors.

The model functions `PRIMITIVE` and `PATH` should work in the following cases.

-   to flip from one to another `PRIMITIVE(PATH(PRIMITIVE(x)))` should work for any kind of 'x' model
-   for any sf object: <https://github.com/mdsumner/scsf>
-   convert to igraph objects WIP: <https://github.com/mdsumner/scgraph>
-   spatstat objects WIP: <https://github.com/mdsumner/scspatstat>
-   ...

The classes for all variants of simple features are not worked out, for instance a MULTIPOINT can end up with a degenerate (and expensive) segment table.

More functions `sc_uid` provides unique IDs, and `sc_node` is a worker for a arc-node intermediate model.

Intermediate models

-   Arc-node (WIP)
-   Monotone polygons (future work)

Why?
====

Geographic Information System (GIS) tools provide data structures optimized for a relatively narrow class of workflows that leverage a combination of *spatial*, graphics, drawing-design, imagery, geodetic and database techniques. When modern GIS was born in the 1990s it adopted a set of compromises that divorced it from its roots in graph theory (arc-node topology) to provide the best performance for what were the most complicated sets of cartographic and land-management system data at the time.

The huge success of ArcView and the shapefile brought this arcane domain into common usage and helped establish our modern view of what "geo-spatial data" is. The creation of the "simple features standard"" in the early 2000s formalized this modern view and provided a basis to avoid some of the inconsistencies and incompleteness that are present in the shapefile specification.

Spatial, graphics, drawing-design, imagery, geodetic and database techniques are broader than any GIS, are used in combination in many fields, but no other field combines them in the way that GIS tools do. GIS does however impose a certain view point, a lens through which each of those very general fields is seen via the perspective of the optimizations, the careful constraints and compromises that were formalized in the early days.

This lens is seen side-on when 1) bringing graphics data (images, drawings) into a GIS where a localization metadata context is assumed 2) attempting to visualize geo-spatial raster data with graphics tools 3) creating lines to represent the path of sensor platforms that record many variables like temperature, salinity, radiative flux as well as location in time and space.

The word "spatial" has a rather general meaning, and while GIS idioms sometimes extend into the Z dimension time is usually treated in a special way. Where GIS really starts to show its limits is in the boundary between discrete and continuous measures and entities. We prefer to default to the most general meaning of spatial, work with tools that allow flexibility despite the (rather arbitrary) choice of topological and geometric structures and dimensions that a given model needs. When the particular optimizations and clever constraints of the simple features and GIS world are required and/or valuable then we use those, but prefer not to see that 1) this model must fit into this GIS view 2) GIS has no place in this model. For us the boundaries are not so sharp and there's valuable cross-over in many fields.

The particular GIS-like limitations that we seek are as follows.

-   flexibility in the number and type/s of attribute stored as "coordinates", x, y, lon, lat, z, time, temperature, etc.
-   ability to store attributes on parts i.e. the state is the object, the county is the part
-   shared vertices
-   the ability to leverage topology engines like D3 to dynamically segmentize a piecewise graph using geodetic curvature
-   the ability to extend the hierarchical view in GIS to 3D, 4D spatial, graphical, network and general modelling domains
-   clarity on the distinction between topology and geometry
-   clarity on the distinction between vector and raster data, without having an arbitrary boundary between them
-   multiple models of raster `georeferencing` from basic offset/scale, general affine transform, full curvilinear and partial curvilinear with affine and rectilinear optimizations where applicable
-   ability to store points, lines and areas together, with shared topology as appropriate
-   a flexible and powerful basis for conversion between formats both in the GIS idioms and outside them
-   flexibility, ease-of-use, composability, modularity, tidy-ness
-   integration with specialist computational engines, database systems, geometric algorithms, drawing tools and other systems
-   interactivity, integration with D3, shiny, ggplot2, ggvis, leaflet
-   scaleability, the ability to leverage back-end databases, specialist parallelism engines, remote compute and otherwise distributed compute systems

Flexibility in attributes generally is the key to breaking out of traditional GIS constraints that don't allow clear continuous / discrete distinctions, or time-varying objects/events, 3D/4D geometry, or clarity on topology versus geometry. When everything is tables this becomes natural, and we can build structures like link-relations between tables that transfer data only when required.

The ability many GIS tools from R in a consistent way is long-term goal, and this will be best done via dplyr "back-ending" or a model very like it.

Approach
========

We can't possibly provide all the aspirations detailed above, but we hope to

-   demonstrate the clear need, interest and opportunities that currently exist for fostering their development
-   illustrate links between existing systems that from a slightly different perspective become achievable goals rather than insurmountable challenges
-   provide a platform for generalizing some of the currently fragmented translations that occur across the R community between commonly used tools that aren't formally speaking to each other.
-   provide tools that we build along the way

This package is intended to provide support to the `common form` approach described here. The package is not fully functional yet, but see these projects that are informed by this approach.

-   **rbgm** - [Atlantis Box Geometry Model](https://github.com/AustralianAntarcticDivision/rbgm), a "doubly-connected edge-list" form of linked faces and boxes in a spatially-explicit 3D ecosystem model
-   **rangl** - [Primitives for Spatial data](https://github.com/r-gris/rangl), a generalization of GIS forms with simple 3D plotting
-   **spbabel** - [Translators for R Spatial](https://github.com/mdsumner/spbabel), tools to convert from and to spatial forms, provides the general decomposition framework for paths, used by `rangl`
-   **sfct** - [Constrained Triangulation for Simple Features](https://github.com/r-gris/sfct) tools to decompose `simple features` into (non-mesh-indexed) primitives.

Design
------

There is a hierarchy of sorts with layer, object, path, primitives, coordinates, and vertices.

The current design uses capitalized function names `PATH`, `PRIMITIVE` ... that act on layers, while prefixed lower-case function names produce or derive the named entity at a given level for a given input. E.g. `sc_path` will decompose all the geometries in an `sf` layer to the PATH model and return them in generic form. `PATH` will decompose the layer as a whole, including the component geometries.

`PATH()` is the main model used to decompose inputs, as it is the a more general form of the GIS idioms (simple features and georeferenced raster data) This treats connected *paths* as fully-fledged entities like vertices and objects are, creating a relational model that stores all *vertices* in one table, all *paths* in another, and and all highest-level *objects* in another. The PATH model also takes the extra step of *normalizing* vertices, finding duplicates in a given geometric space and creating an intermediate link table to record all *instances of the vertices*. The PATH model does not currently normalize paths, but this is something that could be done, and is close to what arc-node topology is.

The `PRIMITIVE` function decomposes a layer into actual primitives, rather than "paths", these are point, line segment, triangle, tetrahedron, and so on.

Currently `PATH()` and `PRIMITIVE` are the highest level functions to decompose simple features objects.

There are decomposition functions for lower-level `sf` objects organized as `sc_path`, `sc_coord`, and `sc_object`. `sc_path` does all the work, building a simple map of all the parts and the vertex count. This is used to classify the vertex table when it is extracted, which makes the unique-id management for path-vertex normalization much simpler than it was in `gris` or `rangl`.

**NOTE:** earlier versions of this used the concept of "branch" rather than path, so there is some ongoing migration of the use of these words. *Branch* is a more general concept than implemented in geo-spatial systems generally, and so *path* is more accurate We reserve branch for possible future models that are general. A "point PATH" has meaning in the sense of being a single-vertex "path", and so a multipoint is a collection of these degenerate forms. "Path" as a concept is clearly rooted in optimization suited to planar forms, and so is more accurate than "branch".

In our terminology a branch or path is the group between the raw geometry and the objects, and so applies to a connected polygon ring, closed or open linestring, a single coordinate with a multipoint (a path with one vertex). In this scheme a polygon ring and a closed linestring are exactly the same (since they actually are exactly the same) and there are no plane-filling branches, or indeed volume-filling branches. This is a clear limitation of the branch model and it matches that used by GIS.

Exceptions
----------

TopoJSON, Eonfusion, PostGIS, QGIS geometry generators, Fledermaus, ...

Examples - sf round trip
------------------------

See scsf

Example - sf to SQLite and filtered-read
----------------------------------------

See scdb

Primitives, the planar straight line graph and TopoJSON
-------------------------------------------------------

### Arc-node topoplogy

WIP

``` r

example(arc_node)
```
