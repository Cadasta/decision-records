# 0005: Single Page Map Application

- **Author:** Lindsey Jacks
- **Status:** Accepted, 2017-01-25



## Context

As our platform grows and demands for an increasing number of records are being made, we need to continue to find ways to support and optimize performance. One of the bottle necks for performance is the rendering of spatial records onto map display pages within a project view. Project view currently consist of four tabs: 1) Overview 2) Map 3) Parties 4) Resources.

Both **overview** and **map** contain a leaflet map which loads all of the projects spatial records onto the page. Currently, when switching from tab to tab this data has to be reloaded with each page. 

## Decision

To remove the delay in loading spatial records across multiple pages in a single project, we will combine all of the tabs within a project into a single page application, which will pass previously loaded data between tabs. In order to do this we will need to implement the following things:

1. **We will enable GeoJSON vector tiles.** We will change spatial.asyc.SpatialUnitList so it can serve GeoJSON vector tiles. The view will support the [XYZ tile convention](http://wiki.openstreetmap.org/wiki/Slippy_map_tilenames). We will change loading of features on the client-side to support GeoJSON vector tiles.
2. **We will update existing views and templates so they can be served asynchronously.** For consistency, both the URLs and view classes will be moved to the async component the corresponding Django app. Scaffolding will be removed from the HTML template, so only the contents of the right-hand side panel are rendered.
3. **We will implement a minimal router and async loading.** The URL displayed in the address bar will be coordinated with the current view. The view should be updated according to the URL when users access the page via a permalink. (Example: [A JavaScript router in 20 lines](http://joakim.beng.se/blog/posts/a-javascript-router-in-20-lines.html))
4. **We will implement asynchronous loading into the client-side.** We will design and implement a framework to handle information flows uni directionally. Changes to the UI will only happen when the URL in the browser changes. This will trigger an event, which will load the resource from the backend and display some loading info to the user. Upon receiving the response, the loading info will be removed, and the contents are displayed in the panel. Ideally, we would track to app's current state in one central variable and update only the state when the user interacts with the platform. We will listen for changes to the stated and update the UI according to the current state.
5.  **We will store the map position** either in the browser session and/or the URL so it can be recovered through a permalink.

## Consequences

Transferring previously loaded data between views will reduce page loading times.

