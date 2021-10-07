# Basic function in GEE

#### Load data

```javascript
Map.addLayer(table_example)

Map.centerObject(table_example, 5)

```



#### Date

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
