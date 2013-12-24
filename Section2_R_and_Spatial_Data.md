R and Spatial Data
==================

Spatial Data in R
-----------------

In any data analysis project, spatial or otherwise, it is important to
have a strong understanding of the dataset before progressing. This
section will therefore begin with a description of the input data. 
We will see how data can be loaded into R and exported
to other formats, before going into more detail about the underlying
structure of spatial data in R: how it 'sees' spatial data is quite
unique.

### Loading spatial data in R

In most situations, the starting point of spatial analysis tasks is
loading in pre-existing datasets. These may originate from government
agencies, remote sensing devices or 'volunteered geographical
information' (Goodchild 2007). The diversity of
geographical data formats is large.

R is able to import a very wide range of spatial data formats thanks to
its interface with the Geospatial Data Abstraction Library (GDAL), which
is enabled by the package `rgdal`. Below we will load
data from two spatial data formats: GPS eXchange (`.gpx`) and an ESRI
Shapefile.

`readOGR` is in fact capable of loading dozens more file formats, so the
focus is on the *method* rather than the specific formats. The 'take
home message' is that the `readOGR` function is capable of loading most
common spatial file formats, but behaves differently depending on file
type. Let's start with a `.gpx` file, a tracklog recording a bicycle
ride from Sheffield to Wakefield which was uploaded Open Street Map
[3].



```r
download.file("http://www.openstreetmap.org/trace/1619756/data", destfile = "data/gps-trace.gpx")
library(rgdal)  # load the gdal package
shf2lds <- readOGR(dsn = "data/gps-trace.gpx", layer = "tracks")  # load track
plot(shf2lds)
shf2lds.p <- readOGR(dsn = "data/gps-trace.gpx", layer = "track_points")  # load points
points(shf2lds.p[seq(1, 3000, 100), ])
```

![plot of chunk Leeds to Sheffield GPS data](figure/Leeds_to_Sheffield_GPS_data.png) 


There is a lot going on in the preceding 7 lines of code, including
functions that you are unlikely to have encountered before. Let us think
about what has happened, line-by-line.

First, we used R to *download* a file from the internet, using the
function `download.file`. The two essential arguments of this function
are `url` (we could have typed `url =` before the link) and `destfile`,
the destination file. As with any function, more optional
arguments can be viewed by by typing `?download.file`.

When `rgdal` has successfully loaded, the next task is not to import the
file directly, but to find out which *layers* are available to import, 
with `ogrListLayers`. The output from this command tells us
that various layers are available, including `tracks` and
`track_points`: try it. These are imported into R's *workspace* using `readOGR`. 

Finally, the basic
`plot` function is used to visualize the newly imported objects, ensuring
they make sense. In the second `plot` function, we take a subset of the
object (see section ... for more on this).

As stated in the help documentation (accessed by entering `?readOGR`),
the `dsn =` argument is interpreted differently depending on the type of
file used. In the above example, the file name was the file name.
To load Shapefiles, by contrast, the *folder* containing the data is
used:


```r
lnd <- readOGR(dsn = "data/", "london_sport")
```


Here, the files reside in a folder entitled `data` which in
R's current working directory (remember to check this using `getwd()`).
If the files were stored in the working directory, one would use
`dsn = "."` instead. Again, it may be wise to plot the data that
results, to ensure that it has worked correctly. Now that the data has
been loaded into R's own `sp` format, try interrogating and plotting it,
using functions such as `summary` and `plot`.

### The size of spatial datasets in R

Any data that has been read into R's *workspace*, which constitutes all
objects that can be accessed by name and can be listed using the `ls()`
function, can be saved in R's own data storage file type, `.RData`.
Spatial datasets can get quite large and this can cause problems on
computers by consuming all available memory (RAM) or hard
disk space. It is wise to understand
roughly how large spatial objects are, providing insight
into how long certain functions will take to run.

In the absence of prior knowledge, which of the two objects loaded in
the previous section would one expect to be larger? One could
hypothesize that the London boroughs represented by the object `lnd`
would be larger based on its greater spatial extent, but how much larger? 
The answer in R is found in the function `object.size`:


```r
object.size(shf2lds)
```

