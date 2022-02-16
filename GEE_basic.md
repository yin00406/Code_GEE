# Basic function in GEE

## Online learning resources

[GEE community](https://developers.google.cn/earth-engine/tutorials/community/sar-basics?hl=zh-cn)

## Load data

```javascript
Map.addLayer(table_example, {color: 'orange'}, "t")
Map.centerObject(table_example, 5)
Map.setOptions("HYBRID") // TERRAIN

```

## Output data

```javascript
Export.table.toDrive({
  collection: table,
  description: 'table',
  folder: 'CDL',
  fileFormat: 'SHP'
})
```



## Date

```javascript
var date = ee.Date.fromYMD(2000,1,1) // 2020-01-01
```

#### ImageCollection data

```javascript
print(ImageCollection_example.size()) // Returns the number of elements in the collection

ImageCollection_example.filterDate("", "")
					   .filterBounds()
					   .first() // 

```



#### FeatureCollection data

```javascript
//? FeatureCollection data has geometry, but the features in the collection don't have
```



## Geometry

```javascript
var geoArea = geometry.area(maxError); // Finding the area of a geometry
var linLen = lineString.length(maxError); // Finding the length of a line
var geoPeri = geometry.perimeter(maxError); // Finding the perimeter of a geometry
var simpGeo = geometry.simplify(maxError); // Reducing number of vertices in geometry
var centrGeo = geometry.centroid(maxError); // Finding the centroid of a geometry
var buffGeo = geometry.buffer(radius, maxError); // Creating buffer around a geometry
var bounGeo = geometry.bounds(maxError); // Finding the bounding rectangle of a geometry
var convexGeo = geometry.convexHull(maxError); // Finding the smallest polygon that can envelope a geometry
var interGeo = geometry1.intersection(geometry2, maxError); // Finding common areas between two or more geometries
var unGeo = geometry1.union(geometry2, maxError); // Finding the area that includes two or more geometries
```

#### An example about geometry operations

```java
//t We begin by zooming to the region of interest and loading/creating the geometries of interest by extracting them from the corresponding features.
// Set map center over the state of CT.
Map.setCenter(-72.6978, 41.6798, 8);
// Load US county dataset.
var countyData = ee.FeatureCollection('TIGER/2018/Counties');
// Filter the counties that are in Connecticut (more on filters later).
var countyConnect = countyData.filter(ee.Filter.eq('STATEFP', '09'));
// Get the union of all the county geometries in Connecticut.
var countyConnectDiss = countyConnect.union(100);
// Create a circular area using the first county in the Connecticut
// FeatureCollection.
var circle = ee.Feature(countyConnect.first())
    .geometry().centroid(100).buffer(50000, 100);
// Add the layers to the map with a specified color and layer name.
Map.addLayer(countyConnectDiss, {color: 'red'}, 'CT dissolved');
Map.addLayer(circle, {color: 'orange'}, 'Circle');

//t Using the bounds() function, we can find the rectangle that emcompasses the southernmost, westernmost, easternmost, and northernmost points of the geometry.
var bound = countyConnectDiss.geometry().bounds(100);
// Add the layer to the map with a specified color and layer name.
Map.addLayer(bound, {color: 'yellow'}, 'Bounds');

//t In the same vein, but not restricting ourselves to a rectangle, a convex hull (convexHull()) is a polygon covering the extremities of the geometry.
var convex = countyConnectDiss.geometry().convexHull(100);
// Add the layer to the map with a specified color and layer name.
Map.addLayer(convex, {color: 'blue'}, 'Convex Hull');

//t Moving on to some basic operations to combine multiple geometries, the intersection (intersection()) is the area common to two or more geometries.
var intersect = convex.intersection(circle, 100);
// Add the layer to the map with a specified color and layer name.
Map.addLayer(intersect, {color: 'green'}, 'Circle and convex intersection');

//t The union (union()) is the area encompassing two or more features.
// number is the maximum error in meters.
var union = convex.union(circle, 100);
// Add the layer to the map with a specified color and layer name.
Map.addLayer(union, {color: 'purple'}, 'Circle and convex union');

//t We can also find the spatial difference (difference()) between two geometries.
var diff = convex.difference(circle, 100);
// Add the layer to the map with a specified color and layer name.
Map.addLayer(diff, {color: 'brown'}, 'Circle and convex difference');

//t Finally, we can calculate and display the area, length, perimeter, etc. of our geometries.
// Find area of feature.
var ar = countyConnectDiss.geometry().area(100);
print(ar);
// Find length of line geometry (You get zero since this is a polygon).
var length = countyConnectDiss.geometry().length(100);
print(length);
// Find perimeter of feature.
var peri = countyConnectDiss.geometry().perimeter(100);
print(peri);

```

