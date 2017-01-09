---
title: "General forms for hierachical data"
author: "Michael Sumner"
date: "January 2017"
output: html_document
---


Applicant: [Michael Sumner](https://github.com/mdsumner/), [Australian Antarctic Division](http://www.antarctica.gov.au/); [mdsumner@gmail.com](mailto:mdsumner@gmail.com)

Supporting Authors: 

Note, this is a reduced version of this early description: https://github.com/r-gris/table-r-book/blob/master/01-2-overview.Rmd

# the problem

There is no common generic form for spatial data that covers the complexity of geometric and topological forms widely used in R. Worse, there is no way to augment  spatial data with user-driven visualization and interactivity, these expressions tend to be one-way or are converted to  forms that are richer in graphical and interactivity properties at the expense of losing the specialist rigour that the raw data was delivered with. 

The tidyverse is revolutionizing data manipulation, analysis and visualization by systematizing on core database principles and providing common-tools that are fast, reliable and flexible. The grammar of graphics works from a single, possibly nested, tidy data frame and applies `stats` within arranged groupings of data mapped to `aesthetics` and `scales` that are used to build `geom` layers as visualizations. 

Complex spatial data consists of hierarchically arranged and grouped coordinates linked to (usually) `object` level metadata. Visualizations are created by providing user-specified mappings to object level metadata, these are geom layers without any explicit aesthetic or geometric mapping or scaling mappings. It is rare that spatial data includes an in-built mapping of aesthetics, but the standard forms are not capable of storing this information internally. A `ggplot2` object is complex spatial data with all of this rich user-choice-driven metadata applied at the right level, object, group, or primitive. A `ggplot2` object could be re-expressed in standard spatial form by dropping everything but a particular geometric scaling, and object (feature) level aesthetics. 

The richness in R's specialist forms currently lacks a central language for conversion to generic storage and transmission. Most formats are either purely geometry and topology and fields with no aesthetics, or pure aesthetics baked-in to graphical primitives without the original data used to create the mappings. 

This project aims to provide a system to re-express that graphical object without loss to a common form, and that common form is used to generate other specialist forms. That common form is informed directly by database principles, storing data as relations in multiple tables organized by the entities required. It is trivial to support this form in a scaleable way with standard database systems and techniques. The goal is not specifically to be able re-express ggplot2 objects or simple features, but to provide a language and common tools for creating those converter pipelines. 

The common form is scaleable in terms of memory and computation, but also in terms of geometric and topological dimensionality. Our existing prototypes illustrate that generality by supporting round-trip workflows of specialist multidimensional data forms. 


# The plan

Request for advice on key parties to contribute, funding for working groups and presentations. 

Investigate best options for front-end user interfaces and back-end systems. 

* lists of tables, as illustrated in spbabel, rangl, rbgm
* sf-like forms, list-columns with shared-entity semantics
* tricky environments with vertex, primitives pools
* database or database-like connections in list-columns - nested tibbles that are back-ended?

Key outputs

1. Provide tools for decomposing geo-spatial and other complex data to common general forms. 
3. Document and promote use of the general principles and approach rather than localized implementations. Help the R community to generalize and reduce fragmentation. 
4. Implement a prototype general-form geo-spatial-graphics data structure that can store geometry, topology, aesthetic mappings. 
5. Document workflows for file formats and interchange requirements for the general form based on standard database techniques. 


## Existing implementations

The multiple-table-in-list approach described here is used in the following packages.


* **rbgm** - [Atlantis Box Geometry Model](https://github.com/AustralianAntarcticDivision/rbgm), a "doubly-connected edge-list" form of linked faces and boxes in a spatially-explicit 3D ecosystem model
* **rangl** - [Primitives for Spatial data](https://github.com/r-gris/rangl), a generalization of GIS forms with simple 3D plotting
* **spbabel** - [Translators for R Spatial](https://github.com/mdsumner/spbabel), tools to convert from and to spatial forms, provides the general decomposition framework for branches, used by `rangl`

# Hierarchical spatial data 

GIS-based vector data provides a complex set of data structures, and the [modern R form](https://cran.r-project.org/package=sf) is compliant with the [simple features standard](https://en.wikipedia.org/wiki/Simple_Features). In R these are stored in *nested lists* where the physical structure matches the logical structures in the data. 

R can deal with a much wider variety of complex data types than the simple features standard, with key examples in the `ggplot2` family, `igraph` networks, `rgl` indexed mesh models, `rhdf5` and `ncdf4` arrays, and many "tracking" models such as `adehabitatLT`, `trajectories` and `move`. There is a need for a common language for translating and storing data between these specialized forms, and while no single file format or class is sufficiently flexible there are well-established database techniques for dealing with arbitrarily complex models. 

A `Spatial` object (`sp`) is a complex nested list of coordinates stored in matrices with S4 classes, with each top-level item linked by ID to rows in a data frame. A newer form to replace `Spatial` that handles more kinds of data sets is simple features in `sf`. Here the coordinates are in nested lists of matrices or vectors.  The `sf` package stores each object with its data frame row rather than by a remote link ID, and the package is compliant with the simple features standard which removes some ambiguities that existed in `sp`. The simple features approach is more aligned with the tidyverse principles, but does this by way of a formal API to switch from lists to data frames in pre-specified ways. 

Nested lists of data can be inverted and stored in an *inside-out* way as normal data frames. Various packages provide conversions between nested and table forms,  but there is no overall approach that works in the general case and no categorization of the common conversions that are most useful. Worse, there are fragmented approaches spread across dozens of implementations without one central framework or vision.  There is room for extension and improvement to the handling of data structures with conversion tools, and ideally a central form-converter framework. Here we focus on GIS vector data to show the limitations of the nested structures, and how being able to flip between representations provides much added power and provides a clear pathway forward for many complex problems that don't fit in the traditional GIS model.


There are four main categories in the classification which will drive the set of translator tools. 

* Bespoke hierarchical (nested lists of things)
* Tidy hierarchical (nested data frames, single or double)
* Fortify (two tables, geometry and object metadata)
* Branch (three tables, coordinates, branches, objects)
* Primitives (usually four tables, object, primitives, links, vertices)

Each of these forms has direct applications for a variety of tasks, either for transferring between forms or for applications that are more efficient in a given form. 

Key advantages of each form

* in the branch model parts are identifiable and track-able - i.e. size of rings, length of line strings - in simple features we need to explode an object, badge every part with an ID and track those
* higher dimensional topological forms are provided naturally, the 2D primitives approach fits naturally as an extension to 1D primitives model
* entity tables provide unlimited room for extra information in the right places, i.e. length, area, duration, name can be stored on branches or primitives as needed and used for aesthetics in visualizations. For a triangulated surface with a Z geometry, this belongs on the unique vertex table and not on the link-instances-of-coordinates. For GPS data, we can densify the X-Y planar coordinates (i.e intersecting tracks at a depot) while keeping individual track time, measurement information on the link-instance table. 
* The branch and primitives models may be combined, so that the branch table records the way the original simple features are constructed by a branch-link-primitives table - so we can have a perfect record of the original data, recreatable if needed - or completely reworked by operations on primitives that when recombined provide a modified object.


# The problem

There is a strong relationship between the hierarchical forms of data structure used in geo-spatial analysis and in the grammars of data analysis and graphics but the translation between geo-spatial forms and the grammars is disjointed and sometimes awkward, relying on localized implementations.  It is possible to classify and structure geo-spatial data in ways complementary to those used in the grammars, allowing for more general and extensible model development. 

Many projects are seeking to extend the types in the `sf` package but these types are already closed-form, they do not have a general structure. The `tidyverse` seeks to provide an *always data frame* approach, and there is now strong support for data frames that maintain their integrity as tabels and remove some problems, and include the ability to store hierarchical structures kept inside list-columns. These list-columns can stored nested tables, bespoke recursive structures (sf, stat models), binary blobs, and even functions and references. 

Having a single data frame is very powerful and simple, and many operations can be done naturally without customization. Unnesting a nested table automatically matches a parent ID to its children, copying the selected attributes as needed. Dplyr verbs can evaluate user-level functions on list-column elements, keeping track of the vectorized result in list-form within a tidy row-based data structure. However, many optimizations and specializations in geo-spatial data rely on indexing, with a key linking a unique element in another structure. These are not suited to storage within a single data frame, but worse they are not even implemented in simple features. The standard provides no way to index a unique coordinate within a shared network. 

To transform the coordinates in simple features, we must extract all of the fragmented matrixes of values and either transform each one or collect them together for transformation, then replace them where they live in the nested list. What if we want to identify the features that share an edge, or share a vertex? We can pull them all out and do tests on the set, and keep a record of ... what?  There is no way to record what part in which feature a given vertex is from. This is important, because of topological surfaces that share boundaries and edges and vertices in a big mesh. The unique vertices is the right place to store the geometry, and we can add a Z, Time, Temperature, Salinity and so on to those. If we store all the vertices as entities in a table, we automatically have a sensible place to add those measurements as columns. We can transform them together as one set already. 

# Supporting documentation

https://github.com/r-gris/table-r-book/blob/master/01-2-overview.Rmd

https://rpubs.com/cyclemumner/spatial-normal-forms