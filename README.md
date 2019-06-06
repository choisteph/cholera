
<!-- README.md is generated from README.Rmd. Please edit that file -->
[![CRAN\_Status\_Badge](http://www.r-pkg.org/badges/version/cholera)](https://cran.r-project.org/package=cholera)
[![GitHub\_Status\_Badge](https://img.shields.io/badge/GitHub-0.6.0.9031-red.svg)](https://github.com/lindbrook/cholera/blob/master/NEWS)

## cholera: amend, augment and aid analysis of Snow’s cholera map

### package features

  - Fixes three apparent coding errors in Dodson and Tobler’s 1992
    digitization of Snow’s map.
  - “Unstacks” the data in two ways to make analysis and visualization
    easier and more meaningful.
  - Computes and visualizes “pump neighborhoods” based on Euclidean
    distance (Voronoi tessellation) and walking distance.
  - Overlay graphical elements and features like kernel density
    estimates, Voronoi diagrams, Snow’s Broad Street neighborhood, and
    notable landmarks (John Snow’s residence, the Lion Brewery, etc.)
    via `add*()` functions.
  - Includes a variety of functions to find and highlight cases, roads,
    pumps and paths.
  - Appends street names to the `roads` data set.
  - Includes the revised pump data used in the second version of Snow’s
    map from the Vestry report, which also includes the “correct”
    location of the Broad Street pump.
  - Adds two aggregate time series fatalities data sets, taken from the
    Vestry report.

### getting started

To install ‘cholera’ from CRAN:

``` r
install.packages("cholera")
```

To install the current development version from GitHub:

``` r
# Note that you may need to install the 'devtools' package:
# install.packages("devtools")

# For 'devtools' (< 2.0.0)
devtools::install_github("lindbrook/cholera", build_vignettes = TRUE)

# For 'devtools' (>= 2.0.0)
devtools::install_github("lindbrook/cholera", build_opts = c("--no-resave-data", "--no-manual"))
```

### warning message

With R version 3.6.0, you may see the warning below when loading
‘cholera’:

    > library(cholera)
    Registered S3 methods overwritten by 'ggplot2':
      method         from
      [.quosures     rlang
      c.quosures     rlang
      print.quosures rlang

My understanding is that it’s more annoying than problematic. It’s fixed
in ‘ggplot2’ version \>= 3.2.0. If it’s not yet on CRAN, install the
development version from GitHub:

``` r
# For 'devtools' (>= 2.0.0)
devtools::install_github("tidyverse/ggplot2", build_opts = c("--no-resave-data", "--no-manual"))
```

## background

John Snow’s map of the 1854 cholera outbreak in London, which appeared
in *On The Mode Of Communication Of Cholera*, is one of the best known
examples of data visualization and information design:

![](vignettes/msu-snows-mapB.jpg)

By plotting the number and location of fatalities on a map, Snow was
able to a perform a task that is easily taken for granted today:
visualizing a spatial distribution. Looking at the results with our
modern eye, the pattern on the map seems unmistakable, if not
self-evident. The map appears to support Snow’s claims that cholera is a
waterborne disease and that the pump on Broad Street is the source of
the outbreak. And yet, despite its virtues, the map failed to convince
either the authorities or Snow’s colleagues in the medical and
scientific communities. Even today, many are skeptical of this or any
map’s ability to demonstrate such claims.

Beyond considerations of time and place, what critics past and present
are picking up on is the fact that a concentration of cases around the
Broad Street pump alone is not convincing. To put it differently, as is
the map does not refute the primary rival explanation to waterborne
transmission: the pattern we see is not unlike what airborne
transmission (miasma theory) might look like. And while the presence of
a pump at or near the epicenter of the distribution of fatalities is
strong circumstantial evidence, it is nonetheless circumstantial.

### pump neighborhoods

This may be the reason why Snow added a graphical annotation to a second
lesser-known version of the map:\[1\]

![](vignettes/fig12-6.png)

Despite its hand-drawn, back-of-the-envelope appearance, Snow writes:
“The inner dotted line on the map shews \[sic\] the various points
which have been found by careful measurement to be at an equal distance
by the nearest road from the pump in Broad Street and the surrounding
pumps …” (Ibid., p. 109.). In other words, guided by the principle that
all else being equal people tend to choose the closest pump Snow is
computing a *pump neighborhood*: the set of addresses or locations
defined by their relative proximity to a specific water pump. By
identifying the neighborhood of the Broad Street pump, Snow’s annotation
sets limits on where we should and should *not* find fatalities. In
short, Snow’s annotation is a hypothesis or prediction.

## computing pump neighborhoods

While the specifics of his data and method of computation appear to be
lost to history, I reverse engineer what I consider to be his approach
by doing the following. First, from the quotation above I infer that his
measure of proximity is the walking distance along the streets of
Soho.\[2\] Second, putting aside aside questions about the map’s
accuracy,\[3\] which might be excellent, I set the map to be the
definitive “text” and de facto source of data.\[4\] I then write
functions to compute and visualized walking distances on the map.

The value of these functions goes beyond replicating and validating
Snow’s methods. By creating counterfactual scenarios by computing
hypothetical neighborhoods via the selective inclusion or exclusion of
pumps or by considering additional measures of proximity (e.g.,
Euclidean), we can better assess the ability of the map to test Snow’s
claims.

## walking v. Euclidean neighborhoods

While walking distanced based neighborhoods are based on paths that
follow streets, Euclidean distance based neighborhoods are based on
straight line paths between a location and the nearest (or
selected):

``` r
streetNameLocator(zoom = 1, cases = NULL, highlight = FALSE, add.subtitle = FALSE, add.title = FALSE)
title(main = "Walking Distances")
invisible(lapply(c(1, 191, 46, 363, 85), addWalkingPath))

streetNameLocator(zoom = 1, cases = NULL, highlight = FALSE, add.subtitle = FALSE, add.title = FALSE)
title(main = "Euclidean Distances")
invisible(lapply(c(1, 191, 46, 363, 85), addEuclideanPath))
```

<img src="man/figures/README-unnamed-chunk-5-1.png" width="50%" /><img src="man/figures/README-unnamed-chunk-5-2.png" width="50%" />

To build a neighborhood, we apply this algorithm to each location or
“address” with at least one observed fatality. This builds the
“observed” neighborhoods:

``` r
plot(neighborhoodWalking())
plot(neighborhoodEuclidean())
```

<img src="man/figures/README-unnamed-chunk-6-1.png" width="50%" /><img src="man/figures/README-unnamed-chunk-6-2.png" width="50%" />

Ultimately, for testing purposes we also want the “expected”
neighborhoods. For walking neighborhoods, I use the same approach but
use simulated data. Using `sp::spsample()` and `sp::Polygon()`, I place
20,000 regularly spaced points, which lie approximately 6 meters apart,
`unitMeter(dist(regular.cases[1:2, ]))`, across the face of the map and
then compute the shortest path to the nearest pump.\[5\] For Euclidean
neighborhoods, we can compute Voronoi diagrams.\[6\]

``` r
plot(neighborhoodWalking(case.set = "expected"), "area.polygons")

plot(neighborhoodVoronoi())
```

<img src="man/figures/README-unnamed-chunk-7-1.png" width="50%" /><img src="man/figures/README-unnamed-chunk-7-2.png" width="50%" />

### Walking neighborhoods

To explore “observed” walking neighborhoods, use `neighborhoodWalking()`
with the `pump.select` argument:

``` r
plot(neighborhoodWalking(6:7))
plot(neighborhoodWalking(-7))
```

<img src="man/figures/README-unnamed-chunk-8-1.png" width="50%" /><img src="man/figures/README-unnamed-chunk-8-2.png" width="50%" />

To explore “expected” walking neighborhoods, add case.set = “expected”
argument:

``` r
plot(neighborhoodWalking(6:7, case.set = "expected"), type = "area.polygons")
plot(neighborhoodWalking(-7, case.set = "expected"), type = "area.polygons")
```

<img src="man/figures/README-unnamed-chunk-9-1.png" width="50%" /><img src="man/figures/README-unnamed-chunk-9-2.png" width="50%" />

### Euclidean neighborhoods

To explore “observed” Euclidean neighborhoods, use
`neighborhoodEuclidean()` with the `pump.select` argument:

``` r
plot(neighborhoodEuclidean(6:7))
plot(neighborhoodEuclidean(-7))
```

<img src="man/figures/README-unnamed-chunk-10-1.png" width="50%" /><img src="man/figures/README-unnamed-chunk-10-2.png" width="50%" />

To explore “expected” Euclidean neighborhoods, use
`neighborhoodVoronoi()` with the `pump.select` argument:

``` r
plot(neighborhoodVoronoi(6:7))
plot(neighborhoodVoronoi(-7))
```

<img src="man/figures/README-unnamed-chunk-11-1.png" width="50%" /><img src="man/figures/README-unnamed-chunk-11-2.png" width="50%" />

### note on `neighborhoodWalking()` and `neighborhoodEuclidean()`

`neighborhoodWalking()` and `neighborhoodEuclidean()` are
computationally intensive. Using R version 3.5.2 on a single core of a
2.3 GHz Intel i7, plotting observed paths to PDF takes about 5 seconds;
doing the same for expected paths takes about 30 seconds. Using the
function’s parallel implementation on 4 physical (8 logical) cores, the
times fall to about 4 and 13 seconds.

Note that parallelization is currently only available on Linux and Mac.

Also, note that although some precautions are taken in R.app on macOS,
the developers of the ‘parallel’ package, which `neighborhoodWalking()`
uses, strongly discourage against using parallelization within a GUI or
embedded environment. See `vignette("parallel")` for details.

### vignettes

The vignettes, which are available in the package as well as online at
the links below, go into detail on a variety of topics.

[Duplicate and Missing
Cases](https://github.com/lindbrook/cholera/blob/master/docs/vignettes/duplicate.missing.cases.md)
describes the two coding errors and three misplaced cases I argue are
present in Dodson and Tobler’s (1992) digitization of Snow’s map.
Documentation and details about the fix are found online in [“Note on
Duplicate and Missing
Cases”](https://github.com/lindbrook/cholera/blob/master/docs/notes/duplicate.missing.cases.notes.md).

[“Unstacking”
Bars](https://github.com/lindbrook/cholera/blob/master/docs/vignettes/unstacking.bars.md)
discusses the inferential and visual importance of “unstacking” the bars
in Snow’s map and the two “unstacked” data sets, which use “fatalities”
and “addresses” as the units of
observation.

[Roads](https://github.com/lindbrook/cholera/blob/master/docs/vignettes/roads.md)
covers issues related to roads. This includes discussion of how and why
I move pump \#5 from Queen Street (I) to Marlborough Mews, the overall
structure of the `roads` data set, “valid” road names, and my back of
the envelope translation from the map’s nominal scale to meters (and
yards).

[voronoiPolygons(): Tiles, Triangles and
Polygons](https://github.com/lindbrook/cholera/blob/master/docs/vignettes/tiles.polygons.md)
focuses on the `voronoiPolygons()`, which extracts the vertices of
triangles (Delauny triangulation) and tiles (Dirichelet or Voronoi
tessellation) from `deldir::deldir()` for use with polygon() and related
functions.

[Kernel Density
Plot](https://github.com/lindbrook/cholera/blob/master/docs/vignettes/kernel.density.md)
discusses the the syntax of `addKernelDensity()`, which allows you to
define “populations” and subsets of pumps. This syntax is used in many
of the functions in ‘cholera’.

[Time
Series](https://github.com/lindbrook/cholera/blob/master/docs/vignettes/time.series.md)
discusses functions and data related to fatalities time series data and
the question of the effect of the removal of the handle from the Broad
Street pump.

### lab notes

The lab notes, which are only available online, go into greater detail
about some of the issues and topics discussed in the vignettes:

[note on duplicate and missing
cases](https://github.com/lindbrook/cholera/blob/master/docs/notes/duplicate.missing.cases.notes.md)
documents the specifics of how I “fixed” two apparent coding errors and
three misplaced case in Dodson and Tobler’s data.

[computing street
addresses](https://github.com/lindbrook/cholera/blob/master/docs/notes/unstacking.bars.notes.md)
discusses how I use orthogonal projection and hierarchical cluster
analysis to “unstack” bars and compute a stack’s “address”.

[Euclidean v. Voronoi
neighborhoods](https://github.com/lindbrook/cholera/blob/master/docs/notes/euclidean.voronoi.md)
discusses why there are separate functions for `neighborhoodEuclidean()`
and `neighborhoodVoronoi()`.

[points v.
polygons](https://github.com/lindbrook/cholera/blob/master/docs/notes/pump.neighborhoods.notes.md)
discusses the tradeoff between using points() and polygon() to plot
“expected” area neighborhood plots and the computation of polygon
vertices.

[references](https://github.com/lindbrook/cholera/blob/master/docs/notes/references.md)
is an informal list of articles and books about cholera, John Snow and
the 1854 outbreak.

### Notes

1.   *Report On The Cholera Outbreak In The Parish Of St. James,
    Westminster, During The Autumn Of 1854*

2.  The computation of walking distance is by no means new (see Shiode,
    2012). Another approach is to use GIS. For applications that don’t
    need to consider the actual historic walking distances, this
    layers-based approach, which typically relies on current maps, may
    be sufficient: e.g.,
    <https://www.theguardian.com/news/datablog/2013/mar/15/john-snow-cholera-map>.

3.  The map is actually a commercial map that Snow annotated.

4.  I use a modified version of Dodson and Tobler’s 1992 digitization.
    Each bar and each pump is assigned a unique x-y coordinate. Each
    road is translated into a series of straight line segments, defined
    by those segments’ endpoints. While the original data,
    <http://www.ncgia.ucsb.edu/pubs/snow/snow.html>, are no longer
    available, they are preserved in Michael Friendly’s ‘HistData’
    package. Note that a future version of this package will re-digitize
    and geocode the map.

5.  These data are found in `regular.cases` data frame. Note that
    because the map is not rectangular, there are only 19,993 cases.

6.  <http://www.ams.org/samplings/feature-column/fcarc-voronoi>
