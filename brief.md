Brief (for authors only)
========================================================

This is the master .Rmd document that will eventually be used to tie
all the sections together for the final book chapter.
"to do" lists, ideas and conventions can be written here, 
but it's main purpose is to provide an up-to-date structure on the final chapter.

The sections are preliminary; it is recommended that the sections are 
written as separate .Rmd files before merging them all into one master document
to reduce complexity and time taken to compile. 

6000 words – including figures, references and tables

We therefore need to include some introductory material. The key to fitting all this in will be to keep it applied – we can assume some prior knowledge of spatial data.

### Typographic conventions (for authors)

Use !!! to flag an issue that requires future attention - e.g. "see section !!!"

Endnotes - add this section to the end of each section.

# Introduction

 What is R? The rise of R's spatial capabilities.  Why R for spatial data visualisation?
 R for Reproducible research. An introductory session. Chapter overview.


# Components of a Good Map

Krygier and Wood 2005's checklist. 
What makes for a good map and why R is a valuable tool for making them.

# R and Spatial Data

Loading and saving data.
The structure of spatial data in R.
Spatial* data. Simplifying spatial data with `fortify`. 
The main spatial packages (sp, rgdal rgeos).


Maps with ggplot2. Adding base maps with ggmap. 
Manipulating spatial data. 
Coordinate reference systems and transformations. 
Attribute joins. Spatial joins.
Aggregation.
Clipping.



# A detailed worked example

This needs to somehow incorporate: 

Choropleths, points and segments

Layering

Basemaps with ggmap

The inclusion of images and other adornments such as scale and north arrow

Colour selection- both discrete and continuous palettes.

Line widths and transparency

Faceting

# Conclusion

# Taking spatial analysis in R further

# Misc/unallocated topics/discussion

Key spatial packages and the roles they play- rgdal, sp, maptools(?)
Key visualisation packages: ggplot2 and ggmap. Base plots may also be worth a mention, I don’t want to go near lattice or anything like that.

How spatial data are stored and how their elements (attribute table, coordinates, bbox etc) can be accessed. 

Common types of plot –choropleth, points, lines, basemaps.

Concept of layering.

Key parameters to consider- colour, transparency, line width, background.

Adornments: title, north arrow? Scale?

Exporting graphics for publication.

# References

Bivand, R., & Gebhardt, A. (2000). Implementing functions for spatial statistical analysis using the language. Journal of Geographical Systems, 2(3), 307–317.

Bivand, R. S., Pebesma, E. J., & Rubio, V. G. (2008). Applied spatial data: analysis with R. Springer.

Burrough, P. A., & McDonnell, R. A. (1998). Principals of Geographic Information Systems (revised edition). Clarendon Press, Oxford.

Harris, R. (2012). A Short Introduction to R. 
[social-statistics.org](http://www.social-statistics.org/).

Kabacoff, R. (2011). R in Action. Manning Publications Co.

Krygier, J. Wood, D. 2011. Making Maps: A Visual Guide to Map Design for 
GIS (2nd Ed.). New York: The Guildford Press.

Longley, P., Goodchild, M. F., Maguire, D. J., & Rhind, D. W. (2005). 
Geographic information systems and science. John Wiley & Sons.

Ramsey, P., & Dubovsky, D. (2013). Geospatial Software's Open Future. 
GeoInformatics, 16(4). 

Sherman, G. (2008). Desktop GIS: Mapping the Planet with Open Source Tools. Pragmatic Bookshelf.

Torfs and Brauer (2012). A (very) short Introduction to R. The Comprehensive R Archive Network.

Venables, W. N., Smith, D. M., & Team, R. D. C. (2013). An introduction to R. The Comprehensive R Archive Network (CRAN). Retrieved from http://cran.ma.imperial.ac.uk/doc/manuals/r-devel/R-intro.pdf .

Wickham, H. (2009). ggplot2: elegant graphics for data analysis. Springer.