```
## 103168 bytes
```

```r
object.size(lnd)
```

```
## 79168 bytes
```


Surprisingly, the GPS data is larger. To see why, we can find out how
many *vertices* (points connected by lines) are contained in each
dataset:


```r
sapply(lnd@polygons, function(x) length(x))
```

```
##  [1] 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
```

```r
x <- sapply(lnd@polygons, function(x) nrow(x@Polygons[[1]]@coords))
sum(x)
```

```
## [1] 1102
```

```r

sapply(shf2lds@lines, function(x) length(x))
```

```
## [1] 1
```

```r
(nverts <- sapply(shf2lds@lines, function(x) nrow(x@Lines[[1]]@coords)))
```

```
## [1] 6085
```


It is quite likely that the above code little sense at first. The
important thing to remember is that for each object we performed two
functions: 1) a check that each line or polygon consists only of a
single *part* (that can be joined to attribute data) and 2) the use of
`nrow` to count the number of vertices. The use of the `@` symbol should
seem strange - its meaning will become clear in the section !!!. (Note
also that the function `fortify`, discussed in section !!!, can also be
used to extract the vertice count of spatial objects in R.)

Without worrying, for now, about how these vertice counts were
performed, it is clear that the GPS data has almost 6 times the number
of vertices as does the London data, explaining its larger size. Yet
when plotted, the GPS data does not seem more detailed, implying that
some of the vertices in the object are not needed for visualisation at
the scale of the objects *bounding box*.

### Simplifying geometries

The wastefulness of the GPS data for visualisation (the full dataset may
be useful for other types of analysis) raises the question following
question: can the object be simplified such that its key features
features remain while substantially reducing its size? The answer is
yes. In the code below, we harness the power of the `rgeos` package and
its `gSimplify` function to simplify spatial R objects:


```r
library(rgeos)
```

```
## rgeos version: 0.2-19, (SVN revision 394)
##  GEOS runtime version: 3.3.8-CAPI-1.7.8 
##  Polygon checking: TRUE
```

```r
shf2lds.simple <- gSimplify(shf2lds, tol = 0.001)
(object.size(shf2lds.simple)/object.size(shf2lds))[1]
```

```
## [1] 0.03047
```

```r
plot(shf2lds.simple)
plot(shf2lds, col = "red", add = T)
```


In the above block of code, `gSimplify` is given the object `shf2lds`
and the `tol` argument, short for "tolerance", is set at 0.001 (much
larger values may be needed, for data that use is *projected* - does not
use latitude and longitude). Comparison between the sizes of the
simplified object and the original shows that the new object is less than
3% of its original size. Try plotting the original and simplified tracks
on your computer: when visualized using the `plot` function, it becomes
clear that the object `shf2lds.simple` retains the overall shape of the
line and is virtually indistinguishable from the original object.

This example is rather contrived because even the larger object
`shf2lds` is only 0.107 Mb, negligible compared with the gigabytes of
RAM available to modern computers. However, it underlines a wider point:
for visualizing *small scale* maps, spatial data *geometries*
can often be simplified to reduce processing time and
use of computer memory.

### Saving and exporting spatial objects

A typical R workflow involves loading the data, processing
and finally exporting the data in a new form. `writeOGR`, the 
logical counterpart of `readOGR` is ideal for this task.
Imagine that we want to view the simplified `gpx` data
in software that can only read Shapefiles. This is performed using
the following command:


```r
shf2lds.simple <- SpatialLinesDataFrame(shf2lds.simple, data = data.frame(row.names = "0", 
    a = 1))
writeOGR(shf2lds.simple, dsn = "data/", layer = "shf2lds", driver = "ESRI Shapefile")
```

```
## Error: layer exists, use a new layer name
```


In the above code, the object was first converted into a spatial dataframe class, before 
being exported as a shapefile entitled shf2lds. Unlike with `readOGR`, the driver must 
be specified, in this case with "ESRI Shapefile" [4]. The simplified GPS data is now available
to other GIS programs for further analysis.

The structure of spatial data in R
----------------------------------

