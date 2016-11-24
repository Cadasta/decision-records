# 0002: Map performance upgrades

- **Author:** Oliver Roick
- **Status:** Accepted, 2016-11-23

## Context

Towards the end of Q1 2017, we expect to have at least 200,000 records stored in our database; 100,000 of which will belong to one project. With the current architecture, the map views will not be able to cope with this amount of features. 

Currently, there are two client-side bottlenecks:

1. **Synchronous all-at-once loading of geographic features:** When a page with a map is requested, _all_ spatial units in a project are queried from the database, serialised to a GeoJSON `FeatureCollection`, written to the HTML output, transferred across the network via HTTP, then the HTML page is parsed in the browser, and finally the map is rendered. The page size for Uttaran is currently 3.5M; geometries are almost exclusively point features. The page size will increase many times over when we have to deal wit more complex geometries.
2. **Leaflet.Deflate:** This plugin is used to change the visualization of geographic features based on their size and the map’s zoom level. Every time the user zooms the map, Leaflet.Deflate iterates across _all_ features added to the map and decides whether the feature should be displayed as a point or as a line/polygon. 

## Decision

To increase page loading times and lower latency until the map starts to render, we will introduce asynchronous loading and rendering of the map.

We build on our existing architecture and introduces asynchronous loading of geographic features on the map and attempt to improve the performance client-side map-related Javascript plugins.

### Asynchronous loading

We will extend existing API endpoints that list spatial units with pagination, which allows us to load geographic features using several separate requests that each return a fraction of all spatial units in a project. 

The page will load with an empty map (or a map providing background information, such as background map tiles, the project extent); spatial units are loaded and added to the map when the DOM is ready. 

### Improve client-side map plugins. 

There are three approaches (we need to decide for one):

1. We will lower the number of features that is processed by ignoring points when iterating over features after zooming the map. 
2. We will reduce the amount of features being deflated/inflated after zoom by only considering those features that are affected. That means, if the user zooms from level 2 to level 5, we won’t process features where the threshold for deflating is lower than 2 or greater than 5. 

## Consequences

Splitting uploading and rendering of features will reduce page load times. Improving processing of features in Leaflet.Deflate will also improve the performance of map interactions. 

This is a first step towards improving the client-side performance on map views. We need to keep it mind that especially the work on improving Leaflet.Deflate might not improve performance as required in our longer-term goals. We need to do additional research towards alternative approaches, for instance, using vector tiles or raster tiles. 
