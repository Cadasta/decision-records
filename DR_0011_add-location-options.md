# 0010: External file handling

- **Author:** Chandra Lash
- **Status:** WIP Draft Proposal, 2017-06-29
- **Github Issues:** [#664](https://github.com/Cadasta/cadasta-platform/issues/664), [#1590](https://github.com/Cadasta/cadasta-platform/issues/1590)



### Problem Statement

Some partners are using GPX files to collect geometries and it'd be useful for them to be able to import a GPX file to define the geometry when creating a new location via web UI. 

In addition, some partners using paper surveys are collecting coordinates for data points and need a way to manually enter them into the web UI.

### Context / Use Case

[Detailed requirements for GPX Import](https://devwiki.corp.cadasta.org/GPX%20Import)

### Description of the implemented use cases

### Overview of the suggested solution

Via the web UI, when we are creating a new location and defining the geometry, we would have the following methods to define the geometry:

- Draw on map (current option)
- Input coordinates (modal with latitude and longitude input in short term for data points, eventually text area input of coordinates for polygons)
- Import a file (modal with file-picker to upload a gpx (or shp) file)


### Analysis of the alternatives

An alternative solution is by using a GIS software to convert GPX to SHP and then import that shp file when [#1589](https://github.com/Cadasta/cadasta-platform/issues/1589) is implemented. This does not address the manual input of coordinates.

Another concern regarding the use of GPX files to define the geometries is that these files are usually generated a pretty ugly geometries and usually need some clean up before creating the actual geometry.

### Scalability

### Maintainability