Spatial datasets in R are saved in their own format, defined as 
`Spatial...` classes within the `sp` package. For this reason, 
`sp` is the basic spatial package in R, upon which the others depend. 
Spatial classes range from the simples class `Spatial` to the most complex, 
`SpatialPolygonsDataFrame`: the `Spatial` class contains only two required *slots*[5]:


```r
getSlots("Spatial")
```

```
##        bbox proj4string 
##    "matrix"       "CRS"
```


Further details on these can be found by typing `?bbox` and `?proj4string`. 
All other spatial classes in R build on 
this foundation of a bounding box and a projection system (which 
is set automatically to `NA` if it is not known). However, more complex 
classes contain more slots, some of which are lists which contain additional 
lists. To find out the slots of `shf2lds.simple`, for example, we would first 
ascertain its class and then use the `getSlots` command:


```r
class(shf2lds.simple)  # identify the object's class
```

```
## [1] "SpatialLinesDataFrame"
## attr(,"package")
## [1] "sp"
```

```r
getSlots("SpatialLinesDataFrame")  # find the associated slots
```

```
##         data        lines         bbox  proj4string 
## "data.frame"       "list"     "matrix"        "CRS"
```


The same principles apply to all spatial classes including 
`Spatial* Points`, `Polygons` `Grids` and `Pixels`
as well as associated `*DataFrame` classes. For more information on 
this, see the `sp` documentation: `?Spatial`.




Manipulating spatial data
-------------------------

### Coordinate reference systems

As mentioned in the previous section, all `Spatial` objects in R
are allocated a coordinate reference system (CRS).
The CRS of any spatial object can be found using the command 
`proj4string`. In some cases the CRS is not known: in this case 
the result will simply be `NA`.
To discover the CRS of the `lnd` object for example, 
type the following: 


```r
proj4string(lnd)
```

```
## [1] "+proj=tmerc +lat_0=49 +lon_0=-2 +k=0.9996012717 +x_0=400000 +y_0=-100000 +ellps=airy +units=m +no_defs"
```


