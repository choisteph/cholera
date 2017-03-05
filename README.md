
<!-- README.md is generated from README.Rmd. Please edit that file -->
### cholera: amend, augment and aid analysis of John Snow's 1854 cholera data

John Snow's map of the 1854 Soho, London cholera outbreak is one of the best known examples of data visualization and information design:

![](vignettes/msu-snows-mapB.jpg)

The "textbook" account is that Snow used the map to show that cholera was a waterborne not airborne disease, to identify the water pump on Broad Street as the source of the disease, and to convince officials to remove pump's handle thereby stemming the tide of the outbreak. Little of this stands up to careful scrutiny. The greater puzzle is why despite the evidence (and the map), Snow failed to convince both local officials and his peers in the scientific community of the validity of his claims, and why we hold the map such high regard today.

To address these questions, this package is designed to help users assess and explore the map and the data behind it. The starting point is the second and lesser-known version of the map, which appeared in the official report on the outbreak. While the first map (above) is more important, the second (below) is more important:

![](vignettes/fig12-6.png)

### pump neighborhoods

What makes this second version important is the graphical annotation that describes the "neighborhood" of the Broad Street pump: the residences that are most likely to use the pump that Snow suspected as being the source of the outbreak. These "neighborhoods" are pivotal to Snow's claim that cholera is a waterborne rather than airborne disease. The reason is that in 1854 getting drinking water meant physically fetching it from a public pump. As a result, if Snow's argument about cholera was correct, the outbreak should literally stop at the neighborhood's borders.

While the details and the actual method Snow used to annotate the map are lost to time, we can with the help of an amended digitization of the map replicate his results.

I do so by providing for two different approaches to computing pump neighborhoods. The first is Voronoi tessellation, which is based on the Euclidean distance between pumps:

``` r
library(cholera)
plot(neighborhoodVoronoi())
```

![](README-voronoi-1.png)

While a popular and easy to compute choice, the drawback is that it assumes that people walk through walls. The more accurate but harder to compute choice is to use the actual walking distances along the streets of Soho. In fact, Snow writes that his annotation is based on walking distance: "the various points which have been found by careful measurement to be at an equal distance by the nearest road from the pump in Broad Street and the surrounding pumps". To reconstruct and extend his efforts, I do the following.

First, I transform the street data, which records roads as line segments, into a "social" graph or network. With this, computing walking distance becomes a graph theory problem: it is the shortest path between a given case (observed or simulated) and its nearest pump:

``` r
walkingPath(150, zoom = TRUE)
```

![](README-path-1.png)

Next, to compute the different walking distance-based neighborhoods, I efficiently apply the "rinse and repeat" principle to all cases:

``` r
plot(neighborhoodWalking())
```

![](README-walk-1.png)

We can further explore the data by including or excluding certain pumps. This is important because factors other than time or distance may play a role. For example, Snow argues that water from the pump on Little Marlborough Street pump (\#6) was of low quality and that people in that neighborhood preferred the water from the Broad Street pump (\#7). To investigate this possibility, we simply exclude the pump on Little Marlborough Street from our computation:

``` r
plot(neighborhoodWalking(-6))
```

![](README-walk6-1.png)

### installation

``` r
# install.packages("devtools")
devtools::install_github("lindbrook/cholera", build_vignettes = TRUE)
```
