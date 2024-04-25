# Common functions

## get header from a FeatureCollection

```javascript
// Function to extract property names as an ee.List
function getPropertyNames(featureCollection) {
  // Get the first feature from the collection
  var firstFeature = ee.Feature(featureCollection.first());
  
  // Retrieve property names from the first feature
  var propertyNames = firstFeature.propertyNames();
  
  return propertyNames;
}
```

## Exclude items from a list

```javascript
var excludeList = ee.List(["iterm1", "iterm2"])
var filteredList = list.filter(ee.Filter.inList('item', excludeList).not())
```