The output may seem cryptic but is in fact highly informative: `lnd` has *projected*
coordinates, based on the 
[*Transverse Mercator*](http://en.wikipedia.org/wiki/Transverse_Mercator_projection)
system (hence `"+proj=tmerc"` in the output) and its origin is at latitude 49N, -2E.
This point is 

If we *know* that the CRS is incorrectly specified, it can be re-set.
In this case, for example we know that `lnd` actually has a CRS OSGB1936.
Knowing also that the code for this is 27700, it can be updated as follows:


```r
proj4string(lnd) <- CRS("+init=epsg:27700")
proj4string(lnd)
```

```
## [1] "+init=epsg:27700 +proj=tmerc +lat_0=49 +lon_0=-2 +k=0.9996012717 +x_0=400000 +y_0=-100000 +datum=OSGB36 +units=m +no_defs +ellps=airy +towgs84=446.448,-125.157,542.060,0.1502,0.2470,0.8421,-20.4894"
```


The CRS has now been updated - note that the key details are all the same as before. 
Note: this method **should never** be used as an attempt to *reproject* data from
one CRS to another. 

### Reprojecting data

Transforming the coordinates of spatial data from one CRS to another (reprojection)
is a common task in GIS. This is because data
from national sources are generally provided in *projected* coordinates
(the location on the cartesian coordinates of a map) whereas data from GPSs and the internet 
are generally provided in *geographic* coordinates, with
latitude and longitude measured in degrees to locate points on the surface of the globe.

Reprojecting data in R is quite simple: all you need is a spatial object with a known
CRS and knowledge of the CRS you wish to transform it to. To illustrate why that is necessary, 
try to plot the objects `lnd` and `shf2lnd.simple` on the same map:


```r
combined <- rbind(fortify(shf2lds.simple)[, 1:2], fortify(lnd)[, 1:2])
```

```
## Regions defined for each Polygons
```

```r
plot(combined)
```

![plot of chunk Plot of spatial objects with different CRS](figure/Plot_of_spatial_objects_with_different_CRS.png) 


In the above code we first extracted the coordinates of each point 
using `fortify` and then plotted them using `plot`. The image shows why 
reprojection is necessary: the .gpx data are on a totally different scale
than the shapefile of London. Hence the tiny dot at the bottom right of the graph.
We will now reproject the data, allowing `lnd` and `shf2lds.simple` to be
usefully plotted on the same graphic:


```r
lnd.wgs84 <- spTransform(lnd, CRSobj = CRS("+init=epsg:4326"))
```


The above code created a new object,`lnd.wgs84`, that contains the 
same geometries as the original but in a new CRS using the 
`spTransform` function. The `CRS` argument was set to 
`"+init=epsg:4326"`, which represents the WGS84 CRS via an 
EPSG code [6]. Now `lnd` has been reprojected we can plot it 
next to the GPS data:


```r
combined <- rbind(fortify(shf2lds.simple)[, 1:2], fortify(lnd.wgs84)[, 1:2])
```

```
## Regions defined for each Polygons
```

```r
plot(combined)
```

![plot of chunk Plot of spatial objects sharing the same CRS](figure/Plot_of_spatial_objects_sharing_the_same_CRS.png) 


Although the plot of the reprojected data is squashed because the axis scales are not fixed
and distorted (*geographic* coordinates such as WGS84 should not usually be used for plotting), 
at least the relative position and shape of both objects can now be seen. The presence of the 
dotted line in the top left of the plot confirms our assumption that the GPS data is 
from around Sheffield, which is northwest of London.

### Attribute join

Because boroughs are official administrative
zones, there is much data available at this level that we 
can link to the polygons in the `lnd` object. We will use the example 
of crime data to illustrate this data availability, which is 
stored in the `data` folder available from this project's github page.


```r
load("data/crimeAg.Rdata")  # load the crime dataset from an R dataset
```


After the dataset has been explored (e.g. using the `summary` and `head` functions)
to ensure compatibility, it can be joined to `lnd`. We will use the
the `join` function in the `plyr` package but the `merge` function 
could equally be used (remember to type `library(plyr)` if needed).




`join` requires all joining variables to have the 
same name, but this work has already been done. Once 
this preparation has been done, the 
join funtion is actually very simple:


```r
lnd@data <- join(lnd@data, crimeAg)
```

```
## Joining by: name
```


Take a look at the `lnd@data` object. You should 
see new variables added, meaning the attribute join 
was successful. 

### Spatial join

A spatial join, like attribute joins, is used to transfer information
from one dataset to another. There is a clearly defined direction to
spatial joins, with the *target layer* receiving information from
another spatial layer based on the proximity of elements from both
layers to each other. There are three broad types of spatial join:
one-to-one, many-to-one and one-to-many. We will focus only the former
two as the third type is rarely used.

One-to-one spatial joins are by far the easiest to understand and
compute because they simply involve the transfer of attributes in one
layer to another, based on location. A one-to-one join is depicted in
figure x below.

![plot of chunk Illustration of a one-to-one spatial
join](figure/Illustration_of_a_one-to-one_spatial_join_.png)

Many-to-one spatial joins involve taking a spatial layer with many
elements and allocating the attributes associated with these elements to
relatively few elements in the target spatial layer. A common type of
many-to-one spatial join is the allocation of data collected at many
point sources unevenly scattered over space to polygons representing
administrative boundaries, as represented in Fig. x.


```r
lnd.stations <- readOGR("data/", "lnd-stns", p4s = "+init=epsg:27700")
```

```
## OGR data source with driver: ESRI Shapefile 
## Source: "data/", layer: "lnd-stns"
## with 2532 features and 6 fields
## Feature type: wkbPoint with 2 dimensions
```

```r
plot(lnd)
plot(lnd.stations[round(runif(n = 500, min = 1, max = nrow(lnd.stations))), 
    ], add = T)
```

![plot of chunk Input data for a spatial join](figure/Input_data_for_a_spatial_join.png) 


The above code reads in a `SpatialPointsDataFrame` consisting of 2532
transport nodes in and surrounding London and then plots a random sample
of 500 of these over the previously loaded borough level administrative
boundaries. The reason for piloting a sample of the points rather than
all of them is that the boundary data becomes difficult to see if all of
the points are piloted. It is also useful to see and practice sampling
techniques in practice; try to plot only the first 500 points, rather
than a random selection, and describe the difference.

The most obvious issue with the point data from the perspective of a
spatial join with the borough data is that many of the points in the
dataset are in fact located outside the region of interest. Thus, the
first stage in the analysis is to filter the point data such that only
those that lie within London's administrative zones are selected. This
in itself is a kind of spatial join, and can be accomplished with the
following code.


```r
proj4string(lnd) <- proj4string(lnd.stations)
lnd.stations <- lnd.stations[lnd, ]  # select only points within lnd
plot(lnd.stations)  # check the result
```

![plot of chunk A spatial subset of the points](figure/A_spatial_subset_of_the_points.png) 


The station points now clearly follow the form of the `lnd` shape,
indicating that the procedure worked. Let's review the code that allowed
this to happen: the first line ensured that the CRS associated with each
layer is *exactly* the same: this step should not be required in most
cases, but it is worth knowing about. Of course, if the coordinate
systems are *actually* different in each layer, the function
`spTransform` will be needed to make them compatible. This procedure is
discussed in section !!!. In this case, only the name was slightly
different hence direct alteration of the CRS name via the function
`proj4string`.

The second line of code is where the magic happens and the brilliance of
R's sp package becomes clear: all that was needed was to place another
spatial object in the row index of the points (`[lnd, ]`) and R
automatically understood that a subset based on location should be
produced. This line of code is an example of R's 'terseness' - only a
single line of code is needed to perform what is in fact quite a complex
operation.

Spatial aggregation
-------------------

Now that only stations which *intersect* with the `lnd` polygon have
been selected, the next stage is to extract information about the points
within each zone. This many-to-one spatial join is also known as
*spatial aggregation*. To do this there are a couple of approaches: one
using the `sp` package and the other using `rgeos` (see Bivand et al.
2013, 5.3).

As with the *spatial subest* method described above, the developers of R
have been very clever in their implementation of spatial aggregations
methods. To minimise typing and ensure consistency with R's base
functions, `sp` extends the capabilities of the `aggregate` function to
automatically detect whether the user is asking for a spatial or a
non-spatial aggregation (they are, in essence, the same thing - we
recommend learning about the non-spatial use of `aggregate` in R for
comparison).

Continuing with the example of station points in London polygons, let us
use the spatial extension of `aggregate` to count how many points are in
each borough:


```r
lndStC <- aggregate(lnd.stations, by = lnd, FUN = length)
summary(lndStC)
plot(lndStC)
```


As with the spatial subset function, the above code is extremely terse.
The aggregate function here does three things: 1) identifies which
stations are in which London borough; 2) uses this information to
perform a function on the output, in this case `length`, which simply
means "count" in this context; and 3) creates a new spatial object
equivalent to `lnd` but with updated attribute data to reflect the
results of the spatial aggregation. The results, with a legend and
colours added, are presented in Fig !!! below.




