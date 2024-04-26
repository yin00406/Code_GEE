# Classification with RF

when script reports an `out of memory`, you can split the script into multi steps and split region into multi subregions.

## Save trained RF model to asset

```javascript
var trainedRF= ee.Classifier.smileRandomForest({
      numberOfTrees: 130,
      minLeafPopulation: 1,
      bagFraction: 0.5,
      seed: 666
    }).train({
      features: SC_2022_18,
      classProperty: 'class',
      inputProperties: propertyNamesList
    });

//SAVE TRAINED MODEL
var trees = ee.List(trainedRF.explain().get('trees'))
// print(trees)
var dummy = ee.Feature(SC_point)
var col = ee.FeatureCollection(trees.map(function(x){return dummy.set('tree',x)}))
Export.table.toAsset(col,model_ID+'_'+year+"_"+time_stop,'trainedModel/'+model_ID+'_'+year+"_"+time_stop)
```

## Import saved RF model for application

```javascript
// TRANSFER RF MODEL
var RF_model = 'projects/ee-yin-ars/assets/trainedModel/'+model_ID+'_'+year+'_'+time_stop
var trees = ee.FeatureCollection(RF_model).aggregate_array('tree')
var trainedRF = ee.Classifier.decisionTreeEnsemble(trees)

var PRED_MAP = features.classify(trainedRF)
```

## Possible problems when you save RF model

If you config `numberOfTrees` with a large number (say `300`), when print `trees` in line 14, you may find that fewer than 300 trees are printed in line 14. This may because there is an upper limit for string output. In the early-prediction case, the upper limit is 130.

## Alternative code

```javascript
// RF MODEL TRAIN AND SAVE
var decisionTrees = ee.List(trainedRF.explain().get('trees'))

function encodeFeatureCollection(value) {
  var string = ee.String.encodeJSON(value)
  var stringLength = string.length()
  var maxLength = 100000
  var maxProperties = 1000
  var values = ee.List.sequence(0, stringLength, maxLength)
    .map(function (start) {
      start = ee.Number(start)
      var end = start.add(maxLength).min(stringLength)
      return string.slice(start, end)
    })
    .filter(ee.Filter.neq('item', ''))
  var numberOfProperties = values.size()
  return ee.FeatureCollection(ee.List.sequence(0, values.size(), maxProperties)
    .map(function (start) {
      start = ee.Number(start)
      var end = start.add(maxProperties).min(numberOfProperties)
      var propertyValues = values.slice(start, end)
      var propertyKeys = ee.List.sequence(1, propertyValues.size())
        .map(function (i) {
          return ee.Number(i).format('%d')
        })
      var properties = ee.Dictionary.fromLists(propertyKeys, propertyValues)
      return ee.Feature(ee.Geometry.Point([0, 0]), properties)
    }).filter(ee.Filter.notNull(['1']))
  )
}
  
Export.table.toDrive(
  {collection:encodeFeatureCollection(decisionTrees), 
  description: model_ID+'_'+year+"_"+time_stop,
  folder: 'GEE_exported_table'
  })


// RESTORE MODEL    
 function decodeFeatureCollection(featureCollection) {
    return featureCollection
      .map(function (feature) {
        var dict = feature.toDictionary()
        var keys = dict.keys()
          .map(function (key) {
            return ee.Number.parse(ee.String(key))
          })
        var value = dict.values().sort(keys).join()
        return ee.Feature(null, {value: value})
      })
      .aggregate_array('value')
      .join()
      .decodeJSON()
    }
    
    var decisionTrees = decodeFeatureCollection(RF_model)
    var classifier = ee.Classifier.decisionTreeEnsemble(decisionTrees)
    c = c.add(features.classify(classifier))
```



