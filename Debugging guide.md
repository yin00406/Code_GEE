# Debugging guide

- **Don't include "." in your img or feature property, which could cause error in some cases**

- **A mapped function's arguments cannot be used in client-side operations**

  - First, Let's figure out what does "client-side operations" mean? "Client-side operations" means this method belongs to JavaScript, and it cannot be used on server-side objects. "Server-side" means GEE.
  - This error means you may use print() or other client-side operations to the server-side object (the mapped function's argument).
  - *Why cannot use print in map function?*

- **Error: Image.reduceRegion(s): The default WGS84 projection is invalid for aggregations. Specify a scale or crs & crs_transform**

  - Data that not from GEE, include those generated from GEE data, need to be reprojected before aggregations.

  `````javascript
  img = img.reproject(NLDAS.first().projection()) // change from geographic coordinate system and projected coordinate system
  `````

- 