![Number of stations in London boroughs](figure/nStations.png)

As with any spatial attribute data stored as an `sp` object, we can look
at the attributes of the point data using the `@` symbol:


```r
head(lnd.stations@data)
```

```
##    CODE          LEGEND FILE_NAME NUMBER                   NAME MICE
## 91 5520 Railway Station  gb_south  17607        Belmont Station   19
## 92 5520 Railway Station  gb_south  17608  Woodmansterne Station    5
## 93 5520 Railway Station  gb_south  17609 Coulsdon South Station   11
## 94 5520 Railway Station  gb_south  17610        Smitham Station   14
## 95 5520 Railway Station  gb_south  17611         Kenley Station   11
## 96 5520 Railway Station  gb_south  17612        Reedham Station    8
```


In this case we have three potentially interesting variables: "LEGEND",
telling us what the point is, "NAME", and "MICE", which represents the
number of mice sightings reported by the public at that point (this is a
fictional variable). To illustrate the power of the `aggregate`
function, let us use it to find the average number of mice spotted in
transport points in each London borough, and the standard deviation:


```r
lndAvMice <- aggregate(lnd.stations["MICE"], by = lnd, FUN = mean)
summary(lndAvMice)
lndSdMice <- aggregate(lnd.stations["MICE"], by = lnd, FUN = sd)
summary(lndSdMice)
```





### Clipping
