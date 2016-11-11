# 0002: Map performance upgrades

- **Author:** Oliver Roick
- **Status:** Draft proposal

## Context

Towards the end of Q1 2017, we expect to have at least 200,000 records stored in our database; 100,000 of which will belong to one project. With the current architecture, the map views will not be able to cope with this amount of features. 

Currently, there are two client-side bottlenecks:

1. **Synchronous all-at-once loading of geographic features:** When a page with a map is requested, _all_ spatial units in a project are queried from the database, serialised to a GeoJSON `FeatureCollection`, written to the HTML output, transferred across the network via HTTP, then the HTML page is parsed in the browser, and finally the map is rendered. The page size for Uttaran is currently 3.5M; geometries are almost exclusively point features. The page size will increase many times over when we have to deal wit more complex geometries.
2. **Leaflet.Deflate:** This plugin is used to change the visualization of geographic features based on their size and the map’s zoom level. Every time the user zooms the map, Leaflet.Deflate iterates across _all_ features added to the map and decides whether the feature should be displayed as a point or as a line/polygon. 

## Decision

To increase page loading times and lower latency until the map starts to render, we will introduce asynchronous loading and rendering of the map.

There are three approaches (we need to decide for one):

1. **We improve performance on top of our existing architecture** by developing an approach to gradually load features onto the map. The page loads first with an empty map and as soon as the page is ready, automatically start loading features onto the map. Our existing API needs to be extended with pagination to allow for sliced retrieval of features. 

    This approach also requires improvements to Leaflet.Deflate. (1) We can lower the number of features that is processed by ignoring points when iterating over features after zooming the map. (2) Reduce the amount of features being deflated/inflated after zoom by only considering those features that are affected. That means, if the user zooms from level 2 to level 5, we won’t process features where the threshold for deflating is lower than 2 or greater than 5. (3) We could further restrict that amount of processed data by only considering feature visible in the current map extent. 

2. **We implement a tile map service** using an existing mapping framework; e.g., Mapnik, Mapserver or Geoserver. Mapnik is only a renderer for geographic information, we would need to build the TMS API around it, Mapserver and Geoserver provide this out-of-the-box. The Mapnik approach is more light-weight and flexible in the first place but also requires more long-term maintenance. The tile map server should not run on our primary server; we need a new AWS instance to host the system.
3. **We implement a vector tile service.** There are [several solutions](https://github.com/mapbox/awesome-vector-tiles#servers) out there to set up a server for vector tiles. I don’t have any experience with vector tiles and what it requires to run a vector tile service. It is also currently not clear to me how vector tiles affect the performance of the map in older browsers. This approach requires _a lot_ of research. 

**Approach 1** does not require a significant extension of the current architecture; this might be the solution that is quickest to implement. There is, however, a risk that the effect of improving Leaflet.Deflate is not as good as we expected. 

**Approach 2** requires a significant extension of our architecture. An implementation may take longer that approach 1 but can be solved in reasonable time. I’m very confident that this method will improve performance significantly.

**Approach 3** requires a significant extension of our architecture. I think this is the riskiest approach, mostly because I don’t know much about it and it would need some research before implementing a solution. 

## Consequences

TBD. 
