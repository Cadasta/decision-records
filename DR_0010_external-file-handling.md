# 0010: External resource file handling

- **Author:** Chandra Lash
- **Status:** Approved, 2017-07-11
- **Backlog Issue:** [#1646](https://github.com/Cadasta/cadasta-platform/issues/1646)


## Problem overview

The Cadasta Platform is inconsistent in its referencing of external static resource files (CSS, fonts, images, JS) in the Platform. Not only does this complicate the code, it could potentially impede performance and create bugs over time due to uncontrollable third-party changes.


## Suggested solution

- Cadasta should adopt a standard practice of hosting all files locally on our servers.
- New libraries or plug-ins should be located under `cadasta-platform/cadasta/core/static/`. 
- A new folder with a descriptive name should be created.
- A subfolder with the version number should be under the original folder.
- Folders or files should replicate the original source under the version folder (i.e. `css`, `fonts`, `img`, `js`).
- File folder structure example: `core/static/leaflet-geocoder-mapzen/1.9.2/leaflet-geocoder-mapzen.css`.
- We should maintain one version of these resource files to serve on all development environments.
- These resource files should be updated as-needed as new versions are released.


## Other considerations

- Exceptions should be made on a case-by-case basis only. For example our pricing package with Inline Manual prevents us from using a standalone player.
