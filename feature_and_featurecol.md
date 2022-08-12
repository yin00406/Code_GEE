# Feature and FeatureCollection

## Feature

```javascript
// Dictionary to Feature, Feature to FeatureCollection
var dict = {Property_1:xxx, 
             Property_2:xxx, 
             Property_3:xxx
            }
var List = ee.List([])
List = List.add(ee.Feature(null, dict)) // we don't need geo info, so it is null
FeatureCollection = ee.FeatureCollection(List)

```

### Operations related to property

```javascript
// use .get() when you want to get a property
Feature.first().get('NAME') // column is a summary of each feature properties
// Image has the same function
```

### Buffer and morphological operations

#### Compute centroids and buffers for FeatureCollection

````javascript
var polygons = ee.FeatureCollection('xxxxx');

var getCentroids = function(feature) {
  return feature.set({polyCent: feature.centroid()});
};

var bufferPoly = function(feature) {
  return feature.buffer(Z);   // substitute in your value of Z here, unit is meter
};

var fcWithCentroids = polygons.map(getCentroids);
print(fcWithCentroids) // access the features and look at the properties, for which there should be a point with the centroid coords

var bufferedPolys = polygons.map(bufferPoly);
Map.addLayer(bufferedPolys, {}, 'Buffered polygons');
Map.addLayer(polygons, {}, 'Unbuffered polygons'); // for comparison
````

#### Simplify the geometry of FeatureCollection

```javascript
// Set map center over the state of CT.
Map.setCenter(-72.6978, 41.6798, 8);
// Load US county dataset.
var countyData = ee.FeatureCollection('TIGER/2018/Counties');
// Filter the counties that are in Connecticut.
var countyConnect = countyData.filter(
  ee.Filter.eq('STATEFP', '09'));
// Add the layer to the map with a specified color and layer name.
Map.addLayer(countyConnect, {color: 'red'}, 'Original Collection')

function performMap(feature) {
 // Reduce number of vertices in geometry; the number is to specify maximum
 // error in meters. This is only for illustrative purposes, since Earth Engine
 // can handle up to 1 million vertices.
 var simple = feature.simplify(10000);
 // Find centroid of geometry.
 var center = simple.centroid(100);
 // Return buffer around geometry; the number represents the width of buffer,
 // in meters.
 return center.buffer(5000, 100); // 5000: distance of the buffering, unit is meter if no reprojection.
}

var mappedCentroid = countyConnect.map(performMap);
// Add the layer to the map with a specified color and layer name.
Map.addLayer(mappedCentroid, {color: 'blue'}, 'Mapped buffed centroids');
```



## FeatureCollection

```javascript
// creat a empty featurecollection
var list = []
FeatureCollection = ee.FeatureCollection(List)

FeatureCollection = FeatureCollection.merge(FeatureCollection_1) // In general, if many collections need to be merged, consider placing them all in a collection and using FeatureCollection.flatten() instead. Repeated use of FeatureCollection.merge() will result in increasingly long element IDs and reduced performance.

```

### Wait for analysis

