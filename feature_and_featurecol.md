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





### Property related

```javascript
// use .get() when you want to get a property
Feature.first().get('NAME') // column is a summary of each feature properties
// Image has the same function
```



## FeatureCollection

```javascript
// creat a empty featurecollection
var list = []
FeatureCollection = ee.FeatureCollection(List)

FeatureCollection = FeatureCollection.merge(FeatureCollection_1) // In general, if many collections need to be merged, consider placing them all in a collection and using FeatureCollection.flatten() instead. Repeated use of FeatureCollection.merge() will result in increasingly long element IDs and reduced performance.

```

